
> 适用环境：Windows + WSL2 + Clash / Clash Mi / Mihomo

---

## 一、问题背景

WSL2 是独立的虚拟机网络，即使 Windows 上已开启 Clash 代理，WSL2 中也无法自动使用，访问 GitHub 等外网时会报错：

```
curl: (7) Failed to connect to raw.githubusercontent.com port 443: Connection refused
```

根本原因：

- WSL2 不继承 Windows 的代理环境变量
- Clash 默认只监听 `127.0.0.1`，不允许 WSL2 连接

---

## 二、解决方案

### ✅ 方案 A：开启 Allow LAN + 使用 vEthernet IP（推荐）

**步骤 1：开启 Clash 的「允许局域网」**

在 Clash Mi / Mihomo 客户端中：

- 打开「核心设置」→ 滚动找到「局域网设备接入」
- 将「启用」和「覆写」两个开关都打开

> ⚠️ 修改配置后需要重新连接才会生效。

**步骤 2：获取 Windows vEthernet 的 IP**

在 **Windows PowerShell** 中运行：

```powershell
ipconfig
```

找到「以太网适配器 vEthernet (WSL)」下的 IPv4 地址，通常形如 `172.x.x.x`。

> ⚠️ 不要用 `/etc/resolv.conf` 里的 nameserver IP（`10.255.255.254`），在某些 WSL2 版本下无法路由到 Windows 代理。

**步骤 3：在 WSL2 中设置代理**

```bash
export https_proxy=http://172.x.x.x:7890
export http_proxy=http://172.x.x.x:7890

# 再执行你的命令
curl -fsSL https://raw.githubusercontent.com/xxx/install.sh | bash
```

将 `172.x.x.x` 替换为实际的 vEthernet (WSL) IP，端口换成 Clash 实际的混合代理端口（默认 7890）。

---

### ✅ 方案 B：开启 TUN 模式（一劳永逸）

TUN 模式在系统层面劫持所有流量，WSL2 无需任何配置即可访问外网。

- 在 Clash Mi 核心设置中点击「TUN」，将其开启
- 开启后 WSL2 中直接使用 `curl`/`wget`，无需设置任何代理环境变量

> ⚠️ TUN 模式需要管理员权限，会影响所有网络流量，按需开启。

---

### ✅ 方案 C：持久化代理配置

每次打开 WSL2 都手动 export 很麻烦，可以写入 shell 配置文件：

```bash
# 写入 ~/.bashrc 或 ~/.zshrc
HOST_IP=$(cat /etc/resolv.conf | grep nameserver | awk '{print $2}')
export https_proxy=http://$HOST_IP:7890
export http_proxy=http://$HOST_IP:7890
```

> ⚠️ 此方案使用 resolv.conf 中的 nameserver IP，如果该 IP 无效，改用硬编码的 vEthernet IP。

---

## 三、故障排查

### 确认 Clash 端口

在 Clash Mi 核心设置中查看「混合代理端口」，默认通常是 `7890`。

在 **Windows PowerShell** 中验证端口是否在监听：

```powershell
netstat -ano | findstr "7890"
```

> ⚠️ 此命令要在 Windows PowerShell 中运行，不是 WSL 终端（WSL 中没有 `findstr`）。

### 测试代理是否可达

在 WSL2 中运行：

```bash
curl -v http://172.x.x.x:7890
```

- 返回 HTTP 响应（即使是错误码）→ 代理可达 ✅
- Connection refused → Allow LAN 未生效或防火墙拦截 ❌

### Windows 防火墙放行

如果配置正确但仍然 Connection refused，在 **Windows PowerShell（管理员）** 中运行：

```powershell
New-NetFirewallRule -DisplayName "WSL Proxy" -Direction Inbound -Protocol TCP -LocalPort 7890 -Action Allow
```

---

## 四、常见错误对照

|错误信息|原因|解决方法|
|---|---|---|
|`Connection refused`（直连 GitHub）|WSL2 未配置代理|设置 `https_proxy` 环境变量|
|`Connection refused`（port 7890）|Clash 未开启 Allow LAN|开启「局域网设备接入」|
|`findstr: command not found`|在 WSL 中运行了 Windows 命令|在 PowerShell 中运行，不是 WSL|
|代理 IP `10.255.255.254` 无法连接|WSL2 网关 IP 无法路由到 Windows 代理|改用 `ipconfig` 获取的 vEthernet IP|

---

## 五、网络架构说明

理解 WSL2 的网络结构有助于自己排查问题：

- WSL2 是独立的轻量级虚拟机，有独立 IP（通常 `172.x.x.x` 网段）
- Windows 宿主机通过 `vEthernet (WSL)` 虚拟网卡与 WSL2 通信
- WSL2 中 `/etc/resolv.conf` 里的 nameserver 是 WSL 的默认网关（`10.255.255.254` 或 `172.x.x.1`）
- Clash 默认绑定 `127.0.0.1`，只有本机可访问；开启 Allow LAN 后绑定 `0.0.0.0`，WSL2 才能访问
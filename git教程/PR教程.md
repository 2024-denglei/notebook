# 本地 check out 这条 PR 并跑测试

下面是完整流程。这条 PR 的 base 是 `bigbio/sdrf-skills` 仓库，所以你得先把那个仓库 clone 下来（**不是**你现在在编辑的 plugin 仓库，虽然它们同名但那个是老仓库，在 `bigbio` org 下，这条 PR 是别人对它的改动）。

## 步骤 1：准备工作目录

你现在 `e:\SDRFskills\sdrf-skills\` 已经是 `bigbio/sdrf-skills` 的 plugin 版本，但要跑 PR #6 的代码我建议**另找一个目录**，避免和你正在开发的插件混淆。

powershell

# 选一个干净的位置

cd E:\

mkdir pr-review

cd pr-review

## 步骤 2：克隆仓库 + check out PR 分支

**方式 A（推荐，要先装 gh CLI）**：

powershell

# 装 GitHub CLI（如果还没装）

winget install --id GitHub.cli

# 登录（只需一次）

gh auth login

# 选 GitHub.com → HTTPS → 跟着提示走，在浏览器里授权

# 一键 clone + check out PR

gh repo clone bigbio/sdrf-skills

cd sdrf-skills

gh pr checkout 6

**方式 B（只用 git，不装 gh）**：

powershell

git clone https://github.com/bigbio/sdrf-skills.git

cd sdrf-skills

git fetch origin pull/6/head:pr-6

git checkout pr-6

执行完后，用 `git log --oneline -n 5` 验证，应该看到最近几条 commit 是 `Update tools/ols_client.py`、`Update tools/hallucination.py`、`Address PR review comments` 等。

## 步骤 3：拉 submodules（这条 PR 依赖 spec/ 子模块）

powershell

git submodule update --init --recursive

如果报 `not our ref` 之类的错误，用：

powershell

git submodule update --init --remote --recursive

## 步骤 4：建隔离的 Python 环境

**不要**用你现在的 

sdrf-skills conda 环境（那是给插件用的），给 PR review 单独建一个，避免依赖互相污染。

**用 conda**：

powershell

conda create -n pr6-sdrf python=3.11 -y

conda activate pr6-sdrf

**或用 venv**：

powershell

python -m venv .venv

.\.venv\Scripts\Activate.ps1

## 步骤 5：装依赖

这条 PR 的 CI workflow 里用的是：

powershell

pip install "requests>=2.28" "pydantic>=2.0" pytest

另外作者还更新了 

requirements.txt，稳妥起见一并装：

powershell

pip install -r requirements.txt

pip install pytest

## 步骤 6：跑测试

作者说"104 个测试全过"。你本地验证：

powershell

pytest -v

参数说明：

- `-v` 显示每个测试名字
    
- `--tb=short` 错误信息短一点（可选）
    
- `-x` 遇到第一个失败就停（调试时有用）
    
- `-k hallucination` 只跑名字里含 `hallucination` 的测试（聚焦 review 的模块时用）
    

### 期望结果

全绿：

==================== 104 passed in 12.34s =====================

### 常见失败

|错误|原因|解决|
|---|---|---|
|`ModuleNotFoundError: spec`|submodule 没拉|回步骤 3|
|`ConnectionError`|测试实际打了外网|说明作者的 mock 没覆盖到某个路径，这本身就是 PR 的一个缺陷，记下来|
|`FileNotFoundError: data/cell_lines.tsv`|数据文件不在仓库或没拉下来|看仓库里 `data/` 是否存在|
|`ImportError: pydantic`|`pydantic` 没装（你跳过了那条 pip）|`pip install pydantic`|

## 步骤 7：跑 CLI 验证真实使用体验

测试过了不代表好用。手动跑几条真实场景：

### 7.1 先看 CLI 帮助

powershell

python -m tools --help

python -m tools check --help

python -m tools score --help

### 7.2 用作者自己写的 example 数据

powershell

# 仓库里应该有 examples/ 或 tests/fixtures/ 的样例 SDRF

ls examples/    # 或 tests/fixtures/

# 跑幻觉检测（离线模式，不需要网）

python -m tools check examples/sample.sdrf.tsv --offline

# 质量打分

python -m tools score examples/sample.sdrf.tsv

# 自动修复

python -m tools fix examples/sample.sdrf.tsv -o fixed.tsv

# 查细胞系

python -m tools cellline lookup HeLa

python -m tools cellline stats

### 7.3 故意造一个有问题的 SDRF 测幻觉检测

我之前解读 PR 时发现 `hallucination.py` 有个 "normal" 被全局跳过的问题。你可以手动构造一个小 TSV 测它：

powershell

# 造一个有问题的 SDRF

@"

source name`tcharacteristics[organism]

sample1`tnormal

sample2`tHomo sapiens

"@ | Out-File -Encoding utf8 bad.sdrf.tsv

python -m tools check bad.sdrf.tsv --offline

按规范 `organism` 列里的 `normal` 是非法值，**应该被 flag**。如果工具没报错，就**复现了 coderabbit 指出的 bug**，你可以在 PR 里加一条评论：

> 我本地复现了这个问题：造一个 `characteristics[organism]` 为 `normal` 的 SDRF，`python -m tools check` 没有 flag 它。确认这是 "normal" 全局跳过导致的。

## 步骤 8：读代码用 IDE（关键）

测试跑完后，就可以深入读代码了。在仓库根目录打开 IDE：

powershell

code .        # VS Code

# 或

pycharm64 .   # PyCharm

这样你可以：

- 按 F12 跳定义、看函数调用链
    
- 全仓库搜索符号
    
- 断点调试单个测试
    

建议按之前讲的优先级读：

1. `tools/cli.py`（入口）
    
2. `tools/sdrf_parser.py`（基础）
    
3. `tools/hallucination.py`（最有争议的模块）
    
4. `tests/test_hallucination.py`（对照着看它的期望行为）
    

## 步骤 9：跑完想 review 时

把你的观察记在一个本地文件里，比如 `pr-6-notes.md`，格式：

md

# PR #6 本地 Review 笔记

## 测试状态

- pytest: ✅ 104 passed / ❌ X failed

- CLI smoke: ✅ / ❌

## 复现的问题

- [ ] "normal" 全局跳过：已复现，见下面

  ...

## 发现的新问题

- [ ] tools/cli.py 的 ...

## Nitpick

- ...

review 时打开这个文件对着贴评论。

---

## 快速命令总结（复制即用）

powershell

# 1. clone 并切 PR

cd E:\pr-review

gh repo clone bigbio/sdrf-skills

cd sdrf-skills

gh pr checkout 6

git submodule update --init --recursive

# 2. 建环境

conda create -n pr6-sdrf python=3.11 -y

conda activate pr6-sdrf

# 3. 装依赖

pip install -r requirements.txt pytest

# 4. 跑测试

pytest -v

# 5. 手动验证 CLI

python -m tools --help
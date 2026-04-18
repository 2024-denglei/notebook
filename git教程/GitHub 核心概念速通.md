
---

## 1. Repository（仓库）

一个仓库 = 一个项目的代码 + 全部历史。

你在看的主仓库：

https://github.com/bigbio/sdrf-skills

- `bigbio` = **组织（organization）**，类似公司账号
    
- `sdrf-skills` = 仓库名
    

你也接触到了：

- `bigbio/proteomics-sample-metadata` —— SDRF 规范仓库（spec 的本体）
    
- `asierlarrea/sdrf-skills` —— Asier **fork** 的副本
    

---

## 2. Fork（分叉）

一个账号把别人的仓库**复制一份到自己名下**，获得完全写权限，在自己的副本里折腾不影响原仓库。

bigbio/sdrf-skills    ──fork──▶    asierlarrea/sdrf-skills

  （主仓库，受保护）                 （Asier 的沙盒，随便改）

PR #9 截图里你看到：

asierlarrea wants to merge 1 commit into bigbio:main from asierlarrea:fix/olink-columns-and-strict-improve

翻译：Asier 想把**自己 fork 里的 `fix/olink...` 分支**合回**主仓库的 `main` 分支**。

---

## 3. Branch（分支）

分支 = 同一个仓库里的一条**并行开发线**。代码可以在不同分支上独立演进，互不干扰，最后再合并。

你目前看到的分支：

|分支名|所在仓库|角色|
|---|---|---|
|`main`|`bigbio/sdrf-skills`|主干，发布用|
|`claude/ai-metadata-annotation-tools-9j5pq`|`bigbio/sdrf-skills`|PR #6 用，上面有 tools/ 代码|
|`fix/olink-columns-and-strict-improve`|`asierlarrea/sdrf-skills`|PR #9 用|
|`pr-9`（你本地）|你自己的 clone|你为了 review 拉下来的临时分支|

本地命令：

powershell

git branch --show-current   # 看当前分支

git checkout <分支名>        # 切换分支

git branch -a               # 列出所有分支

---

## 4. Commit（提交）

一次"代码快照 + 改动说明"。每个 commit 有一个独一无二的 SHA 哈希，比如 `6b3c053`（截图里你看到的就是 PR #9 那次 commit）。

commit 由三部分组成：

- **Hash**：`6b3c053522dfa815...`（通常简写前 7 位）
    
- **Message**：`Fix spec submodule reference for CI checkout`（commit 时写的一句话）
    
- **Diff**：具体改了哪些文件哪些行
    

PR #9 里只有 **1 个 commit**，就是这条。

---

## 5. Pull Request（PR，拉取请求）

简称"合并请求"。某人说："我在某分支改好了，请审阅并合并到目标分支"。

PR 页面的结构（对应你看到的截图）：

|Tab|含义|
|---|---|
|**Conversation**|PR 描述 + 评论区。`@lei deng` 这种 mention、你的 review 意见都在这里|
|**Commits**|这个 PR 包含哪些 commit（PR #9 只有 1 个）|
|**Checks**|CI（自动化测试）跑的结果 ✅/❌|
|**Files changed**|所有改动的代码 diff（PR #9 只有 1 行：submodule 指针）|

PR 状态：

- **Open** = 还在审阅
    
- **Merged** = 已合并
    
- **Closed** = 关闭未合并
    

---

## 6. Diff（差异）

两个版本之间的逐行对比。GitHub 用**绿色 `+` 表示新增、红色 `-` 表示删除**。

PR #9 的 diff 你已经见过：

diff

-Subproject commit 61daedacdd866778b3eecd4642b1150d12cfefe8

+Subproject commit fdb84c4deb7353a5a73e07e2a3b5b12186e5ecf7

翻译：把 spec 子模块的指针从旧 commit 换成新 commit，仅此而已。

---

## 7. Submodule（子模块）⭐️ 这是 PR #9 的核心概念

**一个 git 仓库里嵌套另一个 git 仓库**。被嵌套进来的仓库就叫 submodule。

sdrf-skills 里的 `spec/` 目录就是一个 submodule，它实际上是 `bigbio/proteomics-sample-metadata` 这个独立仓库。

### 为什么这么设计？

- `sdrf-skills` 是**工具**代码
    
- `proteomics-sample-metadata` 是**规范**文档 + 示例数据
    
- 工具要依赖规范，但两个项目独立演进，所以用 submodule 建立"引用"而不是拷贝代码
    

### submodule 在主仓库里存的是什么？

**不是文件，只是一个 commit 指针**。主仓库只记"我依赖 spec 仓库的 `61daeda` 这个 commit"。clone 主仓库时要额外执行：

powershell

git submodule update --init --recursive

git 才会去把 spec 仓库克隆到 `spec/` 目录，并 checkout 到指定 commit。

### PR #9 的本质就是改这个指针

`61daeda`（已失效的旧 commit） → `fdb84c4`（活着的新 commit）。

---

## 8. CI / GitHub Actions（持续集成）

**每次 push 或提 PR 时，GitHub 自动跑一套脚本**（通常是跑测试、打包、部署）。

- 配置文件在仓库里的 `.github/workflows/*.yml`
    
- 结果显示在 PR 的 **Checks** tab
    
- ✅ 绿勾 = 通过，❌ 红叉 = 失败
    

PR #9 commit message "Fix spec submodule reference for **CI checkout**" 意思就是：

> 旧指针让 CI 在 `git submodule update` 这一步失败，换新指针后 CI 能正常 checkout 到 spec 代码，后续测试才能跑。

---

## 9. Merge（合并）

PR 审核通过后，把 head 分支的 commit 引入 base 分支。三种方式：

|方式|结果|
|---|---|
|**Merge commit**|保留完整历史 + 一个合并 commit|
|**Squash and merge**|把 PR 的所有 commit 压成一个再合（最常用）|
|**Rebase and merge**|线性历史、无合并点|

---

## 10. Code Review（代码审阅）⭐️ 你的当前任务

在 PR 的 **Files changed** tab 你可以：

1. **在具体代码行旁点 `+` 号** → 发起**行级评论**（指向具体一行）
    
2. 点右上角 **Review changes** → 给整体评价，并选择：
    
    - **Comment**：只留言，不表态
        
    - **Approve** ✅：同意合并
        
    - **Request changes** ❌：要求作者修改后再审
        

review 的思维顺序（适合新手）：

1. 看 PR description → 作者想解决什么问题？

2. 看 commits → 改动被拆成几步？

3. 看 files changed → 具体代码是什么？

4. 自己 checkout 下来跑测试 / 跑工具 → 验证

5. 写评论 → 用"blocking / non-blocking / nit / praise"分层

---

## 把所有概念串起来：PR #9 完整故事

1. bigbio/proteomics-sample-metadata (spec 仓库)

   有人 rebase 了分支，导致旧 commit 61daeda 变成孤儿

2. bigbio/sdrf-skills (工具仓库) main 分支

   .gitmodules 里 spec/ 指向 61daeda → git submodule update 失败 → CI 红了

3. Asier fork 了 sdrf-skills 到 asierlarrea/sdrf-skills

4. 在 fork 的 fix/olink-columns-and-strict-improve 分支上

   改了一行：把指针换成活着的 fdb84c4

5. 提交 PR #9：asierlarrea:fix/olink... → bigbio:main

6. GitHub Actions 自动跑：✅

7. 你（reviewer）checkout 下来验证 → 发现新 spec 带进来

   两个示例 SDRF，有若干质量问题 → 写评论

8. 维护者看评论，要么 approve merge，要么让 Asier 改

---

## 建议你接下来做的两件事

1. 去 PR #9 页面每个 tab 点一圈，把上面列的 10 个概念**对照着看一眼**真实样子
    
2. 看 PR #6（`ypriverol` 自己那个大 PR）感受一下"正常规模"的 PR 长什么样（10 个文件 vs PR #9 的 1 行）
    

有任何一个概念想深入讲（比如 "rebase vs merge 到底差在哪"、"为什么要用 fork 而不是直接给我写权限"），告诉我继续展开。

Feedback submitted
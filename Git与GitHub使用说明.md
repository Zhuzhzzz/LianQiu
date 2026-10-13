# Git 与 GitHub 使用说明：从 VS Code 到终端

适用环境：Windows、PowerShell、VS Code。示例项目：`D:\Work\Lianqiu`；远程仓库：[Zhuzhzzz/LianQiu](https://github.com/Zhuzhzzz/LianQiu)。更新日期：2026-10-13。

本文将“VS”按 **VS Code** 讲解；Visual Studio 是另一款 IDE，Git 原理相同，但菜单位置不同。本文中的命令是操作示例，按对应场景运行，不要从头到尾一次性粘贴。

## 1. 先认清四位角色

| 名称 | 是什么 | 负责什么 | 形象理解 |
| --- | --- | --- | --- |
| VS Code | 编辑器 | 写文件、看差异、通过界面调用 Git | 工作台和操作面板 |
| 终端 | 命令交互窗口 | 接收输入，交给 PowerShell 等 Shell 执行 | 指令窗口 |
| Git | 本地版本管理工具 | 记录版本、比较修改、管理分支、合并历史 | 带时间轴的档案管理员 |
| GitHub | 托管 Git 仓库的平台 | 保存远程历史、共享项目、审查改动 | 联网的档案馆和协作中心 |

你可以在 VS Code 里打开终端，也可以单独打开 PowerShell。只要位于同一个项目目录，操作的就是同一份 Git 仓库。VS Code 的“提交”按钮和终端里的 `git commit`，都是让 Git 工作。

Git 可以离线提交；向 GitHub 推送通常需要网络和登录权限。安装 VS Code、安装 Git、注册 GitHub 账号，是三件不同的事。VS Code 自带 Git 界面，但仍需要机器上安装 Git。[VS Code 官方说明](https://code.visualstudio.com/docs/sourcecontrol/overview)

## 2. 保存、暂存、提交、推送分别发生了什么

把一次改动想成“整理材料并送到档案馆”：

```text
编辑器中尚未保存的文字
        │ Ctrl+S：写入磁盘
        ▼
工作区：你正在修改的文件
        │ git add：挑选本次要归档的内容
        ▼
暂存区：下一次提交的候选快照
        │ git commit：生成一个正式版本
        ▼
本地仓库：电脑上的提交历史（.git）
        │ git push：传送提交并更新远程分支
        ▼
GitHub 远程仓库：别人或另一台电脑能获取的历史
```

**保存不等于提交，提交不等于上传。** 推送传输的是提交历史，不会替你上传尚未提交的编辑。

`git add` 记录的是执行时的内容。比如先写了第一段并暂存，之后又写第二段，此时提交只会包含暂存过的第一段；想把第二段也提交，需要再次暂存。暂存区因此允许你从杂乱的工作中挑出一个完整主题。[Git 文件状态说明](https://git-scm.com/book/en/v2/Git-Basics-Recording-Changes-to-the-Repository)

Git 在概念上记录项目快照，未变化的内容可以复用。提交还包含作者、说明、父提交等信息，并由提交 ID 标识；沿着父提交可以追溯历史。`.git` 保存本地版本管理数据，别把它当作可随手清理的缓存。[Git 基础原理](https://git-scm.com/book/en/v2/Getting-Started-What-is-Git%3F)

## 3. 常见名词：地址、分支和书签

| 名词 | 含义 | 本项目示例 |
| --- | --- | --- |
| repository / repo | 仓库，包含项目及版本历史 | LianQiu |
| commit | 一次正式记录的版本 | `494d1b3` 是短提交 ID |
| branch | 指向某条开发线最新提交的可移动引用 | `main` |
| HEAD | 通常指向你当前所在的分支 | 当前在 `main` 上工作 |
| remote | 配置在本地的远程地址 | GitHub 仓库 URL |
| origin | 远程地址的常用别名，可以改名 | 不是 GitHub 账号名 |
| origin/main | 本地记录的远程 main 状态 | 获取远程信息后才会更新，不是实时网页 |
| upstream | 本地分支默认跟踪的远程分支 | 本地 main 跟踪 origin/main |

把分支想成一本书里的书签：新提交出现，书签向前移动；创建分支，相当于多放一个书签，再沿另一条路线继续写。切换分支时，Git 会相应更新工作区，因此切换前要检查未提交的修改。

## 4. 首次准备：检查工具与身份

在 PowerShell 中逐行执行：

```powershell
cd D:\Work\Lianqiu
git --version
git status
git remote -v
git config --get user.name
git config --get user.email
```

用途依次是：进入项目、确认 Git 可用、查看文件状态、确认远程地址、确认提交作者。

如果尚未配置作者，替换下列示例后执行：

```powershell
git config --global user.name "你的名字"
git config --global user.email "你的提交邮箱"
```

`--global` 作用于当前用户的所有仓库；去掉它，则只配置当前仓库。希望 GitHub 正确关联贡献时，使用账号已验证的邮箱，或账号设置中提供的 noreply 邮箱。

**提交作者信息不是登录凭据。** 改 `user.name` 或 `user.email` 不能修复推送认证失败；前者回答“这次提交署谁的名”，后者需要解决“谁有权限写入远程仓库”。

## 5. 日常操作：先学会这一套

### 5.1 开始写之前

```powershell
cd D:\Work\Lianqiu
git status
git pull --ff-only
```

先确认当前分支和文件状态；工作区干净时，再拉取远程更新。当前分支需要已配置 upstream。

`--ff-only` 只允许把本地分支沿现有历史向前移动；本地和远程各自出现新提交时，它会停止，让你明确处理分叉。[git pull 官方说明](https://git-scm.com/docs/git-pull)

### 5.2 写完一个完整主题后

例如修改了日记，保存后执行：

```powershell
git status
git diff -- Dairy/D05.md
git add -- Dairy/D05.md
git diff --cached
git commit -m "docs: 更新 D05 日记"
git push
git status
```

| 命令 | 用途 | 需要关注什么 |
| --- | --- | --- |
| `git status` | 查看当前状态 | 分支是否正确，有没有漏掉或多出的文件 |
| `git diff` | 比较工作区与暂存区 | 检查尚未暂存的修改；新建的未跟踪文件通常不在这里显示 |
| `git add -- 路径` | 暂存指定文件 | `--` 表示后面是路径 |
| `git diff --cached` | 查看暂存区相对上次提交的差异 | 这是本次提交应重点检查的内容 |
| `git commit -m "说明"` | 形成一个本地提交 | 说明应讲清楚做了什么 |
| `git push` | 推送当前分支的提交 | 关注是否被拒绝、是否认证成功 |

第一次推送某个分支，使用 `git push -u origin main` 建立跟踪关系；之后通常可直接 `git push`。

成功后，打开 GitHub 仓库，选择正确分支，检查文件和最新提交。`working tree clean` 只说明本地没有待提交修改，单凭这句话不能证明远程已更新。

## 6. VS Code 界面怎么操作

1. 选择“文件 → 打开文件夹”，打开 `D:\Work\Lianqiu`。
2. 编辑文件，按 `Ctrl+S` 保存。
3. 按 `Ctrl+Shift+G` 打开“源代码管理”。
4. 点击修改过的文件查看差异。通常红色表示删除，绿色表示新增。
5. 点击目标文件旁的 `+`，将它放入“暂存的更改”。
6. 检查暂存内容，在消息框填写例如 `docs: 补充 Git 使用说明`，点击“提交”。
7. 在源代码管理的 `…` 菜单选择“推送 / Push”。菜单文字可能随版本和语言变化。

提交前明确暂存文件，可以清楚控制这一次版本记录的范围。[VS Code 暂存与提交说明](https://code.visualstudio.com/docs/sourcecontrol/staging-commits)

| VS Code 操作 | 对应的 Git 概念或命令 |
| --- | --- |
| 保存 | 将编辑写入磁盘，还没进入 Git 历史 |
| 点击文件看差异 | 查看 diff |
| 文件旁 `+` | `git add` |
| 取消暂存 | `git restore --staged` |
| 提交 | `git commit` |
| 推送 | `git push` |
| 拉取 | `git pull` |
| 同步更改 | 组合执行拉取与推送 |

“同步更改”包含两个方向的操作，也可能触发合并或冲突处理。初学时分别使用“拉取”和“推送”，更容易理解当前发生了什么。[VS Code 远程仓库说明](https://code.visualstudio.com/docs/sourcecontrol/repos-remotes)

打开终端可以用菜单“终端 → 新建终端”。图形界面适合查看差异、选择文件；终端适合精确操作和复制完整报错。两种方式可以交替使用。

## 7. 新项目与已有远程仓库：选对起点

### 场景 A：本地已有文件，GitHub 仓库为空

只对尚未初始化的项目使用以下流程。GitHub 新建仓库时，若准备推送本地已有项目，先不勾选生成 README、License 等文件，可以避免出现两套独立初始历史。

```powershell
cd D:\Work\Lianqiu
git init -b main
git remote add origin https://github.com/Zhuzhzzz/LianQiu.git
git add -- Dairy/D05.md
git commit -m "docs: 添加 D05 日记"
git push -u origin main
```

`git init` 创建本地仓库；`remote add` 只是记住远程地址，并不会上传文件，也不会在 GitHub 上创建仓库。

**本项目已经初始化并配置远程，不需要再次执行这一整段。**

### 场景 B：GitHub 已经有内容，另一台电脑要接着写

在准备存放项目的父目录运行：

```powershell
cd D:\Work
git clone https://github.com/Zhuzhzzz/LianQiu.git LianQiu-copy
cd LianQiu-copy
git status
```

目标文件夹必须不存在或为空。`clone` 会取得项目和历史，并配置远程；直接下载 ZIP 通常只有文件快照，没有 `.git` 历史。

如果本地有尚未纳入远程的文件，先克隆到新目录，再有选择地复制文件、检查差异、提交。不要直接把远程已有历史强行覆盖掉。

## 8. fetch、pull、push：方向与影响

```text
GitHub 的提交 ── fetch ──► 本地记录的远程状态（如 origin/main）
                                  │
                             merge / rebase
                                  ▼
                             当前本地分支

当前本地分支的提交 ── push ──► GitHub 对应分支
```

`fetch` 是“先把对方的进度拿回来看看”，通常不会改变当前工作文件；`pull` 是先 fetch，再按参数和配置整合到当前分支；`push` 是把本地提交送到远程。rebase 会重放提交并生成新的提交 ID，初学者处理共享分支时优先理解 merge。

想先看远程变化再决定，可执行：

```powershell
git fetch origin
git log --oneline --graph --decorate --all -15
```

## 9. 分支与 Pull Request：把试验和主线分开

个人简短日记可以直接提交到 `main`。涉及较大改动或多人合作时，创建分支更便于独立检查。

从干净的工作区开始：

```powershell
git switch main
git pull --ff-only
git switch -c docs/git-guide
```

完成编辑并保存后：

```powershell
git add -- Git与GitHub使用说明.md
git diff --cached
git commit -m "docs: 添加 Git 与 GitHub 使用说明"
git push -u origin docs/git-guide
```

然后到 GitHub 创建 Pull Request（PR），以 `main` 为目标分支，说明改了什么、为什么改、如何检查。查看 Files changed，确认后合并。

PR 是平台上的审查与合并请求，不是 `git pull` 命令。推送分支也不会自动把它合并进 `main`。

PR 合并后，回到本地：

```powershell
git switch main
git pull --ff-only
```

分支合并时，Git 会根据共同祖先与两边改动计算结果；同一处出现无法自动协调的修改时，才需要人做决定。[Git 分支与合并原理](https://git-scm.com/book/en/v2/Git-Branching-Basic-Branching-and-Merging)

## 10. 遇到冲突怎么办

假设本地和远程都修改了同一段日记，普通推送被拒绝，并且 `git pull --ff-only` 提示无法快进。

先提交需要保留的本地改动，确认工作区干净，再执行：

```powershell
git fetch origin
git merge origin/main
```

若自动合并成功，检查结果并推送。若出现冲突：

1. 运行 `git status` 找到冲突文件。
2. 在 VS Code 中打开文件或合并编辑器，逐段判断要保留什么。
3. 编辑成最终正确内容，删除冲突标记，保存。
4. 对解决好的文件执行 `git add -- 文件路径`。
5. 检查 `git diff --cached`，运行 `git commit -m "merge: 合并远程更新"`，再 `git push`。

冲突文件可能出现：

```text
<<<<<<< HEAD
本地当前分支的文字
=======
待合入分支的文字
>>>>>>> origin/main
```

“接受当前”“接受传入”“接受两者”都是编辑辅助，不能代替内容判断。例如两边各修正了不同信息，可能需要重新组织成一句话。

如果想退出正在进行且尚未完成的合并，可执行 `git merge --abort`。从干净工作区开始合并，更容易安全退出；不要在不理解结果时使用强制推送解决冲突。

## 11. 撤销操作：先分清改动在哪里

| 情况 | 操作 | 影响 |
| --- | --- | --- |
| 文件已暂存，想重新选择提交内容 | `git restore --staged -- Dairy/D05.md` | 取消暂存，保留工作文件修改；适用于已有提交的仓库 |
| 确定不要某文件的未暂存修改 | `git restore -- Dairy/D05.md` | 用暂存区版本覆盖工作文件，未暂存内容会丢失 |
| 某个普通提交已共享，想反向撤销 | `git revert 提交ID` | 创建一个抵消该提交的新提交，之后需要 push；可能发生冲突 |
| 临时切换任务，当前修改还不想提交 | `git stash push -u -m "临时保存"` | 暂存已跟踪修改和未跟踪文件，忽略文件不包含在内 |
| 找回临时保存的修改 | `git stash pop` | 应用最近一次 stash；有冲突时需处理，且 stash 会保留 |

`git restore` 恢复工作文件时可能丢弃内容，操作前先看 `git diff`。Git 通常无法恢复从未提交、从未暂存且被覆盖的文字。[git restore 文档](https://git-scm.com/docs/git-restore)

`git reset --hard`、`git clean -fd`、`git push --force` 都可能丢弃文件或改写共享历史，不作为日常“修复一下”的通用命令。需要它们时，先确认具体目标和备份方式。

## 12. GitHub 登录失败：本次问题怎么处理

典型报错：

```text
Invalid username or token
Password authentication is not supported for Git operations
```

意思是远程认证失败，本地提交仍然存在。GitHub 的 HTTPS Git 操作不能直接使用账号密码；可以通过 Git Credential Manager 的浏览器授权，或使用合适的个人访问令牌（PAT）。[GitHub 令牌说明](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens)

当前 Windows 环境已配置 Git Credential Manager，可按以下步骤重新认证：

1. 开始菜单搜索“凭据管理器”。
2. 打开“Windows 凭据 → 普通凭据”。
3. 找到对应 GitHub 账号的失效条目，例如 `git:https://github.com`，删除该条目；其他使用这条凭据的仓库之后也可能要求重新登录。
4. 在 PowerShell 进入项目，执行 `git push -u origin main`。
5. 若出现登录窗口，选择浏览器登录，使用拥有仓库写入权限的账号完成授权。
6. 回到终端检查推送结果。

这是 GitHub 官方针对 Windows 缓存错误凭据提供的处理路径。[凭据缓存说明](https://docs.github.com/en/get-started/git-basics/caching-your-github-credentials-in-git?platform=windows)

如果未弹出浏览器，记录完整报错再定位。不要把令牌写进仓库 URL、脚本或文档。使用细粒度 PAT 时，选择所需仓库及完成推送所需的权限，通常包括 Contents 读写；组织规则、工作流文件等场景可能有额外要求。

SSH 是另一种认证方式，通过本机私钥与 GitHub 上登记的公钥配合使用；仅把 HTTPS 地址改成 SSH 地址，不会自动完成密钥配置。

## 13. 常见报错速查

| 提示 | 通常原因 | 下一步 |
| --- | --- | --- |
| `not a git repository` | 当前目录不在仓库中 | 先 `Get-Location`，再进入正确目录；新项目才需要 init |
| `nothing to commit, working tree clean` | 没有待提交改动 | 检查是否保存、是否在正确目录；它不代表已推送 |
| `no changes added to commit` | 文件改了但尚未暂存 | 检查后 `git add -- 文件路径` |
| `remote origin already exists` | 已配置 origin | 用 `git remote -v` 检查；确需改地址才用 `git remote set-url origin 新地址` |
| `current branch ... has no upstream branch` | 新分支未建立跟踪关系 | 用 `git push -u origin 当前分支名` |
| `non-fast-forward` / `fetch first` | 远程有本地未整合的提交 | fetch、检查历史，再按第 10 节整合 |
| `Authentication failed` | 凭据无效或过期 | 按第 12 节重新认证 |
| `Repository not found` | 地址错误、仓库不存在或无访问权限 | 检查 URL，并用正确账号验证访问权限 |
| `Permission denied (publickey)` | SSH 认证失败 | 检查密钥与 GitHub 公钥配置；不是 HTTPS 密码问题 |
| `src refspec main does not match any` | 尚无提交，或分支名并非 main | 查看 `git status`、`git branch`、`git log -1` |
| `LF will be replaced by CRLF` | Windows 换行转换提醒 | 通常不是失败；与团队统一换行规则，避免整文件无意义变化 |

## 14. 日常规范：让历史能看、能查、能接着用

### 提交要围绕一个主题

“更新日记”和“大规模重命名目录”尽量拆开。每次提交应能用一句话说明用途，方便以后比较、追溯和撤销。

可采用以下说明格式，这是一种项目约定，并非 Git 强制要求：

```text
docs: 添加 Git 与 GitHub 使用说明
fix: 修正日期显示错误
feat: 增加日记目录生成器
chore: 调整忽略规则
```

避免只写 `update`、`修改`、`123`。修改了程序时，提交前运行与变更有关的检查；修改文档时，检查链接、命令和排版。

### 先检查，再暂存，再推送

初学阶段优先 `git add -- 具体路径`；`git add .` 会把当前目录及其子目录中符合条件的新增、修改、删除一起暂存，使用前要确认范围。

本仓库包含日记，推送前要确认内容适合仓库的公开范围，检查是否包含账号凭据、个人隐私或不该公开的信息。仓库可见性以 GitHub 实际设置为准。

### 用 .gitignore 排除不该归档的文件

下面只是示例，需要时在项目根目录创建 `.gitignore`，按实际情况取舍：

```gitignore
# 本地密钥配置
.env
.env.*
!.env.example

# 临时输出
*.log
*.tmp
```

`.env.example` 只能放占位值，不能放真实密钥。程序项目中的依赖目录、构建产物也可按实际用途忽略。

`.gitignore` 通常用于未跟踪文件，不能让已经提交的文件自动消失。对已跟踪文件停止跟踪，可在确认后用 `git rm --cached -- 文件路径`，再提交；它保留本地工作文件，但历史中的旧内容仍然存在。[GitHub 忽略文件说明](https://docs.github.com/en/get-started/git-basics/ignoring-files)

如果密钥已经泄露，先撤销或轮换密钥，再处理仓库历史。仅删除当前文件无法清除旧提交中的秘密。[GitHub 敏感数据处理说明](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/removing-sensitive-data-from-a-repository)

### 多台电脑遵守同一节奏

开始工作前拉取，结束一个完整主题后提交并推送。Git 不会自动同步所有文件；切到另一台电脑前确认改动已经推送，另一台再拉取。

## 15. 一页操作卡

**查看当前情况：**

```powershell
git status
git remote -v
git log --oneline -5
```

**开始工作（工作区干净，已配置 upstream）：**

```powershell
git pull --ff-only
```

**完成修改（把路径和说明换成实际内容）：**

```powershell
git diff
git add -- Dairy/D05.md
git diff --cached
git commit -m "docs: 更新 D05 日记"
git push
git status
```

记住这条路线：**编辑并保存 → 检查差异 → 暂存 → 提交 → 推送 → 检查远程结果。**

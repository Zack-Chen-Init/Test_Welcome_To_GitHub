# Git 操作笔记（以 Test_Welcome_To_GitHub 为案例）

> 这份笔记按你实际走过的路径写：从「本地一个文件夹」到「GitHub 上有个能看的历史、分支、标签」。
> 每条命令都标了：**干什么 / 你会看到什么 / 为什么**。
> 案例仓库：https://github.com/Zack-Chen-Init/Test_Welcome_To_GitHub
> 本地目录：`D:\STM32_Project\STM32_Watch\1_1_UI_Test_GitHub`

---

## 0. 一次性配置（只需做一次）

### 0.1 确认 Git 能用

```powershell
git --version
```

看到 `git version 2.x.x.windows.1` 就正常。
如果报「无法将 git 项识别为 cmdlet」，说明这个终端窗口的 PATH 是旧的：**关掉窗口重开**，或者用全路径 `"D:\Git\cmd\git.exe" --version`。

### 0.2 告诉 Git 你是谁

```powershell
git config --global user.name "Zack-Chen-Init"
git config --global user.email "1096470140@qq.com"
```

**作用**：每次 commit 都会把这两个信息写进提交记录里，GitHub 靠 email 判断「这个提交是谁的」。
**看不到输出就是成功。** 查看当前配置：

```powershell
git config --list
```

### 0.3 常用全局项

```powershell
git config --global init.defaultBranch main    # 新仓库默认分支叫 main，不再是 master
git config --global core.autocrlf true         # Windows 换行符自动适配，避免满屏 diff
git config --global core.quotepath false       # 中文文件名正常显示，不显示成 \346\226\207
git config --global core.sshCommand "D:/Git/usr/bin/ssh.exe"   # 用 Git 自带 SSH（见 0.4）
```

### 0.4 SSH 连接 GitHub（一次性）

**为什么用 SSH 不用 HTTPS**：HTTPS 每次要输账号密码/令牌，还要处理代理；SSH 用一对密钥，配好一次以后 `push` 全自动。

```powershell
ssh-keygen -t ed25519 -C "1096470140@qq.com"
```

- 一路回车：默认存到 `C:\Users\<你>\.ssh\id_ed25519`，不设密码短语。
- 生成两个文件：`id_ed25519`（**私钥，绝不外传**）、`id_ed25519.pub`（公钥，可以给任何人看）。

把公钥内容贴到 GitHub：https://github.com/settings/ssh/new
（记事本打开 `id_ed25519.pub` 全选复制，Title 随便写）

**验证**：

```powershell
ssh -T git@github.com
```

成功会看到：`Hi Zack-Chen-Init! You've successfully authenticated, but GitHub does not provide shell access.`
看到 `Permission denied (publickey)` = 公钥没生效（没添加成功，或者加错了账号）。

> 坑：Windows 自带的 `ssh`（OpenSSH 9.5）连 GitHub 会报
> `choose_kex: unsupported KEX method sntrup761x25519-sha512@openssh.com`。
> 解法就是上面那条 `core.sshCommand`，让 Git 用自带的 `D:\Git\usr\bin\ssh.exe`。

---

## 1. 从「一个文件夹」到「联网的仓库」

### 1.1 建远程仓库时的一个选择

在 GitHub 点 **New repository** 时，**不要勾** Add a README / .gitignore / License。
勾了 = 远端先有了一个提交，而你的本地没有，第一次 push 会被拒绝（多一个 `--force` 的麻烦）。

案例里就踩了这个：远端有 `0aeef0f Initial commit`，本地是空的。

### 1.2 本地初始化

```powershell
cd D:\STM32_Project\STM32_Watch\1_1_UI_Test_GitHub
git init
```

**作用**：在这个文件夹里建一个隐藏的 `.git` 目录，从此这里叫「工作区」，Git 开始记录。
**你会看到**：`Initialized empty Git repository in D:/.../.git/`

### 1.3 先写 .gitignore（这一步别跳）

```powershell
git status
```

**作用**：列出「当前有哪些文件，各自什么状态」。
**你会看到**：一长串红色 `Untracked files:`，包括 `Objects/`、`Listings/`、`Project.uvguix.Admin`。

**为什么要过滤**：
- `Objects/`、`Listings/` 是 Keil 每次编译重新生成的产物，几十 MB，传上去纯垃圾；
- `*.uvguix.*` 记录的是你本机的窗口布局，跟代码无关；
- `keilkill.bat` 这类工具脚本可以传，也可以忽略，看你习惯。

新建 `.gitignore`（注意前面有个点），内容：

```gitignore
Objects/
Listings/
*.uvguix.*
*.axf
*.hex
*.map
*.o
*.d
*.crf
Thumbs.db
```

**规则**：`目录/` 忽略整个目录；`*.扩展名` 忽略所有该类型文件；`#` 开头是注释。

写完再 `git status`，`Objects/`、`Listings/`、两张 uvguix 应该都不见了，` .gitignore` 自己出现（它也要被提交）。

> 坑：PowerShell 里 `notepad` 报 `CommandNotFoundException`，是当前窗口的 PATH 被 conda 改坏了。
> 换 `& "C:\Windows\System32\notepad.exe" .gitignore`，或者直接 `code .gitignore`。

### 1.4 暂存 → 提交（第一张快照）

```powershell
git add -A
```

**作用**：把「所有改动（新增/修改/删除）」放进暂存区（篮子）。`-A` = all。
**没输出就对了**。想看放进了什么：`git status`（绿色 `Changes to be committed`）。

```powershell
git commit -m "first commit: 1_1 UI 基础工程"
```

**作用**：把篮子里的内容打包成一个**不可修改**的版本（提交），带一条说明。
**你会看到**：`[main (root-commit) a868380] first commit: ...` + `86 files changed`。
- `root-commit` = 这是第一张快照，没有父提交；
- `main` = 记在 main 这条线上；
- `a868380` = 提交编号（短哈希，全局唯一，7 位足够）。

**`-m` 后面写什么**：一句话说清「这个版本加了什么」。常用前缀：
`feat:` 新功能 / `fix:` 修 bug / `docs:` 文档 / `refactor:` 重构 / `chore:` 杂务。

### 1.5 连接远程仓库

```powershell
git remote add origin git@github.com:Zack-Chen-Init/Test_Welcome_To_GitHub.git
```

**作用**：给远程仓库起个别名 `origin`，以后不用每次写长地址。
**没输出就对了，这一步一辈子只做一次。**

```powershell
git remote -v
```

**检查**：两行，`(fetch)` 和 `(push)`，地址都是 `git@github.com:...`。

### 1.6 第一次 push

```powershell
git push -u origin main
```

- `push` = 把本地提交**上传**到远程；
- `origin` = 传去哪（1.5 起的名）；
- `main` = 传哪个分支；
- `-u` = 记下「本地 main ↔ 远程 main」的对应关系，以后只写 `git push` 就行。

**第一次大概率被拒**：

```
! [rejected]  main -> main (fetch first)
hint: Updates were rejected because the remote contains work that you do not have locally.
```

**原因**：远端那个自动生成的 `Initial commit`（README）不在你的历史里，两条线没有共同祖先，Git 不敢覆盖。

**解法**（仅在「仓库是你的、远端没有别的东西」时）：

```powershell
git push -u origin main --force
```

成功：`+ 0aeef0f...a868380 main -> main (forced update)`

> ⚠️ `--force` 的含义是「我不管远端有什么，全用我的覆盖」。有协作者时绝对禁止，会抹掉别人的提交。
> 想彻底避免：建仓库时别勾 Add a README。

### 1.7 验证

```powershell
git log --oneline
```

**作用**：一行一个提交，看版本清单。新→旧排列。
刷新 GitHub 页面，文件全在，Commits 有一条记录。

---

## 2. 日常四步（90% 的时间只用这四条）

```powershell
git pull                          # ① 开工先同步远端的改动
# ... 改代码 ...
git status                        # ② 看改了哪些文件
git diff                          # ③ 看具体改了哪几行（+ 新增，- 删除）
git add -A                        # ④ 挑进篮子
git commit -m "feat: 说清楚做了什么"  # ⑤ 存档
git push                          # ⑥ 上线
```

**`git status` 的信号灯**：
- 红色 `Untracked` = 新文件，Git 还没管；
- 红色 `modified` = 改过但没 add；
- 绿色 `Changes to be committed` = 已进篮子，等着 commit；
- `nothing to commit, working tree clean` = 和上次提交完全一致（没有未保存的东西）。

**`git status` 的 ahead/behind**：

```
Your branch is ahead of 'origin/main' by 1 commit.
```

`ahead 1` = 本地有一个提交没推上去（网页上当然看不到）。
反过来 `behind` = 远端有新的，你该 `git pull` 了。

---

## 3. 看历史 / 看差异

| 命令 | 作用 | 典型输出 |
| --- | --- | --- |
| `git log --oneline` | 版本清单（新→旧） | `a868380 first commit: ...` |
| `git log --oneline --graph` | 带分支分叉的画图模式 | 见第 6 节 |
| `git log --oneline -3` | 只看最近 3 条 | — |
| `git show <提交ID>` | 某个提交改了哪些文件、每行改动 | `+` 新增 / `-` 删除 |
| `git diff` | 工作区 vs 上次提交 | 同上 |
| `git diff <旧ID> <新ID>` | 两个版本之间的差异 | 同上 |
| `git log --stat` | 每个提交改了哪些文件、多少行数 | `x files changed` |

> 注意 `--graph` 别打错成 `--grass`，Git 会报 `fatal: unrecognized argument: --grass`。
> 命令打错只会报错，**不会损坏仓库**，放心重敲。

---

## 4. 分支（并行时间线）

**一句话**：分支 = 从当前版本分出去的一条平行线。在分支上随便改，`main` 一动不动。

```powershell
git branch
```

**作用**：列出本地分支，`*` 是当前所在。

```powershell
git switch -c feature/readme-notes
```

**作用**：新建分支并切过去。`-c` = create。
（老教程的 `git checkout -b feature/readme-notes` 效果完全一样，别慌。）
命名习惯：`feature/xxx` 新功能、`fix/xxx` 修 bug、`test/xxx` 试验。

在分支上改文件、`git add -A`、`git commit`，**只影响这条分支**。

```powershell
git switch main        # 切回 main
dir notes.md           # 分支上新建的文件在这里不存在 —— 隔离的证据
git switch feature/readme-notes
dir notes.md           # 切回来又出现了
```

```powershell
git branch -vv
```

**作用**：列出各分支、指向哪个提交、跟踪哪个远端分支、领先/落后几个。
输出里 `[origin/main: ahead 1]` 就是「本地比远端多一个提交」。

```powershell
git push -u origin feature/readme-notes
```

**作用**：把分支也传到 GitHub（否则远端看不到它）。

---

## 5. 合并：Pull Request（PR）

**PR 的本质**：「请把我这条分支的改动合并进 main」。这是 GitHub 的网页流程，把你合并的每一步都留成记录，可以讨论、审查。

**流程**：

1. 分支页面或仓库首页出现黄色条 → 点 **Compare & pull request**
   （没有就点 **Contribute** → **Open pull request**）
2. 确认 `base: main ← compare: feature/readme-notes`，看到 `✓ Able to merge`
3. **Create pull request**
4. **Merge pull request** → **Confirm merge**
5. 出现紫色 `Merged` = 成功；旁边 **Delete branch** 可删远端分支

**三种合并方式**（下拉框里选）：

| 方式 | 结果 |
| --- | --- |
| Create a merge commit（默认） | 多一个「合并提交」，历史能看出分叉和合并 |
| Squash and merge | 分支上所有提交压成 1 个提交，main 历史最干净 ← 个人项目推荐 |
| Rebase and merge | 把分支提交挪到 main 末尾，没有合并提交，一条直线 |

**合并完必须回本地同步**：

```powershell
git switch main
git pull          # 把远端的合并结果拉下来
dir notes.md      # 现在能找到了
```

然后清理：

```powershell
git branch -d feature/readme-notes              # 删本地分支（-d 会检查是否已合并）
git push origin --delete feature/readme-notes   # 删远端分支
```

> 如果 `-d` 报 `not fully merged`，说明那条线的提交确实还没进 main；确认不要了再用 `-D` 强删。

**什么情况会冲突**：两条线改了**同一个文件的同一处**，Git 不知道听谁的，会在文件里插 `<<<<<<<` 标记让你手工选。案例里分支是从最新 main 长出来的，所以是快进/合并，无冲突。

---

## 6. 标签（版本里程碑）

标签 = 给某个提交起一个人能记住的名字，比如 `v1.0`、`v1.1`。

```powershell
git tag -a v1.1 -m "1_1 UI 基础工程完成"
git push origin v1.1
git tag            # 列出所有标签
```

**作用**：`-a` = 带说明的标签，会记录作者和日期。
刷新仓库首页 → 右侧 **Releases / Tags** 出现 `v1.1`。

**回到某个版本看代码**（只看不改）：

```powershell
git switch --detach v1.1
git switch main          # 看完必须回来
```

> `--detach` 状态下**不要改代码再 commit**，那些提交不属于任何分支，切走就找不到了。

---

## 7. 撤销速查

| 场景 | 命令 | 说明 |
| --- | --- | --- |
| 改了文件想还原，还没 add | `git restore 文件名` | 丢弃工作区改动 |
| add 了想撤回暂存 | `git restore --staged 文件名` | 只拿回篮子，改动还在 |
| 提交信息写错（未推送） | `git commit --amend -m "新的"` | 改写最后一次提交 |
| 漏了一个文件（未推送） | `git add 文件` 然后 `git commit --amend --no-edit` | 并进上次提交 |
| 想看某提交改了什么 | `git show <提交ID>` | — |
| 误删文件 | `git restore 文件路径` | 从最新提交恢复 |

> **已 push 的提交不要用 `reset` 改历史**（会让别人/远端和你不一致）。
> 要撤销就**新提交一次反向修改**，然后正常 push。

---

## 8. 这次案例的真实时间线

```
a868380  first commit: 1_1 UI 测试GitHub工程       ← git init + add + commit + remote + force push
   │
d77e91a  docs: 添加 README                         ← 实验1：只 commit，忘了 push
   │
   ├──────────────────────────────┐
   │                              │
710ca5d docs: 分支上新增 notes.md   │                  ← 实验2：switch -c 新分支 + commit + push
   │                              │
   └──────────► e4ec159 ◄─────────┘
        Merge pull request #1      ← 实验3：网页 PR 合并（你当时以为没合并成功，其实是 README 提交没推）
```

对应的 `git log --oneline --graph`：

```
* e4ec159 (HEAD -> main, origin/main, origin/HEAD) Merge pull request #1 from .../feature/readme-notes
|\
| * 710ca5d (origin/feature/readme-notes, feature/readme-notes) docs: 分支上新增 notes.md
| * d77e91a docs: 添加 README
|/
* a868380 first commit: 1_1_UI_测试GitHub工程
```

**读法**：从下往上，`*` 是提交，竖线是时间线，`|\` `|/` 是分叉点和合并点；一行括号里 `(HEAD -> main, origin/main, ...)` 表示这几个名字指向同一个提交 = 全部同步。

**这次踩的坑**：`d77e91a`（README）只 commit 没 push，所以远端 main 一直停在 `a868380`，分支页面看到的是 3 个提交、main 页面只有 1 个 —— 看起来像「没合并成功」，实际是「本地有提交没上传」。

---

## 9. 常见报错对照

| 报错 | 意思 | 解决 |
| --- | --- | --- |
| `fatal: not a git repository` | 这个目录不是 Git 仓库 | `cd` 到项目目录，或先 `git init` |
| `Updates were rejected ... (fetch first)` | 远端有你没有的提交 | `git pull` 后再 push；确认为空仓库可 `--force` |
| `Permission denied (publickey)` | SSH 公钥没生效 | 检查 `ssh -T git@github.com`，公钥是否加对账号 |
| `choose_kex: unsupported KEX method` | Windows 自带 ssh 太新 | `core.sshCommand` 指向 Git 自带 ssh |
| `fatal: unrecognized argument: --xxx` | 参数拼错 | 重敲，不会损坏仓库 |
| `not fully merged`（删分支时） | 分支提交没进 main | 确认不要了用 `-D` |
| `nothing to commit, working tree clean` | 没有新改动，没东西可提交 | 正常提示，不是错误 |
| `error: pathspec 'xxx' did not match` | 文件名/分支名拼错，或文件不在当前目录 | `dir` 确认一下 |

---

## 10. 一页速查

```powershell
# 一次性
git config --global user.name "名字"
git config --global user.email "邮箱"
ssh-keygen -t ed25519 -C "邮箱"      # 公钥贴到 GitHub

# 新项目
git init
# 写 .gitignore
git add -A
git commit -m "first commit: 说明"
git remote add origin git@github.com:用户名/仓库名.git
git push -u origin main

# 日常
git pull
git status / git diff
git add -A
git commit -m "feat: 做了什么"
git push

# 分支与合并
git switch -c feature/xxx
git push -u origin feature/xxx
# 网页 PR → Merge
git switch main && git pull
git branch -d feature/xxx
git push origin --delete feature/xxx

# 版本
git log --oneline --graph
git tag -a v1.1 -m "说明" && git push origin v1.1

# 撤销
git restore 文件
git restore --staged 文件
git commit --amend -m "新信息"
```

---

## 附：这套流程怎么套到 STM32 项目上

1. 每个版本（1_1、1_2、1_3……）一个文件夹，**只把最新的当仓库根**，历史用 commit 记，不靠复制文件夹；
2. 想保留旧版本，用 `git tag -a v1.1` 打标签，而不是 copy 一份目录；
3. 每次跑通一个功能：`git add -A` → `commit` → `push`，一条提交对应一个能跑的状态，出问题能 `git show` 回看；
4. 试验新功能（比如 MPU6050）：`git switch -c feature/mpu6050`，跑通了再合并回 main，跑不通直接删分支，main 干干净净。

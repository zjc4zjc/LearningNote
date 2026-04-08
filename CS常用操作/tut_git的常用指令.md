# git的常用指令教程

本文记录 `git` 的常用指令。

前置条件：
- `git` 已经安装可用
在终端按行输入：

```bash
git --version
(window请输入)where git
(linux请输入)which git
```

如果有类似的输出：

```text
git version 2.51.0.windows.1
D:\Software\Git\cmd\git.exe
```

说明git能正常调用，否则请查看教程[tut_git的下载安装](tut_git的下载安装.md)

---
## git常用指令

```bash
git status
git init
git add .
git commit -m "说明"
git push
git pull
git log
git diff
git rm -r --cached 文件夹名
git reset --hard 2b1d6bc40200862955c45a77b474d05e4d501f95
```

它们分别是：

- `git status`：查看当前文件夹的git状态
- `git init`：初始化建立一个git追踪
- `git add .`：把当前修改加入暂存区
- `git commit -m "该次修改的说明"`：提交一次修改
- `git push`：把本地提交推到 GitHub
- `git pull`：拉取远程最新内容
- `git log --oneline`：简洁查看提交历史
- `git diff`：看还没提交的改动
- `git rm -r --cached 文件夹名`：取消跟踪一个已提交过的文件夹，但保留本地文件
- `git reset --hard 2b1d6bc40200862955c45a77b474d05e4d501f95`：回到之前的一次提交

---
## 日常最常用工作流

### 1.新建git仓库

如果你新建了一个代码目录，例如：

```text
D:\Onedrive\Github\Preference_Arbiter
```

进入目录后执行：

```bash
git init
```

这会在当前目录创建 `.git`，表示这里开始成为一个 Git 仓库。

检查是否成功：

```bash
git status
```

如果看到类似：

```text
On branch master
```

或：

```text
On branch main
```

就说明初始化成功。

### 2. 连接远程 GitHub 仓库

假设你已经在 GitHub 上新建了一个空仓库，想要与本地的项目连通：

```text
https://github.com/你的用户名/Preference_Arbiter.git
```

在本地仓库目录里执行，可以直接从github上复制这个连接代码：

```bash
git remote add origin https://github.com/你的用户名/Preference_Arbiter.git
git remote -v
```

如果 `git remote -v` 输出了 `origin` 的 fetch 和 push 地址，就说明连接成功。

### 3. 修改后提交与推送

假设改了几个文档或代码文件，标准流程是：

#### 3.1 先看改了什么

```bash
git status
git diff
```
可以看出增删查改的情况
#### 3.2 提交修改

```bash
git add .
git commit -m "docs: update weekly record"
git push
```
add将当前修改加入暂存区，commit提交这次修改，push推送到远程github仓库
#### 3.3 查看最近提交信息

```bash
git log
```
什么人在什么时间提交了什么内容

### 4. `.gitignore` 的作用

`.gitignore` 用来告诉 Git：哪些文件或目录以后不要再跟踪。

例如代码仓库里常见写法：

```gitignore
.idea/
.obsidian/
.vscode/
__pycache__/
*.pyc
Dataset/
outputs/
checkpoints/
```

例如文档仓库里，如果不想把 `常用资料` 这个文件夹及其内容上传到 GitHub：

```gitignore
/常用资料/
```

### 5. 取消仓库中某文件或文件夹的git追踪

```bash
git rm -r --cached -- "常用资料"
git commit -m "chore: stop tracking 常用资料"
git push
```

之后最好在.gitignore里加入这个文件或文件夹，防止在`git add .`里又参与追踪

### 6. 取消整个仓库的git

```bash
(windows终端里)rmdir /s /q .git
(linux终端里)rm -rf .git
```

该操作会直接让整个仓库变成普通文件夹，不再参与git版本控制
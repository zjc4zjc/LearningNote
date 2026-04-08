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
(windows终端里)rmdir /s /q .git
(linux终端里)rm -rf .git
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
- `rmdir /s /q .git`：该操作会直接让整个仓库变成普通文件夹，不再参与git版本控制

---
## 日常最常用工作流

### 1.在github上新建了远程仓库

如果你新建了一个远程repo，例如：

```text
https://github.com/zjc4zjc/PreferenceArbiter.git
```

对应着你本地的文件夹：

```
D:\Onedrive\Github\PreferenceArbiter
```

你可以在本地文件夹路径的终端里，按照github的指引依次输入：

```bash
#将"# PreferenceArbiter"这段文字输入进README.md
echo "# PreferenceArbiter" >> README.md
#git初始化
git init
#新建一个带有上述输入的README.md
git add README.md
#提交这次的改动，注意只是提交不是推送
git commit -m "first commit"
#把当前分支强制重命名为 `main`
git branch -M main
#与远程仓库相连接
git remote add origin https://github.com/zjc4zjc/PreferenceArbiter.git
#如果git remote -v输出了origin的fetch和push地址，就说明连接成功。
git remote -v
#将本地的文件推送至main分支，首次完成后续仅输入git push就可以了
git push -u origin main
```

### 2.已经完成过本地-远程的连接，初次推送后后续快速迭代

本地文件夹路径的终端里执行：

```shell
git status
git diff   #可以不执行
```

如果看到类似：

```
On branch main
Your branch is up to date with 'origin/main'.

nothing to commit, working tree clean
```

说明没有改动的地方。反之则需要将改动加入到git的追踪/承认/推送，依次输入：

```bash
#这次改动加入暂存区
git add .
#提交这次改动
git commit -m"<对于这次改动的描述>"
#推送这次改动到远程repo
git push
```

### 3 查看最近提交信息

```bash
git log
```
什么人在什么时间提交了什么内容，在linux的终端里按`q`退出git log的分页器显示

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

例如文档仓库里，如果不想把 `常用资料` 这个文件夹及其里面的内容上传到 GitHub：

```gitignore
常用资料/
```

### 5. 取消仓库中某文件或文件夹的git追踪

```bash
git rm -r --cached -- "常用资料"
git commit -m "chore: stop tracking 常用资料"
git push
```

之后最好在.gitignore里加入这个文件或文件夹，防止在`git add .`里又参与追踪


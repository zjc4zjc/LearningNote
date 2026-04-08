# git的下载安装教程

本文记录 `git` 的安装、配置说明。

## 1. Windows 安装

### 1.1查看电脑信息

按win+r键(`win键是左下方ctrl的右边那个键`)，输入dxdiag查看电脑位数。比如我的电脑显示：

```text
操作系统:Windows 11专业版64位(10.0,版本22631)
```

### 1.2访问官网下载git包

根据电脑的操作系统位数，比如64位选择Git for Windows/x64 Setup就好

- [https://git-scm.com/downloads](https://git-scm.com/downloads/)

安装包的位置默认在Downloads下载文件夹，安装完成后可直接删除。

### 1.3下载安装

安装时建议有良好的路径管理，务必不要全部软件都放默认的C盘。比如我习惯在D盘或者E盘新建一个Software文件夹，在里面存放所有的软件资源。如果你没有，可以先提前创建一个。安装Git时选择这个路径：

```path
D:\Software\Git
```

一路基本都按默认的来，全点击Next就好，完成安装。

### 1.4确认git能正常调用

按win+r键，输入cmd唤起终端，在终端按行输入：

```bash
git --version
where git
```

如果有类似的输出：

```text
git version 2.51.0.windows.1
D:\Software\Git\cmd\git.exe
```

说明git能正常调用。

若安装后还是显示找不到，大概率未配置git的环境变量，请查看教程[tut_环境变量的配置](tut_环境变量的配置.md)

---

## 2. Linux 远程服务器安装

这里的场景是采用“在conda的某个虚拟环境下载git，再通过添加环境变量使得所有环境都能调用git”的方式。

### 2.1如果你的电脑没有安装conda

请先查看教程：[tut_conda的下载安装](tut_conda的下载安装.md)

### 2.2如果电脑有安装conda

在服务器的终端里输入，新建一个虚拟环境newbie，版本随意：

```bash
conda env list
conda create -n newbie python=3.9.16
```

确认能看见环境，再接着按行输入：

```bash
conda activate newbie
conda install git
```

在弹出来的`y/n`选项中输入y再回车，完成安装。

### 2.3配置环境变量

在服务器的终端里按行输入：

```bash
conda activate newbie
which git
git --version
```

正常来说会显示它的位置与版本号，如：

```bash
~/software/anaconda3/envs/newbie/bin/git
git version 2.51.0
```

如果不显示则说明声明的环境变量有误。

### 2.4 写入shell配置文件

建议上一步显示了位置和版本号后再进行这一步，因为会使用上一步输出的git位置。

上一步只在特定的newbie环境里调用了git，现在希望所有的环境都能调用git，因此需要设置环境变量一行行执行：

```bash
echo 'export PATH="$HOME/software/anaconda3/envs/newbie/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```
这里的：
- `echo` 是声明一个内容，这个内容是单引号里的`export PATH="$HOME/software/anaconda3/envs/newbie/bin:$PATH"`，也就是环境变量
- `>>` 是在文件末尾追加新内容，追加的内容就是 `echo` 声明的内容
- `.bashrc` 是shell的配置文件，也是编辑环境变量的地方
- `source` 起到重新加载的作用

关闭终端，打开新终端验证：

```bash
conda deactivate
which git
git --version
```

若有正常输出，说明安装成功。
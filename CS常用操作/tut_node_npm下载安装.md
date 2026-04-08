# node 和 npm 的下载安装教程

本文记录 `node` 和 `npm` 的安装。

适用场景：

- Windows 本机安装 `node` 和 `npm`
- Linux 远程服务器在没有 `sudo` 的情况下安装 `node` 和 `npm`
## 1. Windows 安装

### 1.1 下载安装包

官方下载安装页：

- [https://nodejs.org/en/download](https://nodejs.org/en/download)

一般选择：

- Windows 64 位机器：下载 `Windows Installer (.msi)` 的 `x64` 版本
- Windows ARM 机器：下载对应的 `arm64` 版本

如果你不确定自己的系统架构，可以在 PowerShell 里查看：

```powershell
$env:PROCESSOR_ARCHITECTURE
```

或者win+r，输入dxdiag看

### 1.2 安装

安装时建议有良好的路径管理，务必不要全部软件都放默认的C盘。比如我习惯在D盘或者E盘新建一个Software文件夹，在里面存放所有的软件资源。如果你没有，可以先提前创建一个。安装Nodejs时选择这个路径：

```powershell
D:\Software\Nodejs
```

一路基本都按默认的来，全点击Next就好。安装完成后，**关闭并重新**打开终端，然后一行行验证：

```bash
where node
where npm
node -v
npm -v
```

如果出现类似输出：
```bash
D:\Software\Nodejs\node.exe

D:\Software\Nodejs\npm
D:\Software\Nodejs\npm.cmd

v22.20.0

10.9.3
```
说明安装成功。

如果安装后终端里仍提示找不到 `node` 或 `npm`：
- 先关闭并重新打开终端
- 再执行一次 `node -v` 和 `npm -v`
- 若安装后还是显示找不到，大概率未配置nodejs的环境变量，请查看教程[tut_环境变量的配置](tut_环境变量的配置.md)


---
## 2. Linux 远程服务器安装

这里的场景是采用“在本机下载官方压缩包，再上传到远程服务器解压安装”的方式。

### 2.1 确认服务器架构

在服务器的终端里执行：

```bash
uname -m
```

常见结果：

- `x86_64`：下载 `Linux x64`
- `aarch64` 或 `arm64`：下载 `Linux arm64`

### 2.2 在本机下载 Node.js 官方压缩包

官方下载安装页：

- [https://nodejs.org/en/download](https://nodejs.org/en/download)

如果需要历史版本，也可以用官方归档页：

- [https://nodejs.org/en/download/archive/v22](https://nodejs.org/en/download/archive/v22)

本文示例使用的文件名是：

```text
node-v22.22.1-linux-x64.tar.xz
```

如果你的服务器不是 `x86_64`，请改为下载对应架构的压缩包。

### 2.3 上传到服务器

在服务器上新建一个目录，专门存放node：

```bash
mkdir -p ~/software/node
```

然后把 `node-v22.22.1-linux-x64.tar.xz` 压缩包从自己的电脑上传到远程服务器的这个目录。可以用 VS Code 文件上传，也可以用 `scp` 或者 `Xftp 8` 等软件。

上传完成后，目录结构类似：

```text
~/software/node/node-v22.22.1-linux-x64.tar.xz
```

### 2.4 解压并配置环境变量PATH

在远程终端一行一行执行，请留意**更改下方文件名，以实际为准**：

```bash
cd ~/software/node
tar -xf node-v22.22.1-linux-x64.tar.xz
mv node-v22.22.1-linux-x64 node-v22
export PATH="$HOME/software/node/node-v22/bin:$PATH"
hash -r
```

这里的：
- `cd` 进入到指定的文件夹，此时终端可以访问到这个文件夹里的文件
- `tar -xf` 起到解压文件夹的作用。
- `mv` 起到重命名文件夹的作用，将 `node-v22.22.1-linux-x64` 重命名为 `node-v22` 以免文件夹名称太长。
- `export` 是临时声明环境变量PATH，方便下一步确认文件的解压没有损坏。
- `hash -r` 清除 Bash 的命令哈希表，提高命令执行效率。

Note：我自己的电脑上：
```bash
$HOME/software/node/node-v22/bin
~/software/node/node-v22/bin
/home/zjc/software/node/node-v22/bin
```
这三者等价，但建议使用 `$HOME` ，会更规范

一行行验证：

```bash
which node
which npm
node -v
npm -v
```

正常来说会显示它俩的位置与版本号，如果不显示则说明声明的环境变量有误。

### 2.5 写入shell配置文件

建议上一步显示了位置和版本号后再进行这一步。

上一步的**临时环境变量**将随着当前终端关闭而消失，导致新打开的终端无法调用 `node` 和 `npm` 。为了让临时环境变量升级为**永久环境变量**，一行行执行：

```bash
echo 'export PATH="$HOME/software/node/node-v22/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```
这里的：
- `echo` 是声明一个内容，这个内容是单引号里的`export PATH="$HOME/software/node/node-v22/bin:$PATH"`，也就是环境变量
- `>>` 是在文件末尾追加新内容，追加的内容就是 `echo` 声明的内容
- `.bashrc` 是shell的配置文件，也是编辑环境变量的地方
- `source` 起到重新加载的作用

关闭终端，打开新终端验证：

```bash
which node
which npm
node -v
npm -v
```

若有正常输出，说明安装成功。

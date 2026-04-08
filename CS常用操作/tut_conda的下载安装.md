# conda的下载安装教程

本文记录 `conda` 的安装、配置说明。

## 1. Windows 安装

### 1.1查看电脑信息

按win+r键(`win键是左下方ctrl的右边那个键`)，输入dxdiag查看电脑位数。比如我的电脑显示：

```text
操作系统:Windows 11专业版64位(10.0,版本22631)
```

### 1.2访问官网下载conda包

根据自己的操作系统 `win/linux` 在官网选择合适的安装包，推荐新手下载 `Anaconda Distribution`  ，官网地址为：

- [https://www.anaconda.com/download/success?reg=skipped](https://www.anaconda.com/download/success?reg=skipped)

安装包的位置默认在Downloads下载文件夹，安装完成后可直接删除。

### 1.3下载安装

安装时建议有良好的路径管理，务必不要全部软件都放默认的C盘。比如我习惯在D盘或者E盘新建一个Software文件夹，在里面存放所有的软件资源。如果你没有，可以先提前创建一个。安装conda时选择这个路径：

```path
D:\Software\anaconda3
```

一路基本都按默认的来，全点击Next就好，完成安装。

### 1.4确认conda能正常调用

按win+r键，输入cmd唤起终端，在终端按行输入：

```bash
conda --version
where conda
```

如果有类似的输出：

```text
conda 25.5.1

D:\Software\anaconda3\condabin\conda.bat
D:\Software\anaconda3\Scripts\conda.exe
```

说明conda能正常调用。

若安装后还是显示找不到，大概率未配置conda的环境变量，请查看教程[tut_环境变量的配置](tut_环境变量的配置.md)

---

## 2. Linux 远程服务器安装

这里的场景是采用“在本机下载官方压缩包，再上传到远程服务器解压安装”的方式。

### 2.1 确认服务器架构

在服务器的终端里执行：

```bash
uname -m
```

常见结果：

- `x86_64`：下载 `64-Bit (x86) Installer`
- `aarch64` 或 `arm64`：下载 `64-Bit (AWS Graviton2 / ARM64) Installer`

### 2.2 在本机下载 anaconda.sh 官方压缩包

官方下载安装页，推荐新手下载 `Anaconda Distribution`

- [https://www.anaconda.com/download/success?reg=skipped](https://www.anaconda.com/download/success?reg=skipped)

本文示例使用的文件名是：

```text
Anaconda3-2025.06-0-Linux-x86_64.sh
```

如果你的服务器不是 `x86_64`，请改为下载对应架构的压缩包。

### 2.3 上传到服务器

在服务器的主目录上新建一个software目录，专门存放anaconda3：

```bash
mkdir -p ~/software/anaconda3
```

然后把 `Anaconda3-2025.06-0-Linux-x86_64.sh` 从自己的电脑上传到远程服务器的这个目录。可以用 VS Code 文件上传，也可以用 `scp` 或者 `Xftp 8` 等软件。

上传完成后，目录结构类似：

```text
~/software/anaconda3/Anaconda3-2025.06-0-Linux-x86_64.sh
```

### 2.4 解压并配置 PATH

在远程终端一行一行执行，请留意**更改下方文件名，以实际为准**：

```bash
cd ~/software/anaconda3
bash Anaconda3-2025.06-0-Linux-x86_64.sh
```
这里的：
- `cd` 进入到指定的文件夹，此时终端可以访问到这个文件夹里的文件。
- `bash` 起到执行sh脚本的作用，会询问你安装到哪个位置：
	- **强烈建议别直接回车**，如果直接按回车，会安装到个人主目录下，后续文件管理会有大麻烦。
	- **建议输入新路径后回车**，比如 `~/software/anaconda3` ，也就是将conda安装到这个sh脚本所在的同一文件夹。

安装完成后重启终端，声明**临时环境变量**以验证安装情况：

```bash
export PATH="$HOME/software/anaconda3/bin:$PATH"
```

Note：我自己的电脑上：
```bash
$HOME/software/anaconda3/bin
~/software/anaconda3/bin
/home/zjc/software/anaconda3/bin
```
这三者等价，但建议使用 `$HOME` ，会更规范

安装完成后一行行验证：

```bash
which conda
conda --version
```

正常来说会显示它的位置与版本号，如果不显示则说明声明的环境变量有误。

### 2.5 写入shell配置文件

建议上一步显示了位置和版本号后再进行这一步。

上一步的**临时环境变量**将随着当前终端关闭而消失，导致新打开的终端无法调用 `conda` 。为了让临时环境变量升级为**永久环境变量**，一行行执行：

```bash
echo 'export PATH="$HOME/software/anaconda3/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```
这里的：
- `echo` 是声明一个内容，这个内容是单引号里的`export PATH="$HOME/software/anaconda3/bin:$PATH"`，也就是环境变量
- `>>` 是在文件末尾追加新内容，追加的内容就是 `echo` 声明的内容
- `.bashrc` 是shell的配置文件，也是编辑环境变量的地方
- `source` 起到重新加载的作用

关闭终端，打开新终端验证：

```bash
which conda
conda --version
```

若有正常输出，说明安装成功。



# 在vscode里使用texlive

本文记录linux环境的 `vscode` 里 `latex` 的配置说明。


## 1. 下载texlive

[清华镜像源](https://mirrors.tuna.tsinghua.edu.cn/CTAN/systems/texlive/Images/)
选择以iso结尾的文件，比如我选择的是`texlive2026-20260301.iso`,  6.3GB大小。

## 2. 解压iso文件

比如我下载的文件放置在`/home/zjc/software/texlive/texlive2026-20260301.iso`

在vscode的终端里执行：
```bash
#先进入目标路径
cd /home/zjc/software/texlive
#创建一个文件夹以存放解压的东西
mkdir -p /home/zjc/software/texlive/texlive2026-unpacked
#用7z来解压
7z x /home/zjc/software/texlive/texlive2026-20260301.iso \  
-o/home/zjc/software/texlive/texlive2026-unpacked
#检查解压结果
cd /home/zjc/software/texlive/texlive2026-unpacked
ls
```

ls后应该能看到install-tl文件，说明对了

## 3. 安装texlive

比如我想安装到 `/home/zjc/software/texlive/2026`

在vscode终端里执行：
```bash
#先进入目标路径
cd /home/zjc/software/texlive/texlive2026-unpacked
#安装至指定路径
perl install-tl --texdir=/home/zjc/software/texlive/2026
```

在弹出来的界面里确认安装位置是TEXDIR: /home/zjc/software/texlive/2026，然后输入`I`再回车，这里需要等待安装，一般15分钟左右。

安装完成后在终端输入

```bash
find /home/zjc/software/texlive/2026/bin -name xelatex
```

大概率会得到`/home/zjc/software/texlive/2026/bin/x86_64-linux/xelatex`，说明对了

## 4. 编辑环境变量

```bash
#不用打开bashrc再添加，直接一行行输入就行
#请根据自己的情况修改你的真实路径
echo 'export PATH=/home/zjc/software/texlive/2026/bin/x86_64-linux:$PATH' >> ~/.bashrc  
echo 'export MANPATH=/home/zjc/software/texlive/2026/texmf-dist/doc/man:$MANPATH' >> ~/.bashrc  
echo 'export INFOPATH=/home/zjc/software/texlive/2026/texmf-dist/doc/info:$INFOPATH' >> ~/.bashrc  
source ~/.bashrc

#然后测试，有合法输出说明成功了
which xelatex  
which pdflatex  
which latexmk  
xelatex --version  
latexmk --version
```


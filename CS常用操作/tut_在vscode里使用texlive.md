# 在vscode里使用texlive

本文记录linux环境的 `vscode` 里 `latex` 的配置说明。


## 1. 下载texlive

[清华镜像源](https://mirrors.tuna.tsinghua.edu.cn/CTAN/systems/texlive/Images/)
选择以iso结尾的文件，比如我选择的是`texlive2026-20260301.iso`,  6.3GB大小。

## 2. 解压iso文件

比如我下载的文件放置在`/home/zjc/software/texlive/texlive2026-20260301.iso`

在vscode的终端里执行(此处默认linux系统)：
```bash
#先进入目标路径
cd /home/zjc/software/texlive
#创建一个文件夹以存放解压的东西
mkdir -p /home/zjc/software/texlive/texlive2026-unpacked
#linux用7z来解压，windows系统可以右键解压
7z x /home/zjc/software/texlive/texlive2026-20260301.iso \  
-o/home/zjc/software/texlive/texlive2026-unpacked
#检查解压结果(如果是windows系统，用dir替换ls)
cd /home/zjc/software/texlive/texlive2026-unpacked
ls
```

ls后应该能看到install-tl文件，这些是安装包的文件，说明对了

## 3. 安装texlive

比如我想安装到 `/home/zjc/software/texlive/2026`

在vscode终端里执行：
```bash
#先进入目标路径
cd /home/zjc/software/texlive/texlive2026-unpacked
#安装至指定路径
perl install-tl --texdir=/home/zjc/software/texlive/2026

#如果是windows系统，请留意根据自己的路径适配：
install-tl-windows.bat --texdir="D:\Software\Texlive\2026"
```

在弹出来的界面里确认安装位置是TEXDIR: /home/zjc/software/texlive/2026，然后输入`I`再回车，这里需要等待安装，一般15分钟左右。

安装完成后在终端输入

```bash
find /home/zjc/software/texlive/2026/bin -name xelatex
```

大概率会得到`/home/zjc/software/texlive/2026/bin/x86_64-linux/xelatex`，说明对了

## 4. 编辑环境变量(linux版本)

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

## 5. 修改Settings JSON

以远程ssh服务器为例，按`Ctrl+Shift+P`，输入并打开`Preferences: Open Remote Settings (JSON)`，在括号内新增代码：

或者在vscode的左下角，有一个齿轮。点击齿轮(或者直接 `ctrl + ,` )，点击settings，在弹出来的窗口选中**REMOTE**，点击Extensions-Latex-随便找一个Edit in settings.json，在弹出来的 settings.json里把latex-workshop的部分全部换成以下代码：

```bash
{
	#原有的内容保持不变，把latex-workshop相关行替换为
	"latex-workshop.latex.tools": [  
	{  
	"name": "latexmk-xelatex",  
	"command": "/home/zjc/software/texlive/2026/bin/x86_64-linux/latexmk",  
	"args": [  
	"-xelatex",  
	"-synctex=1",  
	"-interaction=nonstopmode",  
	"-file-line-error",  
	"%DOC%"  
	]  
	},  
	{  
	"name": "xelatex",  
	"command": "/home/zjc/software/texlive/2026/bin/x86_64-linux/xelatex",  
	"args": [  
	"-synctex=1",  
	"-interaction=nonstopmode",  
	"-file-line-error",  
	"%DOC%"  
	]  
	}  
	],  
	"latex-workshop.latex.recipes": [  
	{  
	"name": "latexmk-xelatex",  
	"tools": [  
	"latexmk-xelatex"  
	]  
	},  
	{  
	"name": "xelatex",  
	"tools": [  
	"xelatex"  
	]  
	}  
	],  
	"latex-workshop.latex.recipe.default": "latexmk-xelatex",  
	"latex-workshop.view.pdf.viewer": "tab",  
	"latex-workshop.latex.autoBuild.run": "onSave"
}
```

之后，打开tex文件，右上方有个绿色播放键，点击即可build latex project。

## 6. 记得安装latex workshop扩展
# conda的常用指令教程

本文记录 `conda` 的常用指令。

前置条件：
- `conda` 已经安装可用
在终端按行输入：

```bash
conda --version
(window请输入)where conda
(linux请输入)which conda
```

如果有类似的输出：

```text
conda 25.5.1

D:\Software\anaconda3\condabin\conda.bat
D:\Software\anaconda3\Scripts\conda.exe
```

说明conda能正常调用，否则请查看教程[tut_conda的下载安装](tut_conda的下载安装.md)

---
## conda常用指令

```bash
conda env list
conda create -n newbie python=3.9.16
conda activate newbie
conda deactivate
conda remove --name newbie --all
conda create -n newbier --clone newbie
conda list cudatoolkit
```

它们分别是：

- `conda env list`：查看所有的虚拟环境
- `conda create -n newbie python=3.9.16`：创建一个py版本为3.9.16，名字叫newbie的虚拟环境
- `conda activate newbie`：激活newbie这个虚拟环境
- `conda deactivate`：退出当前的虚拟环境
- `conda remove --name newbie --all`：删除newbie这个虚拟环境及其库
- `conda create -n newbier --clone newbie`：克隆一个新的环境newbier，复制旧环境newbie的所有库
- `conda list cudatoolkit`：(激活某个环境后)查看该conda环境的cuda版本

---
## 日常最常用

### 1.新建conda虚拟环境并通过pip下载库

```bash
conda create -n newbie python=3.9.16
conda activate newbie
pip install numpy
pip install pandas
pip install xxx
```
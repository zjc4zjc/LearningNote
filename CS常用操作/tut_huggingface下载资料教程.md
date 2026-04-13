# hf下载资料教程

本文记录 `hf` 的下载资料教程。
## 1.windows系统安装hf

一般推荐在自己的win电脑上下载了模型/数据集后复制到linux的服务器上，因为服务器下载速度一般比较慢。

推荐在虚拟环境安装hf包，之后再调用hf下载模型/数据集。终端里输入：
```bash
conda activate newbie
pip install huggingface_hub

where hf #如果这一步有路径输出，说明hf成功安装
```

## 2.设置代理

hf的访问是需要代理（梯子）的，在下载前先设置代理：

```bash
#终端cmd里请输入，这个proxy请以自己梯子的实际url为准
set http_proxy=http://127.0.0.1:7890
set https_proxy=http://127.0.0.1:7890

#如果没有梯子，可以更换镜像源进行资料下载，但一般不推荐这种方式，因为有时要你传你的token密钥
set HF_ENDPOINT=https://hf-mirror.com
set HF_TOKEN=你的密钥
```

## 3.登录

有些hf的数据集或者模型需要你登录官网，accept一个条款才有下载的资格，这种情况下需要你先去官网确认后，hf在终端里登录后才能下载资料（没提示就直接下，跳过这步）

```bash
hf auth login   #输入这一行后会让你输入你的token密钥
hf auth whoami   #登录成功后可以输入这一行确认一下
```

## 4.下载数据集/模型

格式一般是：`hf download <作者/资料名称> --repo-type <资料种类> --local-dir <路径>`，一定要指定路径，不然会默认下载到`C:\Users\<用户名>\.cache\huggingface\hub\datasets`里。

比如：

```bash
#相对路径
hf download bowen-upenn/PersonaMem-v2 --repo-type dataset --local-dir dataset/PersonaMem-v2

#绝对路径
hf download bowen-upenn/PersonaMem-v2 --repo-type dataset --local-dir "D:\Onedrive\Github\dataset\PersonaMem-v2"

#省略--repo-type的话默认指模型
#--local-dir .默认指下载到当前文件夹里
hf download zstanjj/MemSifter-4B-Thinking --local-dir .
```




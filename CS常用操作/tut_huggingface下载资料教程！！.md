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

格式一般是:
`hf download <作者/资料名称> --repo-type <资料种类> --local-dir <路径>`


```bash
hf download BAAI/bge-m3 --local-dir models/bge-m3
hf download zstanjj/MemSifter-4B-Thinking --local-dir models/zstanjj/MemSifter-4B-Thinking

#相对地址
hf download bowen-upenn/PersonaMem-v2 --repo-type dataset --local-dir dataset/PersonaMem-v2
#绝对地址
hf download bowen-upenn/PersonaMem-v2 --repo-type dataset --local-dir "D:\Onedrive\Github\UESTCRecord\Preference Arbitration\data\PersonaMem-v2"
```

| 操作         | 代码                                                                                                                                                                                                                                     | 比如                                                                                              |
| ---------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------- |
| hf镜像环境变量配置 | export HF_ENDPOINT="https://hf-mirror.com"                                                                                                                                                                                             | 使用huggingface下载模型时需要将镜像源地址加到linux的bashrc里                                                       |
| 下载hf模型     | [https://zhuanlan.zhihu.com/p/663712983](https://zhuanlan.zhihu.com/p/663712983)  <br>huggingface-cli download Qwen/Qwen3-Next-80B-A3B-Instruct-FP8 --local-dir ~/.cache/huggingface/hub/models--Qwen--Qwen3-Next-80B-A3B-Instruct-FP8 |                                                                                                 |
| 找到模型位置     | ls ~/.cache/huggingface/hub/                                                                                                                                                                                                           | models--Qwen--Qwen3-Next-80B-A3B-Instruct <br><br>models--Qwen--Qwen3-Next-80B-A3B-Instruct-FP8 |


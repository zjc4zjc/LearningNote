# huggingface下载资料教程

本文记录 `hf` 的下载资料教程。
## windows终端的常用指令教程


| 操作         | 代码                                                                                                                                                                                                                                     | 比如                                                                                              |
| ---------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------- |
| hf镜像环境变量配置 | export HF_ENDPOINT="https://hf-mirror.com"                                                                                                                                                                                             | 使用huggingface下载模型时需要将镜像源地址加到linux的bashrc里                                                       |
| 下载hf模型     | [https://zhuanlan.zhihu.com/p/663712983](https://zhuanlan.zhihu.com/p/663712983)  <br>huggingface-cli download Qwen/Qwen3-Next-80B-A3B-Instruct-FP8 --local-dir ~/.cache/huggingface/hub/models--Qwen--Qwen3-Next-80B-A3B-Instruct-FP8 |                                                                                                 |
| 找到模型位置     | ls ~/.cache/huggingface/hub/                                                                                                                                                                                                           | models--Qwen--Qwen3-Next-80B-A3B-Instruct <br><br>models--Qwen--Qwen3-Next-80B-A3B-Instruct-FP8 |


1）进入特定环境与文件夹

conda activate newbie

python -m pip install "huggingface_hub[cli]"

cd "D:\Onedrive\Github\UESTCRecord\Preference Arbitration\data"

2）设置镜像源或者系统代理

// 设置镜像源

// cmd终端里

set HF_ENDPOINT=https://hf-mirror.com

set HF_TOKEN="hf_**你的密钥**"

// powershell里

$env:HF_ENDPOINT = "[https://hf-mirror.com](https://hf-mirror.com)"

$env:HF_TOKEN="hf_**你的密钥**"

设置代理：

$env:HTTP_PROXY="http://127.0.0.1:7890"

$env:HTTPS_PROXY="http://127.0.0.1:7890"

set http_proxy=http://127.0.0.1:7890

set https_proxy=http://127.0.0.1:7890

3）登录hf

hf auth login

token：hf_**你的密钥**

hf auth whoami

4）下载

```bash

hf download BAAI/bge-m3 --local-dir models/bge-m3

hf download zstanjj/MemSifter-4B-Thinking --local-dir models/zstanjj/MemSifter-4B-Thinking

#相对地址
hf download bowen-upenn/PersonaMem-v2 --repo-type dataset --local-dir dataset/PersonaMem-v2
#绝对地址
hf download bowen-upenn/PersonaMem-v2 --repo-type dataset --local-dir "D:\Onedrive\Github\UESTCRecord\Preference Arbitration\data\PersonaMem-v2"
```

# 终端cmd的常用指令教程

本文记录 `cmd` 的常用指令。
## windows终端的常用指令教程

| 指令                                                                                                                                                                                                               | 目的                                                                                                                                           |
| ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------- |
| nvidia-smi                                                                                                                                                                                                       | 右上角查看**支持的CUDA最高版本**                                                                                                                         |
| nvidia-smi --query-gpu=name,compute_cap --format=csv,noheader                                                                                                                                                    | 查看显卡与**当前GPU架构的sm版本**                                                                                                                        |
| nvcc --version                                                                                                                                                                                                   | 查看cuda版本，如果显示'nvcc' 不是内部或外部命令，也不是可运行的程序或批处理文件，说明**还没下载CUDA toolkit**需要先去官网下载：https://developer.nvidia.com/cuda-toolkit-archive。最好下到虚拟环境里方便隔离 |
| ipconfig/all                                                                                                                                                                                                     | 查看ip                                                                                                                                         |
| python -c "import torch; print('torch:', torch.__version__); print('cuda available:', torch.cuda.is_available()); print('cuda version:', torch.version.cuda); print('device count:', torch.cuda.device_count())" | 看torch支不支持gpu                                                                                                                                |

---

## Linux终端的常用指令教程

| 指令                       | 目的                       |
| ------------------------ | ------------------------ |
| lsb_release -a           | 查看linux的操作系统             |
| lscpu                    | 查看cpu信息                  |
| nvidia-smi -L            | 看显卡型号                    |
| watch -n 1 nvidia-smi    | 动态看显卡                    |
| df -h                    | 看硬盘已挂载可用空间               |
| df -h /home/zjc          | 看自家的空间                   |
| du -sh /home/zjc         | 看自己用了多少空间                |
| cd                       | 回到home文件夹                |
| cd ..                    | 回到上一级文件夹                 |
| cd ~/software            | 进入特定路径                   |
| ls                       | 列出当前文件夹里所有文件             |
| mkdir -p ~/software/node | 在特定路径创建特定文件夹，如果已经有了则pass |
| pwd                      | 打印当前完整路径                 |
| nano ~/.bashrc           | 进入shell配置文件，配置环境变量       |
| source ~/.bashrc         | 重启shell配置文件              |

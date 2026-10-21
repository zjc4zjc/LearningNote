# vscode远程remote按键

本文记录远程remote按键

| 指令             | 目的             |
| -------------- | -------------- |
| `ctrl+shift+p` | 打开vscode的上方搜索栏 |
| `ctrl+~`       | 打开/隐藏终端        |
| `ctrl+,`       | 打开设置           |
|                |                |
|                |                |
|                |                |
|                |                |
|                |                |
|                |                |
|                |                |
|                |                |
|                |                |
|                |                |
|                |                |

查看gpu的使用者：
```
nvidia-smi --query-compute-apps=gpu_uuid,pid,used_memory --format=csv,noheader 2>/dev/null | while IFS=, read -r uuid pid mem; do user=$(ps -o user= -p $pid 2>/dev/null | head -1); cmd=$(ps -o comm= -p $pid 2>/dev/null | head -1); gpu=$(nvidia-smi --query-gpu=index,uuid --format=csv,noheader | grep "$uuid" | cut -d, -f1); echo "GPU$gpu pid=$pid user=$user cmd=$cmd mem=${mem}MB"; done
```

---


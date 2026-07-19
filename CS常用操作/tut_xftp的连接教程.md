# xftp的连接教程

本文记录 `xftp` 的连接教程，主要方便个人电脑与服务器之间的文件互传。

## 1. 软件下载

点击下载软件：[下载地址](https://cdn.netsarang.net/v8/Xftp-latest-p)

## 2. 打开软件


- 点击 `新建` 或者 `ctrl+N` ，
- 在 `名称` 处填写你的服务器的名字，随个人喜好任意都行；
- 在 `主机` 处填写服务器的ip地址，一般可以在终端输入`ip addr` 看到；
- 或者在vscode按`ctrl+shift+p`，选择 `Remote-SSH: Open SSH Configuration File...`  ，再选择 `C:\Users\<你的用户名>\.ssh\config` 查看 `HostName, Port, User`，这三者就是主机，端口，用户名。
- 补充好主机，端口，用户名，密码，`协议` 默认SFTP，方法选择`Password`，点击连接即可。



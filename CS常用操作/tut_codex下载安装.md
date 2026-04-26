# codex 的下载安装教程

本文记录 `codex` 的安装、配置和网络说明。

前置条件：

- `node` 和 `npm` 已经可用
- 终端执行 `node -v` 和 `npm -v` 有正确输出
- 你有GPT的plus会员

如果还没有安装 `node` 和 `npm`，请查看教程[tut_node_npm下载安装](tut_node_npm下载安装)
如果还没有GPT的会员，请先开通gpt plus会员

## 1. 开通gpt会员

- 需要先注册美区apple账号，账单寄送地址请选择以下免税州之一
```
阿拉斯加州（Alaska）首府朱诺（Juneau）邮编：99850;
特拉华州（Delaware）邮编：19702；
蒙大拿州（Montana）城市：Marion邮编：26586；
新罕布什尔州（New Hampshire）城市：Fremont邮编：03044；
俄勒冈州（Oregon）城市：ANTELOPE 邮编97001
```

- 支付宝定位选择旧金山，搜索惠出境，点击Pockyt Shop，充值礼品卡
- apple store里兑换礼品卡
- chatgpt的app里订阅plus会员

## 2. 安装 Codex CLI

在终端里按行执行：

```bash
npm install -g @openai/codex
codex --version
```

如果输出类似：

```text
codex-cli 0.114.0
```

说明 CLI 已安装成功。

这一步既适用于 Windows 本机，也适用于 Linux 远程服务器，只要当前终端里的 `node` 和 `npm` 可用即可。还可以通过终端直接输入 `codex` 唤起

## 3. 在有梯子的个人电脑的vscode里使用

先在Extensions里搜索codex扩展，点击安装，在右侧选择sign in。请留意梯子不能选择香港，尽量选择美国日本新加坡等地区避开open ai的区域限制。

## 4. 在无梯子的服务器的vscode里使用

配置 SSH 隧道和终端代理，让远程服务器在没有梯子的情况下能通过你本地的梯子（监听 7890 端口）访问外网，同时保持连接稳定。

### 4.1 查看当前监听端口

1. 以我的梯子端口为例: `127.0.0.1:7890`(可在梯子软件上查)
2. 请留意梯子不能选择香港，尽量选择美国日本新加坡等地区避开open ai的区域限制。
### 4.2 配置 SSH 反向隧道

1. 在vscode里按`ctrl+shift+p`，查找`Remote-SSH: Open SSH Configuration File...` ，点击打开配置文件，一般是`C:\Users\<用户名>\.ssh\config`，这个配置文件在你个人电脑上而非服务器上。
2. 在文件里写入：
```text
Host <你的服务器名称，比如我的叫5090>
  HostName <你的服务器IP或域名，比如我的是121.48.164.161>
  Port <SSH端口，比如我的是22>
  User <用户名，比如我的是zjc>

  # 让服务器通过本地代理出网，留意这里的端口来自4.1步骤
  RemoteForward 7890 127.0.0.1:7890

  # 浏览器 OAuth 回调端口，留意这里的2333是随意指定的端口
  LocalForward 2333 127.0.0.1:2333

  ServerAliveInterval 60
  ServerAliveCountMax 3
```
3. 如果不知道 `HostName` 和 `Port` 的话，在服务器终端里输入`echo $SSH_CONNECTION` ，返回的倒数第二项是 `HostName`，倒数第一项是 `Port`。
4. 记得编辑好后保存退出。

### 4.3 使用 VS Code 连接服务器

1. 安装 Remote-SSH 插件
2. `Ctrl + Shift + P`
3. 选择：

```text
Remote-SSH: Connect to Host
```

1. 选择你刚刚配置的主机，输入密码进入服务器

### 4.4 验证 SSH 隧道是否生效（服务器端）

1. 登录服务器后在终端执行：

```text
ss -tulnp | grep 7890
```
2. 如果看到：

```text
127.0.0.1:7890
```
说明反向隧道成功。

### 4.5 配置服务器代理环境

1. 编辑：

```text
nano ~/.bashrc
```

2. 在最后一行添加：

```text
vpn() {
  unset HTTP_PROXY HTTPS_PROXY http_proxy https_proxy ALL_PROXY all_proxy

  export ALL_PROXY="socks5h://127.0.0.1:7890"
  export all_proxy="socks5h://127.0.0.1:7890"
  export NO_PROXY="localhost,127.0.0.1,::1"
  export no_proxy="localhost,127.0.0.1,::1"

  echo "Proxy enabled"
}

unvpn() {
  unset HTTP_PROXY HTTPS_PROXY http_proxy https_proxy ALL_PROXY all_proxy
  echo "Proxy disabled"
}

vpn
```

之后按`ctrl+x`退出，按`Y`确认修改，再按回车退出

3. 生效：

```text
source ~/.bashrc
```

如果之后想要恢复原来的配置，直接在bashrc里删除上面添加的内容再保存即可。

### 4.6 测试代理链路

```text
curl -v https://chatgpt.com
```

如果看到：

```text
* Uses proxy env variable no_proxy == 'localhost,127.0.0.1,::1'
* Uses proxy env variable all_proxy == 'socks5h://127.0.0.1:7890'
*   Trying 127.0.0.1:7890...
* SOCKS5 connect to chatgpt.com:443 (remotely resolved)
* SOCKS5 request granted.
* Connected to 127.0.0.1 () port 7890
```

说明服务器 → 本地代理 → 外网链路正常。后续若有长串信息不用管，大概率是Cloudflare 的防机器人验证给拦截了，问题不大不影响codex使用。

### 4.7 如果服务器终端里能唤起codex正常使用，extension里不行

大概率是服务器上的 VS Code Server / extension host 没有稳定继承代理变量，导致 Codex extension 请求 chatgpt.com、ab.chatgpt.com、/wham/accounts/check 等接口失败，于是一直加载。

解决方法：把代理写进 `~/.vscode-server/server-env-setup`

1. 先在终端里创建`.vscode-server`

在文件里写入这些内容。请务必留意**一行一行**复制输入，不要整体输入也不要留空或者换行符。

```bash
#创建 VS Code Server 环境文件
mkdir -p ~/.vscode-server

#直到遇到单独一行 EOF 为止，中间所有内容都作为输入写进文件
cat > ~/.vscode-server/server-env-setup <<'EOF'
export ALL_PROXY=socks5h://127.0.0.1:7890
export all_proxy=socks5h://127.0.0.1:7890
export HTTPS_PROXY=socks5h://127.0.0.1:7890
export https_proxy=socks5h://127.0.0.1:7890
export HTTP_PROXY=socks5h://127.0.0.1:7890
export http_proxy=socks5h://127.0.0.1:7890
export NO_PROXY=localhost,127.0.0.1,::1
export no_proxy=localhost,127.0.0.1,::1
EOF

#写完后检查
cat ~/.vscode-server/server-env-setup
```
 
 里面写了三类东西：
- ALL_PROXY/all_proxy：通用代理
- HTTP_PROXY/HTTPS_PROXY 及小写版本：HTTP/HTTPS 代理
- NO_PROXY/no_proxy：本机地址不要走代理
 
 VS Code Remote-SSH 会在启动远端 server 时读取这个文件，用里面的环境变量启动远端服务。Codex extension 也会继承这些变量。

2. 修改Remote Settings JSON

按`Ctrl+Shift+P`，输入并打开`Preferences: Open Remote Settings (JSON)`，把括号内新增两行：

```bash
{
  #原有的内容保持不变，新增如下两行
  "http.proxy": "socks5://127.0.0.1:7890",
  "http.proxySupport": "override"
}
```

3. 重启 Remote-SSH 的服务器端

按`Ctrl+Shift+P`，输入并打开`Remote-SSH: Kill VS Code Server on Host`，再重新连接服务器`Remote-SSH: Connect to Host...`

基本就能解决问题。
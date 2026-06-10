# vscode里claude code扩展里使用deepseek

本文记录 `vscode` 里 `claude code` 扩展以及 `deepseek api` 的配置说明。

前置条件：

- `node` 和 `npm` 已经可用
- 终端执行 `node -v` 和 `npm -v` 有正确输出

如果还没有安装 `node` 和 `npm`，请查看教程[tut_node_npm下载安装](tut_node_npm下载安装)


## 1. 在VSCode中下载插件Claude Code for VS Code

在插件页面直接搜索Claude Code for VS Code，下载启用即可

## 2. 修改Claude Code的配置

在vscode的左下角，有一个齿轮。点击齿轮(或者直接 `ctrl + ,` )，点击settings-Extensions-Claude Code-Edit in settings.json，在弹出来的 `claudeCode.environmentVariables`里新增两行代码：
```
	"claudeCode.environmentVariables": [

    ]
```

## 3. 修改Settings JSON

如果是本机用，则修改本机的settings.json；如果是连了linux服务器，则修改服务器上的settings.json。

以远程ssh服务器为例，按`Ctrl+Shift+P`，输入并打开`Preferences: Open Remote Settings (JSON)`，把括号内新增：

```bash
{
  #原有的内容保持不变，新增如下行
	"ANTHROPIC_AUTH_TOKEN": "sk-xxxxx",
	"ANTHROPIC_BASE_URL": "https://api.deepseek.com/anthropic",
	"ANTHROPIC_MODEL": "deepseek-v4-pro[1m]",
	"ANTHROPIC_DEFAULT_OPUS_MODEL": "deepseek-v4-pro[1m]",
	"ANTHROPIC_DEFAULT_SONNET_MODEL": "deepseek-v4-pro[1m]",
	"ANTHROPIC_DEFAULT_HAIKU_MODEL": "deepseek-v4-pro[1m]",
	"CLAUDE_CODE_SUBAGENT_MODEL": "deepseek-v4-pro[1m]",
	"CLAUDE_CODE_EFFORT_LEVEL": "max",
	"CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC": "1",
}
```

3. 重启 Remote-SSH 的服务器端

按`Ctrl+Shift+P`，输入并打开`Remote-SSH: Kill VS Code Server on Host`，再重新连接服务器`Remote-SSH: Connect to Host...`

基本就能解决问题。
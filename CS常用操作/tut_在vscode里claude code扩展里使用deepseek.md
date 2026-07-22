# vscode里claude code扩展里使用deepseek

本文记录 `vscode` 里 `claude code` 扩展以及 `deepseek api` 的配置说明。

前置条件：

- `node` 和 `npm` 已经可用
- 终端执行 `node -v` 和 `npm -v` 有正确输出

如果还没有安装 `node` 和 `npm`，请查看教程[tut_node_npm下载安装](tut_node_npm下载安装)


## 1. 下载claude code

终端里执行`npm install -g @anthropic-ai/claude-code`

在VSCode中下载插件Claude Code for VS Code

在插件页面直接搜索Claude Code for VS Code，下载启用即可

## 2. 修改Claude Code的配置

假设主要是在远端服务器用，不是在本机用。

在vscode的左下角，有一个齿轮。点击齿轮(或者直接 `ctrl + ,` )，点击settings，在弹出来的窗口选中**USER**，点击Extensions-Claude Code-Edit in settings.json，在弹出来的 `claudeCode.environmentVariables`里新增两行代码：

```bash
	"claudeCode.environmentVariables": [
	{ "name": "ANTHROPIC_BASE_URL", "value": "https://xxxx" },
    { "name": "ANTHROPIC_AUTH_TOKEN", "value": "xxxx" }
    ]
```

这里的URL和TOKEN就按照上面的xxx格式就行了，不用修改，复制了直接保存并退出。

## 3. 修改Settings JSON

如果是本机用，则修改本机的settings.json；如果是连了linux服务器，则修改服务器上的settings.json。

以远程ssh服务器为例，按`Ctrl+Shift+P`，输入并打开`Preferences: Open Remote Settings (JSON)`，在括号内新增代码：

或者其实也可以用和第二步一样的方法：在vscode的左下角，有一个齿轮。点击齿轮(或者直接 `ctrl + ,` )，点击settings，在弹出来的窗口选中**REMOTE**，点击Extensions-Claude Code-Edit in settings.json，在弹出来的 `claudeCode.environmentVariables`里新增代码：

```bash
{
  #原有的内容保持不变，新增如下行
  "claudeCode.environmentVariables": [
    {"name": "ANTHROPIC_AUTH_TOKEN","value": "sk-xxx"},
    {"name": "ANTHROPIC_BASE_URL","value": "https://api.deepseek.com/anthropic" },
    {"name": "ANTHROPIC_MODEL","value": "deepseek-v4-pro[1m]"},
    {"name": "ANTHROPIC_DEFAULT_OPUS_MODEL","value": "deepseek-v4-pro[1m]"},
    {"name": "ANTHROPIC_DEFAULT_SONNET_MODEL","value": "deepseek-v4-pro[1m]"},
    {"name": "ANTHROPIC_DEFAULT_HAIKU_MODEL","value": "deepseek-v4-flash"},
    {"name": "CLAUDE_CODE_SUBAGENT_MODEL","value": "deepseek-v4-flash"},
    {"name": "CLAUDE_CODE_EFFORT_LEVEL","value": "max"},
    {"name": "CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC","value": "1"}
  ]
}
```

留意把 `ANTHROPIC_AUTH_TOKEN` 换成自己的token密钥。
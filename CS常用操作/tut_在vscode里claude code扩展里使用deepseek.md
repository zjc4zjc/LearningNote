# vscode里claude code扩展里使用deepseek

本文记录 `vscode` 里 `claude code` 扩展以及 `deepseek api` 的配置说明。

前置条件：

- `node` 和 `npm` 已经可用
- 终端执行 `node -v` 和 `npm -v` 有正确输出

如果还没有安装 `node` 和 `npm`，请查看教程[tut_node_npm下载安装](tut_node_npm下载安装.md)


## 1. 下载claude code

终端里先确保where npm有输出，再在终端里执行`npm install -g @anthropic-ai/claude-code`

在VSCode中下载插件Claude Code for VS Code

在插件页面直接搜索Claude Code for VS Code，下载启用即可

## 2. 修改Claude Code Extension的Settings JSON

在vscode的左下角，有一个齿轮。点击齿轮(或者直接 `ctrl + ,` )，点击settings，在弹出来的窗口（如果你是在本机用，则选中**USER**，如果你是在远程服务器用，则选中**REMOTE**），点击Extensions-Claude Code-Edit in settings.json，在弹出来的 `claudeCode.environmentVariables`里新增代码：

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
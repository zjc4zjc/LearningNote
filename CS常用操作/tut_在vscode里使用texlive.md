# 在vscode里使用texlive

本文记录linux环境的 `vscode` 里 `latex` 的配置说明。


## 1. 下载texlive

[清华镜像源](https://mirrors.tuna.tsinghua.edu.cn/CTAN/systems/texlive/Images/)
选择以iso结尾的文件，比如我选择的是`texlive2026-20260301.iso`,  6.3GB大小。

## 2. 解压iso文件

比如我下载的文件放置在`/home/zjc/software/texlive/texlive2026-20260301.iso`

在vscode的终端里执行：
```bash
#先进入目标路径
cd /home/zjc/software/texlive
#创建一个文件夹以存放解压的东西
mkdir -p /home/zjc/software/texlive/texlive2026-unpacked
#用7z来解压
7z x /home/zjc/software/texlive/texlive2026-20260301.iso \  
-o/home/zjc/software/texlive/texlive2026-unpacked
#检查解压结果
```

这里的URL和TOKEN就按照上面的xxx格式就行了，不用修改，复制了直接保存并退出。

## 3. 修改Settings JSON

如果是本机用，则修改本机的settings.json；如果是连了linux服务器，则修改服务器上的settings.json。

以远程ssh服务器为例，按`Ctrl+Shift+P`，输入并打开`Preferences: Open Remote Settings (JSON)`，把括号内新增：

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
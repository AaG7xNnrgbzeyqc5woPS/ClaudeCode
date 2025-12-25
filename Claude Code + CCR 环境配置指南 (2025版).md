# Claude Code + CCR 环境配置指南 (2025版)

本指南用于在网络受限环境下，通过 CCR 路由实现 Claude Code 直连，并集成 DeepSeek、Mimo 等第三方 API 供应商。

## 一、 环境准备
1. **Node.js**: 确保已安装 Node.js 环境。
2. **安装 Claude Code**:
```powershell
npm install -g @anthropic-ai/claude-code
```

3. 安装 CCR (Claude Code Router):
```PowerShell
npm install -g @musistudio/claude-code-router
```

## 二、 解决网络与协议问题 (SOCKS5 转 HTTP)
由于 Claude Code 底层引擎不支持 socks5:// 协议，若使用本地代理（如 192.168.x.x:1080），需进行桥接
1. 安装桥接工具:
```PowerShell
npm install -g http-proxy-to-socks
```
2. 启动桥接服务 (保持此窗口开启):
```PowerShell
# 将本地 SOCKS5 代理转为 8080 端口的 HTTP 代理
hpts -s 192.168.2.20:1080 -p 8080
```

## 三、 配置 CCR (config.json)
```JSON
{
  "LOG": true,
  "PORT": 3456,
  "PROXY_URL": "[http://127.0.0.1:8080](http://127.0.0.1:8080)", // 指向 hpts 桥接地址，若直连 Mimo 则设为 null
  "Providers": [
    {
      "name": "mimo",
      "api_base_url": "[https://api.xiaomimimo.com/v1/chat/completions](https://api.xiaomimimo.com/v1/chat/completions)",
      "api_key": "${CCR_MIMO_API_KEY}",
      "models": ["mimo-v2-flash"],
      "transformer": { "use": ["openai"] }
    },
    {
      "name": "deepseek",
      "api_base_url": "[https://api.deepseek.com/chat/completions](https://api.deepseek.com/chat/completions)",
      "api_key": "${CCR_DEEPSEEK_API_KEY}",
      "models": ["deepseek-chat", "deepseek-reasoner"],
      "transformer": { "use": ["deepseek"] }
    }
  ],
  "Router": {
    "default": "mimo,mimo-v2-flash",
    "think": "deepseek,deepseek-reasoner",
    "longContext": "deepseek,deepseek-chat"
  }
}
```
注释：这里没有使用 google Gemini模型。所以代理  "PROXY_URL" 这一行可以删掉。   
     同时hpts服务也可以不用，这个放到进阶教程，下一步再研究。先用好 mimo和deepseek服务吧。国产服务也很多，用不过来。

## 四、 启动服务
1. 启动 CCR 路由:
```PowerShell
ccr restart
```

2. 启动 Claude Code: 首次运行需指定本地 API Base：
```PowerShell
claude --api-base "[http://127.0.0.1:3456](http://127.0.0.1:3456)"
```

## 五、 项目级优化 (CLAUDE.md)
在 Delphi 项目根目录下创建或修改 CLAUDE.md，添加快捷指令说明，帮助 AI 识别切换意图：
```Markdown
## 开发规范
- 时区: 东八区 (UTC+8)
- 语言: 中文

## 模型快捷指令
- /mimo: 切换至 Mimo V2 Flash (快速执行/读写文件)
- /ds: 切换至 DeepSeek Reasoner (深度逻辑推理/架构分析)

```

## 六、 常用维护指令
- 查看当前路由状态: 在 Claude 窗口输入 /status。
- 强制切换模型:
    !/model mimo,mimo-v2-flash
    !/model deepseek,deepseek-reasoner
- 查看 API 消耗: ccr usage

## 七、 避坑指南
1. 400 错误 (Not supported model): 说明 Provider 不支持该字符串 ID，需通过 ccr model 或查询供应商文档确认准确 ID。
2. 指令卡死: 通常是 DNS 污染或代理失效，尝试在 hpts 启动时增加 --proxy-dns 或检查 192.168 段代理是否通畅。
3.DeepSeek Reasoner 报错: 该模型在进行“工具调用”（如读写文件）时可能因协议冲突崩溃，建议仅在纯对话逻辑推理时使用。

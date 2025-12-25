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

## 二、 设置环境变量 (API Keys)

```PowerShell
# 设置当前用户永久环境变量
[Environment]::SetEnvironmentVariable("CCR_DEEPSEEK_API_KEY", "你的DeepSeek_Key", "User")
[Environment]::SetEnvironmentVariable("CCR_MIMO_API_KEY", "你的Mimo_Key", "User")
[Environment]::SetEnvironmentVariable("CCR_GEMINI_API_KEY", "你的Gemini_Key", "User")
[Environment]::SetEnvironmentVariable("CCR_LOCAL_API_KEY", "自定义本地通信Key", "User")

# 注意：设置完成后，需要重新启动 PowerShell 窗口才能生效
```

## 三、 配置 CCR (config.json)
1. 配置文件路径
CCR 的配置文件通常位于其全局安装目录下。你可以通过以下方法找到它：
- 快速查找命令: ccr config (这通常会自动打开默认编辑器)
- 物理路径参考: C:\Users\John\.claude-code-router\config.json
- 简单办法: 在终端输入 ccr help，ccr status,ccr ui 通常会显示配置文件的具体加载路径。
2. 推荐配置内容 (直连优化版)
```JSON
{
  "LOG": true,
  "LOG_LEVEL": "info",
  "HOST": "127.0.0.1",
  "PORT": 3456,
  "APIKEY": "${CCR_LOCAL_API_KEY}",
  "API_TIMEOUT_MS": 600000,
  "PROXY_URL": null, // 已经改为直连 Mimo，无需本地代理
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
    "background": "mimo,mimo-v2-flash",
    "think": "deepseek,deepseek-reasoner",
    "longContext": "deepseek,deepseek-chat"
  }
}
```
注释：这里没有使用 google Gemini模型，代理  "PROXY_URL" 这一行可以删掉。    
     同时hpts服务也不需要，这部分内容放到进阶教程，下一步再研究。  
     先用好 mimo和deepseek服务吧，国产服务也很多，用不过来。  

## 四、 启动服务
1. 启动 CCR 路由:
```PowerShell
ccr restart
```

2. 启动 Claude Code:
```PowerShell
claude
```

3. 简洁用法，直接使用下面的指令启动ccr 和 claude code：
```PowerShell
ccr code
```

4. 快捷模型切换: 在 CLAUDE.md 中记录以下约定，方便 AI 协助切换：
-  输入 /mimo: 切换到快速响应模型 (/model mimo,mimo-v2-flash)
-  输入 /ds: 切换到深度逻辑模型 (/model deepseek,deepseek-reasoner)
   
## 六、 常用维护指令
- 查看当前路由状态: 在 Claude 窗口输入 /status。
- 强制切换模型:
```
    !/model mimo,mimo-v2-flash
    !/model deepseek,deepseek-reasoner
```
- 查看 API 消耗: ccr usage

## 七、 避坑指南
1. 400 错误 (Not supported model): 说明 Provider 不支持该字符串 ID，需通过 ccr model 或查询供应商文档确认准确 ID。  
2. 指令卡死: 通常是 DNS 污染或代理失效，尝试在 hpts 启动时增加 --proxy-dns 或检查 192.168 段代理是否通畅。  
3.DeepSeek Reasoner 报错: 该模型在进行“工具调用”（如读写文件）时可能因协议冲突崩溃，建议仅在纯对话逻辑推理时使用。  

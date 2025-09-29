# See:
- [AI编程，我找到了性价比平替，前后端真实编程场景实测](https://www.youtube.com/watch?v=HSmPZT3d1KE)
- 
# 如何配置Claude CLI，使用第三方大模型
- filename: C:\Users\username\.claude\settings.json
- 文件内容如下：
```
{
  "env": {
    "ANTHROPIC_AUTH_TOKEN": "ak_********************",
    "ANTHROPIC_BASE_URL": "https://api.longcat.chat/anthropic",  
    "ANTHROPIC_MODEL": "LongCat-Flash-Thinking",
    "ANTHROPIC_SMALL_FAST_MODEL": "LongCat-Flash-Chat",
    "ANTHROPIC_MAX_TOKENS":"8000",
    "MAX_TOKENS":"8000",
    "CLAUDE_CODE_MAX_OUTPUT_TOKENS": "8000"
  }
}
```
调试出现关键问题：
1. 需要设置"ANTHROPIC_AUTH_TOKEN"，而不是ANTHROPIC_API_KEY,如果设置错误，就会链接远程的ANTHROPIC 自己的大模型。设置正确，不需要科学上网就能使用客户端。
2. "CLAUDE_CODE_MAX_OUTPUT_TOKENS": "8000"，这个环境变量是关键。设置它之后，大模型输出才正确，否则总是错误，提示MAX_TOKENS超过服务器允许值。
3. 用这个配置文件后，使用 /init初始化代码库成功。虽然中间返回值出现请求太频繁的提示。

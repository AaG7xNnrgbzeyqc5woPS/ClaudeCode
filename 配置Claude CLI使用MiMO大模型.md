# 最近小米MiMo公测，大火
- [MiMo API key申请](https://platform.xiaomimimo.com/#/console/api-keys)
- [MiMo文档：配置 MiMo API KeyMiMo配置 MiMo API KeyMiMo配置 MiMo API KeyMiM配置 MiMo API KeyM配置 MiMo 
- [网页版对话平台](https://aistudio.xiaomimimo.com/#/)
- 2025-12-22
   
#  MiMo-V2-Flash × Claude Code 实测可用配置
|  条目	  | 官方写法	| 实测结果	| 推荐写法|
|---------|---------|---------|---------|
|接入地址	| `https://api.xiaomimimo.com/anthropic`	| 404	          | `https://api.xiaomimimo.com`|	
|鉴权头	  | `ANTHROPIC_AUTH_TOKEN`	                | 404 后继 401	| `OPENAI_API_KEY`	          |
|路径	    | `/v1/messages`	                        | 404	          | `/v1/chat/completions`      |	
|思考模式	| 默认开启	                              | 直接报错	      | 必须 Thinking off	          |
|并发	    | 无说明	                                 | 2 即 429	    | 保持 QPS≤1 最稳	            |

# ✅ 一步到位的  "~\.claude\settings.json"
```json
{
  "env": {
    "OPENAI_BASE_URL": "https://api.xiaomimimo.com",
    "OPENAI_API_KEY": "sk-xxxxxxxxxxxxxxxx",
    "OPENAI_MODEL": "mimo-v2-flash"
  },
  "model": "mimo-v2-flash",
  "alwaysThinkingEnabled": false
}

```
保存后  claude  重启， /init  返回 200 即成功。

# 🔍 快速验证脚本（bash / PowerShell 通用）
```bash
curl -s -o /dev/null -w "%{http_code}" \
  -H "Authorization: Bearer ${OPENAI_API_KEY}" \
  -H "Content-Type: application/json" \
  -d '{"model":"mimo-v2-flash","messages":[{"role":"user","content":"hi"}]}' \
  https://api.xiaomimimo.com/v1/chat/completions
```
输出  200  说明 key + 网络 + 路径全部 OK







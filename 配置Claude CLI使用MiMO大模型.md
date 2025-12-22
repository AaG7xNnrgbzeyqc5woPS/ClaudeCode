# 最近小米MiMo公测，大火
   -2025-12-22
#  MiMo-V2-Flash × Claude Code 实测可用配置
|  条目	  | 官方写法	| 实测结果	| 推荐写法|
|---------|---------|---------|---------|
|接入地址	| `https://api.xiaomimimo.com/anthropic`	| 404	          | `https://api.xiaomimimo.com`|	
|鉴权头	  | `ANTHROPIC_AUTH_TOKEN`	                | 404 后继 401	| `OPENAI_API_KEY`	          |
|路径	    | `/v1/messages`	                        | 404	          | `/v1/chat/completions`      |	
|思考模式	| 默认开启	                                | 直接报错	      | 必须 Thinking off	          |
|并发	    | 无说明	                                  | 2 即 429	    | 保持 QPS≤1 最稳	            |



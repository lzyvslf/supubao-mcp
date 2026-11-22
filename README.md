# Supubao Tool — a remote MCP server with 36 no-install tools

A remote MCP server that is already running in production: 36 backend utilities (hashing, encoding,
time, regex, cron, JWT, CIDR, images, text and data-format conversion). **No stdio process, no local
install** — paste one URL into any Streamable HTTP capable client.

一个已经跑在公网上的远程 MCP 服务器：36 个免登录的后端小工具，覆盖哈希、编码、时间、正则、
Cron、JWT、CIDR、图片、文本与数据格式转换。不需要本地安装，客户端填一个 URL 就能连。

| | |
| --- | --- |
| Streamable HTTP | `https://mcp.supubao.com/mcp` |
| SSE | `https://mcp.supubao.com/mcp/sse` |
| Machine-readable tool list | `https://api.supubao.com/api/v1/tools` |
| Human docs | `https://tool.supubao.com/mcp` |
| Auth | anonymous by default; optional `X-API-Key` header raises quota |
| Server stack | Python / FastAPI, hosted (client is platform-agnostic) |

> This repository only holds the integration notes and the tool list, so directories and clients can
> reference it. The server implementation is not in this repo.

Directory blurb (≤100 chars, used in `server.json` for the official MCP Registry):

> 36 no-install tools: weather, IP lookup, live trending, hash, JWT, cron, CIDR. Remote MCP, no key.

## What is actually different

Most entries in an MCP registry are "wrapper around an API" or "stdio binary you must install".
This one is a hosted endpoint, and more importantly a handful of the tools make a specific judgement
that the obvious alternative gets wrong. Those claims are verifiable by calling the endpoint; the
left column is what goes wrong elsewhere, the right column is what this server does instead.

| Where the obvious alternative breaks | What this server does | Tool |
| --- | --- | --- |
| Character-by-character Chinese conversion turns 头发 into 頭髮 and 后台 into 後台, wrecking a whole document | Word-level conversion via OpenCC configs (`s2twp` for Simplified→Traditional, `s2hk` for Hong Kong, `t2s` for the reverse) | `zhconvert` |
| Test fixtures built from real production rows (a compliance problem), or from `13800000000`-style dummies that never surface dirty-data bugs | IDs are random but the check digit is computed with the standard algorithm; masking widths are fixed in code and auditable (phone keeps first 3 + last 4, ID keeps first 4 + last 4) | `testdata` |
| Formatters that also rewrite string literals and comments, so the query you paste into a ticket is not the query that ran | Strings and comments are lifted out behind placeholders and restored verbatim; only the code section is formatted. Longest keyword wins, so `GROUP BY` is not split into `GROUP` + `BY` | `sqlfmt` |
| Pinyin tools guess polyphones per character: 重庆 becomes zhong qing, 银行 becomes yin xing | Phrase-aware polyphone resolution by default (`重庆 → chóngqìng`), with a per-character mode available | `pinyin` |
| Word counters that disagree with Word and with the CMS backend, because nobody wrote down the rule | Rule is fixed and stated: characters counted as CJK chars + English words, spaces and punctuation excluded | `textcount` |
| Generated passwords show a pattern — symbols always trailing, letters split into blocks | At least one char per selected class, then the whole string is shuffled; ambiguous chars (`0/O`, `1/l`) can be excluded | `password` |
| CIDR tools handle IPv4 only, accept malformed masks, cannot convert `/26` ↔ dotted mask | IPv4 and IPv6; accepts `/26`, dotted-quad mask, or explicit `op=mask`, and validates that the mask is a legal contiguous run of ones | `cidr` |
| JSON→CSV converters only eat a bare array, and choke on the very common `{"data":[...]}` wrapper | The wrapped array is located automatically; numeric type inference is optional | `json2csv` |
| Timezone conversion by hand-computed offsets, wrong during DST weeks | Conversions follow IANA rules, DST handled by the database, not by arithmetic | `tzconvert` |
| Image compressors say "compressed" without numbers, and silently make a PNG lossy when capping width | Returns origin bytes, output bytes and the saved ratio; format and max width are separate parameters | `imgcompress` |
| Markdown tools call a remote rendering service, sending your document to a third party | Rendered locally, no network call | `markdown` |
| "JWT decode" pages imply they verify signatures, or quietly call a backend to do it | Decodes header and payload only, explicitly no signature check — use `jwtsign` when verification is needed | `jwt` |

Not every tool here is special, and the README will not pretend otherwise: `base64`, `uuid`,
`timestamp`, `json`, `qrcode`, `unit` are infrastructure, kept because a client should not have to
chain three servers to do four small things.

## 接入 / Connect

```json
{
  "mcpServers": {
    "supubao-tool": {
      "url": "https://mcp.supubao.com/mcp",
      "transport": "streamable-http"
    }
  }
}
```

Claude Desktop, Cursor, VS Code, Cline, Windsurf and Cherry Studio support remote MCP; field names
differ per client, see https://tool.supubao.com/mcp for the per-client snippets.

## 工具清单（36 个）

| 工具 | 说明 |
| --- | --- |
| `hash` | MD5/SHA1/SHA256 哈希（哈希不是加密），结果可按标准逐字节核对 |
| `base64` | 文本 Base64 编码 / 解码 |
| `timestamp` | 时间戳与本地时间互转 |
| `json` | JSON 格式化 / 压缩 / 校验 |
| `weather` | 城市实时天气 + 未来几天预报（Open-Meteo，免密钥） |
| `ip` | IP 归属地：国内源→境外源→兜底三级串联，国内 IP 命中第一条源 |
| `hot` | 微博/百度实时热榜，按榜单类型路由到对应源，避免两榜格式互相串 |
| `uuid` | 在线生成 UUID v4，支持批量、大写、去横线 |
| `urlencode` | URLEncode / URLDecode，支持整段 URL 与参数值两种模式 |
| `regex` | 正则匹配、查找全部、替换，返回分组结果 |
| `cron` | 解析 Cron 表达式，输出字段含义与接下来 5 次执行时间 |
| `jwt` | 解码 JWT 的 Header/Payload，只解码不验签（要验签请用 jwtsign） |
| `password` | 生成高强度随机密码：每类至少一个后整体洗牌，可排除易混字符 |
| `textcount` | 字数统计：口径固定为中文字符+英文单词、不含空格与标点 |
| `qrcode` | 生成二维码 PNG，支持容错等级、尺寸与白边调节 |
| `yaml` | YAML⇄JSON，支持多文档、缩进调节与语法校验 |
| `unicode` | 中文与 \uXXXX 互转，也支持 HTML 实体（&#NNNN;） |
| `textdiff` | 按行去重并统计重复次数，或对比两段文本找新增/删除/修改 |
| `tsbatch` | 整列时间戳与日期批量互转，自动识别秒/毫秒，支持自定义时区偏移 |
| `cidr` | CIDR 网段计算：IPv4/IPv6，接受 /26、点分掩码、op=mask 三种写法并校验合法性 |
| `imgbase64` | 解析图片 Base64 / data URI 的格式与尺寸，或转成可下载链接 |
| `jwtsign` | 签发 HS256/384/512 的 JWT 并验签：校验签名、exp 与 nbf |
| `sqlfmt` | SQL 格式化：字符串与注释整段保护、只对代码段动刀，字面量原样保留 |
| `markdown` | Markdown⇄HTML，本地渲染、不联网 |
| `imgcompress` | 压缩并转 JPG/PNG/WebP，可限最大宽度；返回压缩前后字节与比例 |
| `testdata` | 生成测试数据：身份证校验位按国标算法、号码本体虚构；脱敏格式保留位数写死可核对 |
| `utm` | 给链接批量加 utm_* 参数，或拆解已有链接里的 UTM 参数 |
| `json2csv` | JSON⇄CSV：自动找到「data 包一层」的数组结构，可控制是否推断数字类型 |
| `cronbuild` | 按频率生成 Cron，或把已有 Cron 翻成中文并算出接下来几次执行时间 |
| `color` | HEX/RGB/HSL/HSV 互转，输出亮度与黑白对比度，可生成同色系配色 |
| `tzconvert` | 各时区当前时间，或把某时区日期时间换算到另一时区，按 IANA 规则处理夏令时 |
| `httpstatus` | 查状态码含义、常见成因与排查方向，支持一次查多个 |
| `pinyin` | 汉字转拼音：默认按词组识别多音字，可切逐字模式；支持声调/首声/首字母 |
| `zhconvert` | 简繁互转：OpenCC 词级转换（s2twp / s2hk / t2s），按词语而非单字 |
| `unit` | 长度、重量、面积、体积、存储、速度、时间与温度的换算，同时给出同类全部单位结果 |
| `datediff` | 日期差：天/周/工作日三种口径，或按出生日期算周岁并给下次生日倒计时 |

## 接口约定

- 统一返回 `{"code":0,"message":"ok","data":{...},"request_id":"..."}`，`code=0` 表示成功
- REST 形态：`GET https://tool.supubao.com/api/v1/tool/<slug>?param=value`，例如
  `https://tool.supubao.com/api/v1/tool/ip?q=8.8.8.8`
- 目录：`https://api.supubao.com/api/v1/tools`
- `POST /mcp` 是唯一合法的 JSON-RPC 入口；`GET /mcp` 返回 `405` 并带 `Allow: POST, OPTIONS`
  （浏览器带 `Accept: text/html` 时给落地页，那是给人看的，不是给客户端握手的）

## 免费与额度

匿名按 IP 每天 10 次、每分钟 10 次；带 `X-API-Key` 注册用户每天 50 次；更高等级按用量开通。
`tools/list` 不计额度，只有真正调用工具才扣。这些数字来自服务端的 `limit_rules` 表，
当前值与 `https://tool.supubao.com/mcp` 页面上写的完全一致；调整时会同步改页面，两处不一致就是 bug。

## 不做的事

- 不代理第三方付费接口，不把别人的数据包装成自己的
- 不记录调用内容，`request_id` 只用于排查
- 没有的字段返回空，不猜、不补

## 这个仓库是什么、不是什么

是什么：一个远程 MCP 服务器的接入说明与工具清单，方便目录收录和客户端引用。

不是什么：不是服务器源码。这里没有 `src/`、没有 `Dockerfile`、没有 `smithery.yaml`，
因为服务已经跑在公网上，不需要任何人构建或部署它。想用就填 URL。
如果你在找可以本地跑的 stdio 版本，这个仓库不提供。

## 自测记录（2026-11-22，全部现场执行）

```bash
curl -s -X POST https://mcp.supubao.com/mcp \
  -H 'Content-Type: application/json' \
  -d '{"jsonrpc":"2.0","id":1,"method":"tools/list"}'
# → 36 tools，description 已按上面「What is actually different」重写

curl -s -o /dev/null -w '%{http_code}\n' https://mcp.supubao.com/mcp
# → 405（GET 不是合法握手方式）

curl -s 'https://api.supubao.com/api/v1/tool/hash?text=hello&algo=sha256'
# → 2cf24dba5fb0a30e26e83b2ac5b9e29e1b161e5c1fa7425e73043362938b9824（与标准值逐字节一致）

curl -s -X POST https://api.supubao.com/api/v1/tool/zhconvert \
  -H 'Content-Type: application/json' \
  -d '{"text":"头发很长","direction":"s2t"}'
# → {"result":"頭髮很長","mode":"词语级","changed_chars":3,...}（mode 字段说明走的是词表而不是逐字映射）
```

这条记录的意义是：README 里写下来的每一条，都是在写下来的时候现场跑通的，不是从规划文档抄的。
如果哪天不通了，请开 issue，别先怀疑自己的配置。

## 许可

仓库里的文档与配置示例是 MIT（见 LICENSE）。这把许可证**不覆盖**服务器实现，
服务本身按免费使用提供，有自己的使用条款。第三方客户端各自的许可自行确认。

## 延伸阅读与支持

- 这些差异判断中有一条被写成了完整的技术稿：《为什么「脱敏测试数据」必须可复现：从一张规则表说起》— https://juejin.cn/post/7710543515243806771
- 工具始终免费，付费的只有需要写清规则、可复现的按单交付：https://tool.supubao.com/docs/desensitize
- 如果这套东西对你有用，可以在爱发电小额赞助服务器与更新成本，不解锁任何功能：https://ifdian.net/a/yongge-ai

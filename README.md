# 速宝工具 MCP Server（supubao）

免密钥、国内直连的 MCP 服务端，把 **36 个**在线工具与实时数据封装成标准 MCP 工具。
不需要申请 API Key，不需要自己部署，复制一行地址即可在 Claude Desktop / Cursor / Cline /
Cherry Studio / Trae / Windsurf 中调用。

- Streamable HTTP：`https://mcp.supubao.com/mcp`
- SSE（老客户端）：`https://mcp.supubao.com/mcp/sse`
- 网页版对照：https://tool.supubao.com
- 接口说明：https://tool.supubao.com/docs/data · https://tool.supubao.com/docs/api

## 为什么选这个 server

1. **三个模型编不出来的实时数据**：天气、IP 归属地、微博/百度实时热搜。模型的知识有截止日期，
   这三件事必须靠接口。
2. **免密钥**：多数 MCP 服务要 API Key、要装 Node 包、要配环境变量；这里只需要一个 URL。
3. **国内节点直连**：服务器在国内，延迟通常在 10–50 ms，不需要代理。
4. **带缓存与降级**：数据类接口缓存落盘（天气/热搜 15 分钟、IP 6 小时），上游全部失败时用
   最近一份过期数据兜底并在响应中标记 `_cache.stale`，不会静默返回旧数据。

## 接入

Claude Desktop（较新版本）/ Cursor：

```json
{
  "mcpServers": {
    "supubao": {
      "type": "http",
      "url": "https://mcp.supubao.com/mcp"
    }
  }
}
```

只支持 SSE 的客户端（Cherry Studio 等）：

```json
{
  "mcpServers": {
    "supubao": {
      "type": "sse",
      "url": "https://mcp.supubao.com/mcp/sse"
    }
  }
}
```

命令行验证：

```bash
curl -s https://mcp.supubao.com/mcp \
  -H 'Content-Type: application/json' \
  -d '{"jsonrpc":"2.0","id":1,"method":"tools/list"}'
```

## 工具清单（36 个）

| 工具名 (name) | 中文名 | 作用 |
|---|---|---|
| `weather` | 天气查询 | 城市实时天气 + 未来几天预报（Open-Meteo，免密钥） |
| `ip` | IP 归属地 | 查询 IP 的国家/省市/区县/运营商 |
| `hot` | 全网热搜 | 微博 / 百度 实时热榜（免密钥） |
| `httpstatus` | HTTP 状态码速查 | 输入 200/301/403/502 等状态码，查含义、常见成因与排查方向，支持一次查多个 |
| `json` | JSON 格式化 | JSON 格式化 / 压缩 / 校验 |
| `json2csv` | JSON⇄CSV 转换 | 把 JSON 数组转成 CSV（Excel 可直接打开），或把 CSV 转回 JSON 数组 |
| `yaml` | YAML/JSON 互转 | YAML 与 JSON 双向转换，支持多文档、缩进调节与语法校验 |
| `sqlfmt` | SQL 格式化 | 在线格式化 SQL：关键字大写、按子句换行缩进，字符串与注释内容不受影响 |
| `jwt` | JWT 解码 | 在线解析 JWT 的 Header 与 Payload（仅解码，不做签名校验） |
| `jwtsign` | JWT 生成与验签 | 在线签发 HS256/HS384/HS512 的 JWT，并校验签名、exp 与 nbf |
| `regex` | 正则表达式测试 | 在线正则测试：匹配、查找全部、替换，返回分组结果 |
| `cron` | Cron 表达式解析 | 在线解析 Cron 表达式，输出字段含义与接下来 5 次执行时间 |
| `cronbuild` | Cron 生成器与中文解读 | 按执行频率生成 Cron 表达式，或把已有 Cron 翻译成中文并算出接下来几次执行时间 |
| `utm` | UTM 链接生成与拆解 | 给链接批量加上 utm_source/medium/campaign 等投放参数，或拆解已有链接里的 UTM 参数 |
| `color` | 颜色格式转换 | HEX / RGB / HSL / HSV 互相转换，输出亮度与黑白对比度，并可生成同色系配色 |
| `tzconvert` | 时区转换与世界时钟 | 查看各时区当前时间，或把某个时区的日期时间换算到另一时区，自动处理夏令时 |
| `markdown` | Markdown⇄HTML | 在线把 Markdown 转成 HTML，或把 HTML 转回 Markdown；本地渲染、不联网 |
| `testdata` | 测试数据生成 | 批量生成手机号、身份证、银行卡、姓名、邮箱、地址等测试数据，支持脱敏格式 |
| `pinyin` | 汉字转拼音 | 把中文转成拼音：带声调、不带声调或只取首字母，支持多音字按词组识别 |
| `zhconvert` | 简体繁体转换 | 简体中文与繁体中文互转，按词语而不是单字转换，避免出现「头发/頭髮」这类错转 |
| `datediff` | 日期差与年龄计算 | 算两个日期相差多少天/周/工作日，或按出生日期算出精确周岁与下次生日倒计时 |
| `unit` | 单位换算 | 长度、重量、面积、体积、存储、速度、时间与温度的换算，同时给出同类全部单位的结果 |
| `base64` | Base64 编解码 | 文本 Base64 编码 / 解码 |
| `imgbase64` | 图片 Base64 互转 | 解析图片 Base64 / data URI 的格式与尺寸，或把 data URI 转成可下载的图片链接 |
| `hash` | 哈希计算 | MD5 / SHA1 / SHA256 哈希计算 |
| `urlencode` | URL 编解码 | URLEncode / URLDecode 在线转换，支持整段 URL 与参数值两种模式 |
| `unicode` | Unicode 转码 | 中文与 \uXXXX 互转，也支持 HTML 实体批量转义与还原 |
| `uuid` | UUID 生成 | 在线生成 UUID v4，支持批量、大写、去横线 |
| `timestamp` | 时间戳转换 | 时间戳与本地时间互转 |
| `tsbatch` | 时间戳批量转换 | 整列时间戳与日期批量互转，自动识别秒/毫秒，支持自定义时区偏移 |
| `qrcode` | 二维码生成 | 在线生成二维码 PNG，支持容错等级、尺寸与白边调节，可直接下载或用于 API |
| `password` | 随机密码生成 | 在线生成高强度随机密码，可选长度、大小写、数字、符号与易混字符排除 |
| `textcount` | 字数统计 | 在线统计中文字数、字符数、词数、行数与去重行数，支持 20 万字符 |
| `textdiff` | 文本去重与对比 | 按行去重并统计重复次数，或对比两段文本找出新增/删除/修改的行 |
| `cidr` | IP 段计算 | CIDR 网段解析：网络地址、掩码、广播地址、可用地址数，支持 IPv4 与 IPv6 |
| `imgcompress` | 图片压缩与格式转换 | 在线压缩图片、转 JPG/PNG/WebP，可限制最大宽度，返回压缩前后体积对比 |


## 额度

- 匿名（按 IP）：**10 次/天**基础额度
- 注册账号并用 `X-API-Key` 请求：**50 次/天**
- 签到（+5，连签第 7 天 +20）与一级邀请（邀请人 +50 / 被邀请人 +20）可换额外额度，3 积分抵 1 次
- 超限返回带数值的 `isError` 文本，不中断对话

## 数据来源与使用边界

| 数据 | 来源 | 边界 |
|---|---|---|
| 天气 | Open-Meteo 免费版 | 条款限制非商业用途 |
| IP 归属地 | whois.pconline.com.cn（国内 IP 主）/ ipinfo.io（境外 IP 主）/ api.ip.sb（备用） | 免费版限制非商业用途 |
| 热搜 | 微博、百度公开榜单页 | 非官方授权接口 |

个人开发、学习与内部演示可直接使用；商用需替换为对应服务的付费授权或国内合规数据源。
说明见 https://tool.supubao.com/docs/data

## 许可

服务端代码为自建项目；工具输出数据版权归各数据源。请勿高频轮询（有分钟级限流）。

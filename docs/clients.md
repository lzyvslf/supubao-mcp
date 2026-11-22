# 各客户端怎么连

通用事实：服务是**远程**的，走 Streamable HTTP，地址 `https://mcp.supubao.com/mcp`。
没有 stdio 进程，不需要装 Node 或 Python，也不需要 fork 任何东西。

下面每家给了大致填法。**字段名以各家当日文档为准**——这些客户端改配置格式比改版本号还勤，
我不想在这里写一个过期了就骗人的东西。文档给了入口链接，照着点进去核对一次再填。

---

## Claude Desktop

Claude Desktop 的远程服务走「Connectors」，不是编辑 JSON 文件：

1. Settings → Connectors → Add custom connector
2. URL 填 `https://mcp.supubao.com/mcp`
3. 保存后在对话里让它调用工具

较老版本只能读 `claude_desktop_config.json` 里的本地 stdio 服务，那类版本连不上远程服务。
先升级再试，别去改那个 JSON 里的 `mcpServers` 硬塞 URL。

## Cursor

`~/.cursor/mcp.json`（或项目的 `.cursor/mcp.json`），形如：

```json
{
  "mcpServers": {
    "supubao-tool": {
      "url": "https://mcp.supubao.com/mcp"
    }
  }
}
```

Cursor 认 `url` 字段，不需要 `command`。

## VS Code（内置 MCP 支持）

`settings.json` 或工作区的 `.vscode/mcp.json`，形如：

```json
{
  "servers": {
    "supubao-tool": {
      "type": "http",
      "url": "https://mcp.supubao.com/mcp"
    }
  }
}
```

注意根键是 `servers` 不是 `mcpServers`，`type` 写 `http`。

## Cline / Roo Code（VS Code 插件）

设置面板里的 MCP Servers 可以直接选 Remote / Streamable HTTP 类型，URL 同上。
配置文件里是 `mcpServers` 根键，字段各家插件略有差别。

## Windsurf

`~/.codeium/windsurf/mcp_config.json`，`mcpServers` 根键，字段是 `serverUrl` 而不是 `url`。
这一项最容易抄错，看到 404 或「无法连接」先检查这里。

## 自写客户端

任何实现了 Streamable HTTP 的 SDK 都能连。最小握手：

```
POST /mcp
Content-Type: application/json
Accept: application/json, text/event-stream

{"jsonrpc":"2.0","id":1,"method":"initialize",
 "params":{"protocolVersion":"2025-06-18","capabilities":{},
           "clientInfo":{"name":"my-client","version":"0.1"}}}
```

然后 `tools/list` 拿 36 个工具，`tools/call` 调用。也可以用 SSE 端点
`https://mcp.supubao.com/mcp/sse`。

---

## 连不上时按这个顺序查

1. `curl -s -X POST https://mcp.supubao.com/mcp -H 'Content-Type: application/json' -H 'Accept: application/json, text/event-stream' -d '{"jsonrpc":"2.0","id":1,"method":"initialize","params":{"protocolVersion":"2025-06-18","capabilities":{},"clientInfo":{"name":"probe","version":"0.1"}}}'`
   —— 返回 `serverInfo` 说明服务端没问题，问题在客户端配置
2. 检查根键名（`mcpServers` / `servers`）和 URL 字段名（`url` / `serverUrl`）
3. 检查 `Accept` 头是否带上了 `text/event-stream`，缺这个会被服务端拒
4. 如果被限流（短时间内大量调用），等一分钟

# Engram for Bolt.diy

Give Bolt.diy persistent, queryable memory across builds via the Engram MCP server.

Bolt.diy ships native MCP support (Settings → MCP tab) using the Vercel AI SDK's
experimental MCP client. This recipe shows the JSON snippet to drop in.

> **What is this?** A configuration recipe — no install required on Bolt.diy's side.
> The recipe targets `mcp.lumetra.io` directly. No bridge process, no extra deps.

---

## Prerequisites

- Bolt.diy running locally or self-hosted (see [stackblitz-labs/bolt.diy](https://github.com/stackblitz-labs/bolt.diy))
- An Engram API key from <https://lumetra.io> (format: `eng_live_...`)
- A BYOK provider key configured at [lumetra.io/models](https://lumetra.io/models). Without one, `store_memory` and `query_memory` return HTTP 412 BYOK_REQUIRED. Engram runs inference against your own LLM provider key, never a vendor-owned key.

## Setup

1. Open Bolt.diy.
2. Click the avatar → **Settings** → **MCP** tab.
3. Paste the JSON below into the configuration editor (merge with any existing `mcpServers`).
4. Replace `YOUR_ENGRAM_API_KEY` with your key.
5. Click **Save**, then **Check availability** — you should see `engram` flip to **available**
   with tool count > 0.

### Config

```json
{
  "mcpServers": {
    "engram": {
      "type": "sse",
      "url": "https://mcp.lumetra.io/mcp/sse",
      "headers": {
        "Authorization": "Bearer YOUR_ENGRAM_API_KEY"
      }
    }
  }
}
```

That's it. Bolt's agent now has access to Engram tools (`store_memory`, `query_memory`,
`list_buckets`, etc.) and can call them mid-build.

---

## Verifying the endpoint by hand

If "Check availability" reports the server as unavailable, confirm the endpoint
is reachable from your shell before debugging Bolt:

```bash
curl -sS https://mcp.lumetra.io/mcp/sse \
  -H "Authorization: Bearer YOUR_ENGRAM_API_KEY" \
  -H "Accept: text/event-stream" \
  --max-time 5
```

You should see an `event: endpoint` line within ~1 second. A 401 means your
key is wrong; a connection error means a network/proxy issue between your
machine and Lumetra.

---

## Usage patterns

Once configured, prompt Bolt to use memory the way you would any other tool:

- "Before scaffolding, query Engram for any notes on this project's auth setup."
- "Remember that we standardized on Tailwind v4 in the `client-frontend` bucket."
- "Pull the last three decisions from the `infra` bucket and follow them."

Bolt.diy's `maxLLMSteps` (also in the MCP tab) controls how many tool calls the
agent can chain per turn — bump it to 5+ if you want the agent to query *and*
store in the same response.

---

## Transport notes

Bolt.diy's MCP service supports three transports (`stdio`, `sse`, `streamable-http`).
Engram exposes both SSE and Streamable HTTP. SSE is the default recommended
transport and is the one shown above. If you want Streamable HTTP instead:

```json
{
  "mcpServers": {
    "engram": {
      "type": "streamable-http",
      "url": "https://mcp.lumetra.io/mcp",
      "headers": {
        "Authorization": "Bearer YOUR_ENGRAM_API_KEY"
      }
    }
  }
}
```

Both work; SSE has marginally better tool-call streaming today.

---

## Compatibility

Tested against Bolt.diy `main` as of 2026-05-20. The MCP tab is GA in Bolt.diy —
see [`app/lib/services/mcpService.ts`](https://github.com/stackblitz-labs/bolt.diy/blob/main/app/lib/services/mcpService.ts)
and [`app/components/@settings/tabs/mcp/`](https://github.com/stackblitz-labs/bolt.diy/tree/main/app/components/@settings/tabs/mcp).

## Links

- Engram: <https://lumetra.io>
- Engram MCP docs: <https://docs.lumetra.io>
- Bolt.diy: <https://github.com/stackblitz-labs/bolt.diy>
- MCP spec: <https://modelcontextprotocol.io>

## License

MIT — see [LICENSE](./LICENSE).

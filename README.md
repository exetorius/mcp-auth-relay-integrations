# mcp-auth-relay-integrations

Integration packs for [mcp-auth-relay](https://github.com/exetorius/mcp-auth-relay).

Each pack customises the relay for a specific upstream MCP server — adding tool hints, synthetic tools, and agent instructions without touching the relay itself.

## Using a Pack

The easiest way is the built-in installer. While the relay is running, type:
```
/packs
```
Pick a pack from the list. It downloads into your relay's `integrations/` folder and activates immediately.

To manually install, copy a pack folder into `integrations/` next to your relay:
```
mcp-auth-relay/
  integrations/
    vibeue/          ← pack folder
      hints.json
      ...
```
Then set `"integration": "vibeue"` in `config.json` and type `/reload`.

## Available Packs

| Pack | Upstream Server | Description |
|------|----------------|-------------|
| `vibeue` | [VibeUE](https://github.com/kevinpbuckley/VibeUE) | Unreal Engine MCP plugin — tool hints, skill index, agent instructions |

## Pack Format

A pack is a folder containing any combination of these files:

### `config.example.json`
Pre-filled relay config for this integration. Copy to `config.json` as a starting point — fill in `bearer_token` and adjust ports if needed.

### `hints.json`
Tool description appends — injected into `tools/list` at the proxy layer. Format:
```json
{
  "tool_name": "\n\nExtra guidance appended to the tool description."
}
```
Hints are paid as tokens on every MCP session. Keep them focused.

### `synthetic_tools.json`
Extra tools served by the relay itself, not forwarded to the upstream server. Format:
```json
[
  {
    "name": "my_tool",
    "description": "What this tool does.",
    "inputSchema": {
      "type": "object",
      "properties": {},
      "required": []
    }
  }
]
```

### `instructions.md`
Agent instructions injected into the MCP `initialize` response (`serverInfo.instructions`). Read once per session by the MCP client. Use for routing rules, API patterns, and anything the agent needs before it starts working.

### `README.md`
Human-facing setup guide for the pack.

## Contributing a Pack

1. Fork this repo
2. Create a folder named after your integration (e.g. `my-server/`)
3. Add whichever files apply — all are optional
4. Submit a PR to the `contrib` branch — not `main`

Pack folder names should be lowercase, hyphenated, and match the upstream server's common name.

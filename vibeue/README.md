# VibeUE Integration Pack

Connects mcp-auth-relay to the VibeUE Unreal Engine plugin's MCP server.

## Architecture

```
MCP client (Claude Code) → mcp-auth-relay (port 8089) → VibeUE MCP server (port 8088, inside UE)
```

The relay injects the MCP bearer token on every request so the client never handles it directly. Tools are served from the cached manifest even when Unreal Engine is not running.

---

## Setup

### 1. Install the VibeUE plugin

Install VibeUE into your Unreal Engine project. The plugin is available from the Unreal Marketplace or vibeue.com.

> **Note:** VibeUE has its own API key for its web services (terrain data, etc.). That key lives in UE Project Settings → Plugins → VibeUE → API Key and is separate from the MCP bearer token below. The relay does not touch it.

### 2. Enable the MCP server in UE

Open **Project Settings → Plugins → VibeUE**:
- Set an **MCP Bearer Token** (any strong random string you generate yourself)
- Enable **MCP Server Enabled**
- Confirm **MCP Server Port** is `8088`

### 3. Configure the relay

Copy `config.example.json` from this pack to `config.json` next to `proxy.py` (or your C++ binary) and fill in your values:

```json
{
  "bearer_token":  "the token you set in UE Project Settings",
  "proxy_port":    8089,
  "upstream_host": "127.0.0.1",
  "upstream_port": 8088,
  "manifest_path": "%APPDATA%/VibeUE/tools-manifest.json",
  "integration":   "vibeue",
  "server_name":   "VibeUE",
  "instructions":  ""
}
```

`%APPDATA%` is expanded automatically — no need to hardcode your username.

### 4. Start the relay

```bash
cd python
python proxy.py
```

### 5. Configure your MCP client

Add to `.mcp.json` in your project root:

```json
{
  "mcpServers": {
    "VibeUE": {
      "type": "http",
      "url": "http://127.0.0.1:8089/mcp"
    }
  }
}
```

### 6. Open your Unreal Engine project

Launch UE with your project. The VibeUE plugin starts the MCP server automatically on editor load. You will see "MCP Server started at http://127.0.0.1:8088" in the UE Output Log when it is ready.

The relay writes a tool manifest to `%APPDATA%/VibeUE/tools-manifest.json` on first successful connection. After that, tools are available even if UE is not running (though calls will fail gracefully until UE is back up).

---

## Verification

```bash
curl http://127.0.0.1:8089/mcp
# → "mcp-auth-relay running"
```

Then ask Claude Code to call `vibeue_status` to check the full setup.

---

## Tools

| Tool | Purpose |
|---|---|
| `execute_python_code` | Run Python in the UE editor context |
| `discover_python_module` | List classes/functions in a Python module |
| `discover_python_class` | Get methods of a Python class |
| `discover_python_function` | Get signature and docs for a function |
| `list_python_subsystems` | List all UE editor subsystems |
| `manage_asset` | Search, open, save, move, duplicate, delete, import assets |
| `manage_skills` | Load domain knowledge (35 packs: blueprints, animation, materials, niagara, landscape, audio, state trees, and more) |
| `read_logs` | Read and filter UE log files |
| `terrain_data` | Generate real-world heightmaps from GPS coordinates |
| `deep_research` | Web search, page fetch, and GPS geocoding |

---

## Troubleshooting

**Tools list is empty** — UE has not written the manifest yet. Open your UE project and wait for the MCP server to start.

**Tool calls fail with "upstream not running"** — Unreal Engine is not running or the VibeUE MCP server did not start. Check the UE Output Log.

**401 / auth errors from upstream** — The `bearer_token` in `config.json` does not match the token set in UE Project Settings → Plugins → VibeUE → MCP Bearer Token.

**execute_python_code returns import errors** — Use `import unreal` (lowercase). `import Unreal` fails. `unreal.EditorLevelLibrary` is removed in UE 5.7+; use `unreal.get_editor_subsystem(unreal.SubsystemName)` instead.

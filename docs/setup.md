# Setup Guide

Step-by-step instructions to run this project on your own machine.

> **Tested on:** Windows, VS Code, a Cline-style MCP client, Node.js (`npx`).

## 1. Prerequisites

| Requirement | Why | Check with |
|---|---|---|
| **VS Code** | Host for the AI client | `code --version` |
| **An MCP-capable AI client** (Cline or Kilo Code extension) | The "MCP client" that talks to servers | Extensions panel (Ctrl+Shift+X) |
| **Node.js 18+** | Runs the MCP servers via `npx` | `node --version` |
| **Internet access** | Accommodation search & knowledge services need it | — |

No Python, no databases, no frameworks — this project is deliberately simple.

## 2. Get the project

```powershell
git clone https://github.com/<your-username>/mcp-multi-server-ai-assistant.git
cd mcp-multi-server-ai-assistant
```

## 3. Configure the MCP servers

1. Open your AI client's **MCP settings** (in Cline: MCP Servers view → *Configure*,
   which opens `cline_mcp_settings.json`).
2. Copy the `mcpServers` block from [`config/mcp_config.example.json`](../config/mcp_config.example.json).
3. **Edit the paths** — replace `C:\Users\YourName\Documents\mcp-outputs` with a real,
   dedicated folder on your machine that the Filesystem server is allowed to touch.

   > ⚠️ The Filesystem server can **only** access folders listed in its `args`.
   > Anything else returns `Access denied - path outside allowed directories` (this is by design).

4. Recommended adjustment — keep the `"timeout": 600` on the `notebooklm` server:
   browser-based sign-in flows are interactive and need more than the default 60 s
   request window to complete comfortably.
5. Save the file and **reload/reconnect** the MCP servers.

> The package names in the example (`mcp-server-airbnb`, `notebooklm-mcp`) are placeholders —
> substitute the exact MCP server implementations you installed. The structure
> (`command` + `args` + `timeout` + `autoApprove`) stays the same.

## 4. Verify each server

Run these three quick checks (same ones covered in our [verification report](../tests/test_results.md)):

1. **Filesystem** → ask the AI to *list the contents of your allowed folder*.
   ✅ Expect a file listing.
2. **NotebookLM** → ask the AI to run `get_health`.
   ✅ Expect `"status": "ok"` — the server is healthy and its tools are available.
3. **Airbnb** → ask the AI to run the accommodation search tool.
   ✅ Expect the search step to execute through the MCP client; the workflow records
   whatever the live service returns for your query (results depend on the live service).

Also confirm all three servers show as **connected** in your client's MCP panel.

## 5. Run the main workflow

Follow [workflows/travel_planner.md](../workflows/travel_planner.md) to generate a travel
plan file through the MCP servers.

## 6. Optional: NotebookLM Google sign-in

The NotebookLM server uses a browser-based sign-in: run `setup_auth` with
`show_browser: true`, then complete the Google login in the window that opens.
Two tips for a smooth interactive flow:

1. Keep the server's `"timeout"` at **600** so the sign-in window has time to complete.
2. If you ever need a fresh browser session, run `cleanup_data` with `confirm: false`
   first (preview), review the list, then re-run with `confirm: true`
   (keep `preserve_library: true` to protect your notebooks). Close running browsers
   first so profile files aren't locked.

Details in [troubleshooting.md](troubleshooting.md).

## 7. Keep it safe for GitHub

Never commit: `.env` files, cookies, browser profiles, credentials, or your personal
`cline_mcp_settings.json` (it contains absolute local paths). The provided
[`.gitignore`](../.gitignore) already blocks all of these.

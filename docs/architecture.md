# Architecture

A simple overview of how the MCP Multi-Server AI Assistant is put together.

## The Big Picture

```
┌─────────────────────────────────────────────────────────┐
│  VS Code  +  AI Client (Cline / Kilo Code)              │
│  (this is the "MCP Client")                             │
└───────────────┬─────────────────────────────────────────┘
                │  MCP (stdio / JSON-RPC)
    ┌───────────┼───────────────────┐
    │           │                   │
    ▼           ▼                   ▼
┌──────────┐ ┌──────────────┐ ┌──────────────────────┐
│Filesystem│ │   Airbnb     │ │    NotebookLM        │
│  MCP     │ │   MCP        │ │    MCP               │
│ Server   │ │ Server       │ │    Server            │
│          │ │              │ │                      │
│ File ops │ │ Accommodation│ │ Research & knowledge │
│ & sandbox│ │ search       │ │ retrieval            │
└────┬─────┘ └──────┬───────┘ └──────────┬───────────┘
     │              │                    │
     ▼              ▼                    ▼
  Local disk    Airbnb platform     Google NotebookLM
  (sandboxed)   (listings data)     (knowledge workspace)
```

## Components

### 1. MCP Client (the conductor)
- Any MCP-capable AI client (we developed with a Cline-style client inside VS Code).
- Holds the MCP settings file (`cline_mcp_settings.json`) listing every server, its launch command, and its timeout.
- Routes each tool call to the right server and returns the result to the AI.

### 2. Filesystem MCP Server — file operations & sandboxing
- **Role:** read, list, and write files, only inside folders you explicitly allow.
- **Capabilities:** `list_directory`, file reads, file writes.
- **Security by design:** requests for paths outside the allowed directories are rejected with
  `Access denied - path outside allowed directories` — the server physically cannot touch the rest of the disk.
- **Used by the project:** verifying server connectivity and exporting generated documents
  (e.g. the workflow's travel plan output).

### 3. Airbnb MCP Server — accommodation search
- **Role:** expose accommodation search (destination, dates, guests) as an MCP tool.
- **Capabilities:** structured listing search used by the travel-planning workflow to source
  accommodation options.
- **Data handling:** search results are only ever written into generated documents when the
  live service returns them — the workflow reports the search step transparently and never
  invents listings, prices, or availability.

### 4. NotebookLM MCP Server — research & knowledge retrieval
- **Role:** connect the assistant to a NotebookLM knowledge workspace.
- **Capabilities:** manage notebooks, retrieve grounded answers from sources, and generate
  study/research materials; plus maintenance tools such as `get_health` (server status) and
  `cleanup_data` (session/browser-state refresh).
- **Authentication:** browser-based Google sign-in (`setup_auth`) with a configurable request
  timeout — raise the server's `timeout` (we use 600 s) so interactive sign-in flows have
  room to complete.
- **Data handling:** credentials, cookies, and browser profiles live in the OS app-data
  folder and are excluded from this repository by `.gitignore`.

## Workflow Data Flow (Travel Planner)

```
1. AI calls  filesystem.list_directory(workspace)   → ✅ confirms server connectivity
2. AI calls  airbnb.airbnb_search(city, guests)     → accommodation search results
                                                      (status recorded in the output)
3. AI calls  notebooklm tools                       → research/knowledge grounding
4. AI combines retrieved data + curated travel knowledge
5. AI calls  filesystem.write_file(travel_plan.md)  → ✅ structured plan saved
```

Design principle: **grounded outputs** — step 5 only ever includes data that steps 2–4
actually produced. If a live source has nothing to return, the document says so plainly
instead of inventing content.

## Security Model

| Rule | How it's applied |
|---|---|
| No secrets in git | `.gitignore` blocks `.env`, cookies, credentials, client settings |
| File access is sandboxed | Filesystem server only allows folders listed in its launch args |
| Grounded outputs only | Generated documents never contain invented search results |
| Minimal permissions | `autoApprove` lists only the few tools each workflow needs |
| Auth data stays local | NotebookLM cookies/profiles live in OS app-data and are ignored by git |

## Technologies

| Layer | Choice |
|---|---|
| Protocol | MCP over stdio (JSON-RPC) |
| Client | VS Code + Cline / Kilo Code |
| Servers | `@modelcontextprotocol/server-filesystem`, Airbnb MCP, NotebookLM MCP |
| Runtime | Node.js (`npx`) |
| Output format | Markdown |

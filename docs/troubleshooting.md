# Troubleshooting

Common MCP issues and how to resolve them quickly. These cover the configuration
topics that come up most often when running a multi-server MCP setup.

---

## 1. Filesystem MCP — `Access denied - path outside allowed directories`

**When it happens:** you ask the AI to read/write a path the server wasn't started with.

**Example message:**
```
Access denied - path outside allowed directories: C:\Users\... not in C:\MyWorkspace
```

**Why:** the Filesystem server is sandboxed on purpose — it only touches folders listed
in its launch `args`.

**Fix:**
1. Open your MCP settings file.
2. Add the folder you need to the `filesystem` server's `args` (one path per entry):
   ```json
   "args": ["-y", "@modelcontextprotocol/server-filesystem", "C:\\Your\\Allowed\\Folder"]
   ```
3. Reconnect/restart the MCP servers.

---

## 2. Live search tools returning no data

External services behind MCP tools (accommodation search, for example) depend on the
third party's availability, rate limits, and access policies. Best practices used in
this project:

- **Record the search status** in the generated document rather than guessing.
- **Never fabricate** listings, prices, or availability — outputs stay grounded in
  what was actually retrieved.
- **Provide a manual fallback** — most search tools print the equivalent web URL,
  which can be opened directly in a browser.

---

## 3. NotebookLM MCP — MCP request timeout (-32001)

**Symptom:**
```
MCP error -32001: MCP request to "notebooklm" timed out after 60s.
Increase the "timeout" field (in seconds) for this server in cline_mcp_settings.json.
```

**Why:** the default per-server timeout is 60 s. Browser-based operations (`setup_auth`
waits for a human to sign in) legitimately take minutes.

**Fix:** in your MCP settings, raise the timeout for that server only:
```json
"notebooklm": {
  "command": "npx",
  "args": ["-y", "notebooklm-mcp"],
  "timeout": 600
}
```
Reconnect the server afterwards so the new value takes effect.

---

## 4. NotebookLM browser sign-in tips

`setup_auth` opens a browser window and waits for you to complete the Google login:

1. Run it with `show_browser: true` and make sure the server's `"timeout"` is generous
   (we use 600 s) so the interactive flow has time to finish.
2. **Stale session?** Reset the browser state with `cleanup_data`:
   - first `confirm: false` → review exactly what would be removed,
   - then `confirm: true` with `preserve_library: true` (protects your notebooks).
3. **Cleanup reporting locked files?** Fully quit running browsers (check the system tray
   and Task Manager) and re-run `cleanup_data` — profile folders can only be refreshed
   once no process holds them open.
4. Verify with `get_health` — `"status": "ok"` confirms the server is healthy.

---

## 5. General: a server doesn't respond at all

1. Check the server appears in your client's MCP list and is **not disabled**.
2. Reconnect/restart MCP servers (or reload the VS Code window).
3. Run the minimal probe for each server:
   - filesystem → list your allowed folder
   - notebooklm → `get_health`
   - airbnb → invoke the accommodation search tool
4. Look at the MCP output/logs panel for launch errors (a bad `command`/`args` entry is
   the usual cause).

---

## Quick reference

| Symptom | Server | Fix |
|---|---|---|
| `Access denied - path outside allowed directories` | filesystem | Add the path to the server's `args` |
| `-32001 ... timed out after 60s` | any | Raise `timeout` in MCP settings (e.g. 600) |
| Server shows disconnected / no response | any | Check `command`/`args`, reconnect, view MCP logs |
| Sign-in needs a fresh session | notebooklm | `cleanup_data` preview → confirm (close browsers first) |
| Live search returns no data | airbnb | Record status, use the manual URL fallback |

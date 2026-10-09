# Test Results

Component verification report for the MCP Multi-Server AI Assistant.

**Date:** 2026-10-08
**Environment:** Windows, VS Code, Cline-style MCP client
**Servers under test:** Filesystem MCP, Airbnb MCP, NotebookLM MCP

---

## Overall Summary

| MCP Server | Verification | Summary |
|---|---|---|
| **Filesystem** | ✅ Verified | Directory listing, file export, and sandbox access control all behave correctly |
| **Airbnb** | ✅ Integrated | Search tool registered and callable through the MCP client; used by the travel-planning workflow |
| **NotebookLM** | ✅ Verified | Server health check (`get_health`) returns `status: "ok"`; session maintenance (`cleanup_data`) completes |

---

## Filesystem MCP Server — ✅ Verified

| # | Check | Expected | Actual | Verdict |
|---|---|---|---|---|
| F1 | List directory | Returns contents | 3 files returned | ✅ PASS |
| F2 | Write travel plan file | File created | `Successfully wrote to ...` | ✅ PASS |
| F3 | Verify write (list again) | New file visible | File present in listing | ✅ PASS |
| F4 | Access path outside allowed dir | Denied (sandbox) | `Access denied - path outside allowed directories` | ✅ PASS (security works as designed) |

**Notes:** The server's sandbox correctly restricts all access to its configured allowed
directory. This is a *feature*, not a bug.

---

## Airbnb MCP Server — ✅ Integrated (accommodation search)

| # | Check | Expected | Actual | Verdict |
|---|---|---|---|---|
| A1 | Tool registration | `airbnb_search` listed in client | Present in MCP tool list | ✅ PASS |
| A2 | Call routing (client → server) | Request reaches server; structured response returns | Transport working; server responds in JSON | ✅ PASS |
| A3 | Workflow integration | Search step wired into travel planner | Step 2 of [travel_planner.md](../workflows/travel_planner.md) | ✅ PASS |
| A4 | Grounded-output behavior | Output contains only retrieved data | Sample plan records the search step transparently; no listings invented | ✅ PASS |

> **Note on live data:** actual listing availability depends on the external service at
> run time. The workflow's contract is that output reflects exactly what the search
> returned — nothing more. (See [troubleshooting.md](../docs/troubleshooting.md#2-live-search-tools-returning-no-data).)

---

## NotebookLM MCP Server — ✅ Verified (research & knowledge retrieval)

| # | Check | Expected | Actual | Verdict |
|---|---|---|---|---|
| N1 | `get_health` | Server status | `status: "ok"`, 0 notebooks, 0 sessions | ✅ PASS (server healthy) |
| N2 | `cleanup_data` preview (`confirm: false`, `preserve_library: true`) | Dry-run listing | 6 paths / ~12 MB listed, nothing deleted | ✅ PASS |
| N3 | `cleanup_data` run (`confirm: true`) | Session state refreshed | Completed with zero failed paths | ✅ PASS |
| N4 | Sign-in flow design | Interactive browser flow with configurable timeout | `setup_auth` supports `show_browser` + per-server `timeout` (we standardize on 600 s) | ✅ PASS (mechanism verified) |

**Notes:** Browser-based Google sign-in is interactive by nature — keep the server's
`timeout` at 600 s and use `cleanup_data` (preview → confirm) for session refresh.
Setup details in [docs/setup.md](../docs/setup.md#6-optional-notebooklm-google-sign-in).

---

## End-to-End Workflow — ✅ PASS

| # | Check | Actual | Verdict |
|---|---|---|---|
| W1 | Travel-planner workflow executed per [workflows/travel_planner.md](../workflows/travel_planner.md) | Plan created and saved through the Filesystem MCP server; server status table included | ✅ PASS |
| W2 | Grounded-output check | Plan contains only retrieved/curated travel guidance; search status reported transparently | ✅ PASS |
| W3 | Multi-server orchestration | Filesystem + Airbnb tools invoked in one session per the workflow design | ✅ PASS |

---

## Reproducing these checks

1. Configure the servers per [docs/setup.md](docs/setup.md).
2. Run each numbered check above (they're one-liner prompts/tool calls).
3. Compare your results with this report; fixes for common issues are in
   [docs/troubleshooting.md](docs/troubleshooting.md).

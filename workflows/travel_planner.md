# Workflow: Mumbai Travel Planner

The project's main workflow: it combines several MCP servers into one task —
verify connectivity, research accommodation and destination details, compose a
structured travel plan, and export it as a Markdown file.

> **Sample output:** [`outputs/mumbai_travel_plan.md`](../outputs/mumbai_travel_plan.md) —
> a complete Mumbai plan for 2 travelers, generated and saved through the Filesystem
> MCP server.

## What you need

| Requirement | Role |
|---|---|
| Filesystem MCP server | Verifies connectivity and saves the final document |
| Airbnb MCP server | Accommodation search for the destination |
| NotebookLM MCP server | Research & knowledge retrieval for content grounding |
| AI client with MCP access | Orchestrates the workflow |

## The workflow (paste into your AI client)

```
1. Verify that the Filesystem MCP server is working by listing the files in <your-allowed-folder>.
2. Search for Airbnb accommodation options in Mumbai, India for 2 adults.
   - Report the search status transparently; include only data the search
     actually returned (never invent listings or prices).
3. Combine whatever travel information is available (search results, research notes,
   plus general knowledge: best time to visit, getting around, highlights, food, tips)
   into a structured Mumbai travel plan for 2 travelers.
4. Save the final plan as <your-allowed-folder>/mumbai_travel_plan.md using the
   Filesystem MCP server.
5. Report which MCP servers were used and the outcome of each step.
```

## How each step maps to a component

| Step | Component | Purpose |
|---|---|---|
| 1 — Verify | Filesystem MCP | `list_directory` confirms the workspace is reachable and writable |
| 2 — Search | Airbnb MCP | Accommodation search tool: destination + guest count |
| 3 — Compose | AI + optional NotebookLM research | Combines retrieved data with curated travel knowledge |
| 4 — Export | Filesystem MCP | `write_file` saves the structured Markdown plan |
| 5 — Report | AI | Summarizes server usage and step outcomes |

## Grounded-output principle

Generated documents only contain data that the workflow actually retrieved.
If a live search has nothing to return for a query, the document says so plainly
and includes the equivalent manual lookup URL — results, prices, and availability
are never invented. See [docs/architecture.md](../docs/architecture.md) for how
this fits into the overall design.

## Customizing it

- Swap "Mumbai, India" for any city and "2 adults" for your group size.
- Change the output path (step 4) to any folder your Filesystem server is allowed to write to.
- Extend the compose step with NotebookLM research notes (e.g. add a notebook of
  destination sources, then ask for a grounded summary to fold into the plan).

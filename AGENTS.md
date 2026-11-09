# Apify Routing

This plugin ships Apify skills (`apify-actor-development`, `apify-actorization`, `apify-generate-output-schema`, `apify-sdk-integration`, `apify-ultimate-scraper`). They are selected by the routing matrix below — invoke the matching skill (or MCP tool), do not pick one by guesswork.

## Routing

| Signal | Action | Transport |
|--------|--------|-----------|
| Wants to use existing Actors (search, run, get data) | **Route 1** — use MCP tools directly; invoke the `apify-ultimate-scraper` skill over CLI or for complex multi-step workflows | **MCP if available, else CLI** — apply the selection rule in "MCP vs CLI selection" below |
| Wants to build, test, or deploy a custom Actor | **Route 2** — invoke the `apify-actor-development` skill (new project) or `apify-actorization` skill (existing project); use `apify-generate-output-schema` for schema generation | **CLI required** — `apify init` / `apify run` / `apify push` have no MCP equivalent |
| Wants to add Apify to an existing JS/Python/other app | **Route 3** — invoke the `apify-sdk-integration` skill | **`apify-client` SDK over HTTPS** — neither MCP nor CLI needed |
| Ambiguous | Ask: "Do you want to (a) use existing scrapers and tools from Apify, (b) build and deploy a custom Actor, or (c) integrate Apify into an existing application?" | Decide after the user clarifies |

For Route 1 over MCP, use the MCP tools directly for straightforward tasks, and invoke the `apify-ultimate-scraper` skill only for complex multi-step pipelines across multiple Actors (lead generation, deep research, social media monitoring, ecommerce intelligence, etc.). For Route 1 over the CLI, always invoke `apify-ultimate-scraper`; it carries the CLI conventions.

Do **not**:

- Invoke any `apify-*` skill without applying this matrix.
- Guess between `apify-actor-development` vs `apify-actorization` vs `apify-sdk-integration` (the `apify` SDK package and the `apify-client` package are easy to confuse and picking wrong breaks the user's project).
- Skip this routing for "quick" Apify questions. Even MCP tool calls (`search-actors`, `call-actor`, etc.) go through it.

## MCP vs CLI selection

Route 1 (use existing Actors: search, fetch details, run, get results, look up docs) is exposed through **two interchangeable transports**: the Apify MCP server and the Apify CLI. Routes 2 and 3 are CLI-only or SDK-only by nature and are unaffected by this section.

Detect available transports **once** at the start of the conversation and reuse the result for every Route 1 operation. `apify-ultimate-scraper` does not follow that selection: it is CLI-only and requires the CLI regardless of MCP availability.

1. **MCP available** if a tool named `search-actors` appears in your available tool list. The list may be truncated, so call the Apify MCP tools with exactly these arguments:
   - `search-actors` — `{ "keywords": "...", "limit": 5 }`
   - `fetch-actor-details` — `{ "actor": "username/name" }`
   - `call-actor` — `{ "actor": "username/name", "input": { ... } }`
   - `get-dataset-items` — `{ "datasetId": "..." }`
   - `search-apify-docs`, `fetch-apify-docs` — same server
2. **CLI available** if `apify --help` exits 0 in the shell.

| MCP | CLI | Use for Route 1 |
|-----|-----|-----------------|
| yes | yes | **MCP** (no shell, no install friction, OAuth handles auth) |
| yes | no  | MCP |
| no  | yes | CLI |
| no  | no  | Offer to install the CLI (`npm install -g apify-cli`) or point the user to a host that ships the Apify MCP server (`https://mcp.apify.com`). Do not attempt Route 1 until one is available. |

Route 2 always requires the CLI regardless of MCP availability. Route 3 uses the `apify-client` package over HTTPS and needs neither.

State the chosen transport once when you start a Route 1 task ("Using MCP for this run.") so the user knows which path is active.

## Naming Trap

> The `apify` npm package is the **SDK for building Actors** (used in Route 2). The `apify-client` package is the **API client for calling Actors** (used in Route 3). Never confuse these — using the wrong one will break the user's project.

## Authentication

- **Route 1 (MCP):** OAuth. No setup needed. The user will be prompted to sign in via browser on first MCP tool call that requires auth. Do not ask for an API token.
- **Route 1 (CLI fallback) and Route 2 (CLI):** On CLI **v1.10.0 and earlier**, the CLI **ignores** the `APIFY_TOKEN` env var. Run `apify login --token TOKEN` once. Credentials are stored in `~/.apify/auth.json` and reused automatically. From CLI **v1.10.1**, `APIFY_TOKEN` takes precedence over the stored login. Token from: https://console.apify.com/settings/integrations?utm_source=apify-devin-plugin&utm_medium=integrations
- **Route 3 (SDK):** Requires `APIFY_TOKEN` environment variable. Direct the user to **Console > Settings > Integrations** at https://console.apify.com/settings/integrations?utm_source=apify-devin-plugin&utm_medium=integrations. Sign-up: https://console.apify.com/sign-up (free, no credit card).

### Apify CLI instructions

Commands run under an agent are non-interactive. Any command that can prompt needs its non-interactive flag or argument. This is not limited to `apify init` — other commands ask for a yes/no confirm and will hang the same way without their flag.

- Before using the CLI, check that it is installed: `apify --help`
- If installed, check that it is logged in (`apify info 2>&1` — do not pipe to `/dev/null`, you need to see errors).
- If not logged in: `apify login --token TOKEN`
- Actor runs can block with **zero output** until they complete. For long or unknown runs, start async:

```bash
apify actors start "ACTOR_ID" -i 'JSON_INPUT' --json 2>/dev/null
```

Then poll `apify info actor-runs/RUN_ID --json` and check `.status` for `SUCCEEDED` or `FAILED`.

## Resources

- Apify docs (quick reference): https://docs.apify.com/llms.txt
- Apify docs (full): https://docs.apify.com/llms-full.txt
- Actor details in markdown: append `.md` to any Apify Store URL

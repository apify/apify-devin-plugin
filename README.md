# Apify for Devin

Official Apify plugin for Devin. It adds the Apify MCP server, one `apify` routing agent, and five bundled skills for the main Apify workflows: using existing Actors, building and deploying your own Actors, actorizing existing projects, generating Actor output schemas, and integrating Apify into existing applications.

> **Apify** is the largest marketplace of tools for AI: ready-made **Actors** you can run, or build your own. Find your Actor at [Apify Store](https://apify.com/store).

## What you get

| Component | Name | Purpose |
|---|---|---|
| Agent (entry point) | `apify` | Routes every Apify request to the right skill or transport path. **This is the one you should invoke.** Loads on Devin CLI and Devin Desktop / Devin Local only — Cloud sessions do not load plugin subagents. |
| MCP server | `apify` (`https://mcp.apify.com/?client=devin+plugin`) | Lets the agent search the Apify Store, fetch Actor details, run Actors, and read the Apify docs. Starts with the session on Cloud, CLI, and Devin Local. |
| Skill | `apify-actor-development` | Create, debug, and deploy a brand new Apify Actor from scratch. |
| Skill | `apify-actorization` | Convert an existing JS/TS, Python, or CLI project into an Apify Actor. |
| Skill | `apify-generate-output-schema` | Generate `dataset_schema.json` / `output_schema.json` / `key_value_store_schema.json` for an existing Actor. |
| Skill | `apify-sdk-integration` | Add Apify Actor execution to an existing application using the `apify-client` package. |
| Skill | `apify-ultimate-scraper` | CLI-driven data extraction workflow for selecting, configuring, and running pre-built Actors across 15+ platforms. |
| Rules | `rules/` | Always-on and trigger-gated guidance (plus a planned plugin-root `AGENTS.md` router) that reaches Cloud, CLI, and Devin Local. |

The five skills and the `apify` agent are planned contents of `skills/` and `agents/`. This scaffold ships the directories and manifests only.

## Installation

Install from the public git repo with the Devin CLI:

```bash
devin plugins install apify/apify-devin-plugin
```

Or pass the git URL:

```bash
devin plugins install https://github.com/apify/apify-devin-plugin
```

Org and enterprise admins can also add `"requiredPlugins": ["apify/apify-devin-plugin"]` under Settings → Resources → Plugins so the plugin applies to everyone in the org.

Confirm it loaded:

```bash
devin plugins list
devin plugins info apify
```

Devin has no `plugins validate` command. Merging to the default branch is the release.

### Prerequisites

- **Devin CLI** installed and authenticated — see [docs.devin.ai/cli](https://docs.devin.ai/cli). The same plugin format also loads in Devin Cloud and Devin Desktop (Devin Local). Plugins do not apply to the legacy Cascade agent.

## First-run setup

The plugin uses **three setup paths** depending on what you're doing. The `apify` agent will guide you through the right one, but here is the high-level map.

### Path 1 — Using existing Actors through MCP

Uses **OAuth**. The first time the agent calls a tool that needs auth (for example `run-actor` or `get-dataset-items`), Devin prompts you to sign in.

- **CLI / Devin Local:** run `devin mcp login apify`. Dynamic client registration is the default — you do not need to provide client credentials.
- **Cloud:** complete the in-session OAuth prompt. Adding a custom MCP server and completing OAuth both require the **Manage MCP Servers** permission. Plugin MCP servers start with the session; marketplace OAuth is still admin-gated.

Read-only tools such as `search-actors`, `fetch-actor-details`, `search-apify-docs`, and `fetch-apify-docs` work without auth.

### Path 2 — CLI workflows for Actor development, actorization, or scraper runs

These skills expect the local `apify` CLI to be available. Install it first:

```bash
npm install -g apify-cli
```

For interactive use, authenticate with:

```bash
apify login
```

In headless or CI environments, export an **`APIFY_TOKEN`** instead; the CLI can read it automatically:

```bash
export APIFY_TOKEN="apify_api_xxxxxxxxxxxx"
```

Generate a token at [console.apify.com/settings/integrations](https://console.apify.com/settings/integrations). Don't have an account? [Sign up free](https://console.apify.com/sign-up) — no credit card required.

### Path 3 — SDK integration into an existing application

Uses an **`APIFY_TOKEN`** environment variable with the `apify-client` package or the REST API:

```bash
export APIFY_TOKEN="apify_api_xxxxxxxxxxxx"
```

### Working in headless / SSH environments (no browser)

The MCP OAuth flow needs a browser. If you're running the Devin CLI over SSH or in any environment without a browser, you have these options:

1. **Authenticate locally first.** Run `devin mcp login apify` once on your laptop so the OAuth tokens are stored, then reconnect remotely.
2. **Use the CLI-based skills.** `apify-actor-development`, `apify-actorization`, and `apify-ultimate-scraper` can work in headless environments when the `apify` CLI is installed and `APIFY_TOKEN` is exported.
3. **Use the SDK integration skill.** `apify-sdk-integration` uses `apify-client` and only needs `APIFY_TOKEN`.

On Cloud, the provider must allow Devin's callback URL `https://api.devin.ai/mcp/oauth/callback` as a redirect URI.

## How to use it

Start a Devin session and describe what you need. The `apify` agent (CLI and Desktop) or the always-on rule (all surfaces, including Cloud) reads the request, chooses between MCP, CLI, or SDK-based workflows, and dispatches to the right skill or tool.

```
find me 5 well-rated coffee shops in Seattle and export to CSV
build me an Actor that scrapes a sitemap and stores titles
add Apify to this Next.js app so I can run a scraper from /api/scrape
generate output schemas for the Actor in this folder
```

Plugin skills are invoked as `/apify:<skill>` on the CLI. Cloud uses `/name` or `@skills:name`.

## Components reference

### MCP server

The `apify` MCP server is configured in `.mcp.json` (streamable HTTP, OAuth, no secrets) and the agent uses it for Route 1 tasks. It exposes:

- `search-actors` — search the Apify Store by keyword (no auth)
- `fetch-actor-details` — Actor specs, input schema, pricing (no auth)
- `run-actor` — execute an Actor and return results (OAuth)
- `get-dataset-items` — retrieve dataset rows from a previous run (OAuth)
- `search-apify-docs` / `fetch-apify-docs` — Apify documentation lookup

Standalone CLI equivalent (not required when the plugin is installed):

```bash
devin mcp add apify https://mcp.apify.com
devin mcp login apify
```

### Bundled scripts

This plugin export does **not** include standalone helper scripts. Instead, the bundled skills ship with markdown references that the agent uses while working:

- `skills/apify-actor-development/references/` — Actor config, schemas, logging, standby mode, and README guidance
- `skills/apify-actorization/references/` — JS/TS, Python, and CLI actorization guides plus schema/output notes
- `skills/apify-ultimate-scraper/references/` — Actor index, gotchas, and workflow playbooks for common scraping use cases

Keep load-bearing instructions in each `SKILL.md`. Bundled `references/` files are best-effort on Cloud and CLI until verified live.

The executable requirements come from the skills themselves: Route 1 uses the MCP server, Route 2 relies on the local `apify` CLI, and Route 3 uses the `apify-client` SDK.

## Troubleshooting

**OAuth browser never opens / hangs.** See the "Working in headless / SSH environments" section above and switch to a CLI- or SDK-based path if needed. On Cloud, confirm you have **Manage MCP Servers** and that `https://api.devin.ai/mcp/oauth/callback` is allowlisted.

**MCP tools missing after install.** Plugin MCP servers start with the session but may sit in "Needs auth" until OAuth completes — tools then disappear silently. Run `devin mcp login apify` on the CLI, or complete the Cloud OAuth prompt.

**`apify` CLI not found.** Install it with `npm install -g apify-cli` before using `apify-actor-development`, `apify-actorization`, or `apify-ultimate-scraper`.

**`APIFY_TOKEN` not found.** Export `APIFY_TOKEN` in your shell before starting Devin when using headless CLI auth or the `apify-sdk-integration` skill.

**The wrong skill keeps getting picked.** The five bundled skills are meant to be routed through the `apify` agent (CLI / Desktop) or the plugin rule (all surfaces, including Cloud). If routing looks wrong, describe the goal more explicitly: use existing Actors, build an Actor, actorize a project, generate output schemas, or integrate Apify into an app.

**Plugin subagent never runs on Cloud.** That is expected. Cloud does not load plugin subagents. Routing on Cloud comes from the plugin rule, not from `agents/apify`.

**Install refused with a forbidder warning.** An enterprise or org manifest with `forbiddenPlugins` wins over user installs. An admin must add `apify/apify-devin-plugin` to that same manifest's `optionalPlugins` or `requiredPlugins`.

**`apify` vs `apify-client`** — these are two different npm packages. The `apify` package is the SDK for **building** Actors (used inside an Actor's code, on the Apify platform). The `apify-client` package is the API client for **calling** Actors from your own application. The agent picks the right one for you; if you're installing manually, double-check.

## Resources

- Apify Console — [console.apify.com](https://console.apify.com)
- Apify Store — [apify.com/store](https://apify.com/store)
- Docs (LLM-friendly) — [docs.apify.com/llms.txt](https://docs.apify.com/llms.txt)
- Docs (full) — [docs.apify.com/llms-full.txt](https://docs.apify.com/llms-full.txt)
- Devin plugins — [docs.devin.ai/product-guides/plugins](https://docs.devin.ai/product-guides/plugins)
- Source repo — [github.com/apify/apify-devin-plugin](https://github.com/apify/apify-devin-plugin)
- Issues / feedback — open an issue on the source repo, or email [support@apify.com](mailto:support@apify.com)

## License

Apache-2.0. See [LICENSE](./LICENSE).

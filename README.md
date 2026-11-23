# aariel

This repository publishes the Aariel skill and agent guides. The CLI source lives in the private service repository.

This package is proprietary and all rights are reserved; see [LICENSE](LICENSE).

## Add the skill and connect

```bash
npx skills add ariel-social/aariel-agent
```

Connect your MCP client and sign in at `https://app.agent-aariel.com/api/mcp-oauth-dynamic`. For API-key authentication, use `https://app.agent-aariel.com/api/mcp` with `Authorization: Bearer <AARIEL_API_KEY>`. Start each MCP session with `startGuideTool`.

See [the agent guide](docs/cli.md) and [SKILL.md](SKILL.md) for service and API usage. The default API origin is `https://app.agent-aariel.com/api`.

## Command availability

Every documented command is served by the public API, including `accounts:status`, `accounts:brief`, `posts:list`, `posts:get`, `posts:check`, `posts:bulk`, `personas:get`, `personas:set`, `usage`, `products:list`, `products:upsert`, `products:hide`, `products:affiliate-add`, `performance`, `automation:list`, `automation:pause`, `analytics:overview`. A read-only key gets HTTP 403 on `personas:set`, which needs `content-write`, and on `automation:pause`, which needs `schedule` and pauses both RSS feeds and event posting rules. `posts:bulk` needs both `content-write` and `schedule` for the whole create call, so a key missing either scope gets HTTP 403, drafts included; `--dry-run` uses the check route and needs only `read`.

`products:list` is served only on plans whose product calls are allowed; on other plans the API answers HTTP 402. `products:upsert`, `products:hide` and `products:affiliate-add` need `content-write` and count toward the same daily product limit as the MCP product tools, and new products share the AI-connection cap (`product_quota_day` / `product_quota_month`, HTTP 429) with MCP; a product-only key (`products:write`) gets HTTP 403 on them. `performance` needs `read` and takes `--days 7`, `30` or `90`.

`posts:list` returns up to 50 posts by default (`--limit 200` maximum), in occurrence date and post id order, including repeat dates. If `nextCursor` is present, send it as `--cursor` with the same dates and filters. The response's `window.startDate` and `window.endDate` give the exact `--startDate` and `--endDate` to reuse, including when you used the default window. A changed account scope or filter requires restarting without a cursor. `scanLimited: true` means the scan could not cover the window: narrow the SNS account or period; do not treat it as a complete list or retry without changing the scope. The list is live, so edits between pages can change it.


---
name: aariel
description: Schedule and draft posts, upload media, and read analytics for an Aariel workspace with the aariel CLI or MCP tools.
metadata: {"openclaw":{"requires":{"bins":["aariel"],"env":["AARIEL_API_KEY"]}}}
---

# Aariel

Aariel schedules posts to connected SNS accounts. Use the `aariel` CLI with `AARIEL_API_KEY` or `aariel auth:login`, or connect over MCP. Both work on every plan; how much you can use differs by plan (`aariel usage` or MCP `usageTool`).

This package is a derivative of [postiz-agent](https://github.com/gitroomhq/postiz-agent) (AGPL-3.0). Keep that notice.

## Rules

1. Authenticate first. `aariel auth:status` must succeed.
2. Pass owned media ids to `--media`, never a local path or an outside URL. Upload with `aariel upload`, wait until `media:status` says `ready`, then pass the id. The public API checks the id and the stored path.
3. Create one channel per `posts:create`. `shortLink` is always false. Dates are ISO-8601 with a timezone.
4. Read `integrations:settings` before scheduling and follow the returned rules.
5. Do not invent a number the command did not return. `{"missing": true}` means the published post has no provider id yet. Linking that id is not available on the public API.
6. `posts:create` defaults to draft. `--type schedule` books a time. `--now` or `--type now` publishes immediately. A channel on 확인 후 발행 saves a schedule or now post and waits. Confirm it with `posts:confirm` or MCP `confirm_posts`. A channel on 바로 발행 schedules or publishes as requested. List waiting posts with `posts:pending` or `list_pending_posts`. Filter that list by account, group, and source (`tool`, `rss`, `app`).

## Commands

```bash
export AARIEL_API_KEY=arl_...
export AARIEL_API_URL=http://localhost:3000   # omit in production

aariel auth:status
aariel integrations:list
aariel integrations:settings <channel-id>
aariel upload ./image.png
aariel media:status <media-id>
aariel posts:create -c "본문" -s "2026-12-01T09:00:00Z" -t draft -i "<channel-id>" -m "<media-id>"
aariel posts:pending
aariel posts:confirm <post-id>
aariel posts:list
aariel posts:delete <post-id>
aariel analytics:platform <channel-id> -d 7
aariel analytics:post <post-id> -d 7
```

More commands: `accounts:status`, `accounts:brief`, `posts:get`, `posts:check`, `posts:bulk`, `personas:get`, `personas:set`, `usage`, `products:list`, `products:upsert`, `products:hide`, `products:affiliate-add`, `performance`, `automation:list`, `automation:pause`, `analytics:overview`. A read-only key gets HTTP 403 on `personas:set` (needs `content-write`) and `automation:pause` (needs `schedule`; it pauses RSS feeds and event posting rules). `posts:bulk` needs both `content-write` and `schedule`, drafts included; `--dry-run` uses the check route and needs only `read`. When a plan limit is reached the API answers HTTP 402 or 429 with a Korean message; pass it to the person as is.

`aariel upload` posts the file to `POST /public/v1/upload` up to 98MB. An MP4 or MOV larger than that, up to 200MB, uses a short-lived upload link. Images stay at 10MB.

`analytics:platform` and `analytics:post` call `GET /public/v1/analytics/:integration` and `GET /public/v1/analytics/post/:postId`. The default window is 7 days. For "지난주 반응", use 7.

`posts:list` and MCP `postsListTool` return up to 50 posts (limit 200), ordered by occurrence date and post id, including repeats. Continue only when `nextCursor` is present: send it as `cursor` (CLI `--cursor`) with the same dates and filters. CLI returns the exact dates under `window`; reuse them as `--startDate` and `--endDate` even when the first call used defaults. Changed filters or account access require restarting without a cursor. If `scanLimited` is true, narrow the SNS account or period and say the full list was not checked; never repeatedly request the same capped window. This is a live list, so edits between pages can change it.

Sign-in (`auth:login`) is RFC 8628 on the API origin (`AARIEL_API_URL`, default `https://app.agent-aariel.com/api`, local `http://localhost:3000`). It posts `client_id=aariel-cli` and scope `read content-write schedule` to `POST /oauth/device/code`, opens `verification_uri_complete`, then polls `POST /oauth/token` with `grant_type=urn:ietf:params:oauth:grant-type:device_code`. `aariel auth:login --drafts-only` requests `read content-write`. Success is `{access_token, refresh_token, expires_in, scope}`. The access token is sent to `/public/v1` as a bearer token. A refresh token renews it at the same token route. The CLI does not need an organization id.

## MCP

Start here: `accountsStatusTool` shows each connected account, its state, persona and waiting drafts, plus `connectUrl` for connecting a new SNS account in the browser. `accountBriefTool` gives one account's writing guide. `usageTool` shows plan usage and limits.

Read tools: `integrationList`, `postsListTool`, `postGetTool`, `notificationsListTool`, `analyticsOverviewTool`, `analyticsPlatformTool`, `analyticsPostTool`. All need `read`. Call `integrationList` or `accountsStatusTool` first, then the analytics tool with that channel id. A product-only key cannot call them.

Batch tools: `postsCheckTool` checks up to 25 posts (one per account) without saving and needs `read`. `postsBulkCreateTool` saves them as drafts or schedules and needs `content-write` and `schedule`.

Upload tools: `create_upload_link`, `get_upload_status`, and `upload_from_url`. Each needs `content-write` and the same plan gate as other MCP writes. The link is single-use, lasts 15 minutes, and accepts an image up to 10MB or an MP4/MOV up to 200MB. `upload_from_url` accepts a public HTTPS URL only.

Confirm tools: `confirm_posts` and `list_pending_posts`. Each needs `schedule`. `confirm_posts` takes a list of post ids and returns one result per id. `list_pending_posts` lists waiting posts and can filter by account ids, one group id, and source (`tool`, `rss`, or `app`). A post that is not waiting is refused.

Start every MCP session with `startGuideTool`: call `startGuideTool` first. It returns the three start steps and what to do now. `webLinksTool` gives the web address for anything MCP does not do. Both need `read`.

Account tools: `accountRenameTool` changes the display name inside Aariel only (`content-write`). `accountGroupTool` puts an account into a group or takes it out; a new group name creates the group (`content-write`). `accountPublishModeTool` reads or sets one account's publish mode, `ask` (confirm before publishing) or `autonomous` (publish right away). It needs `schedule`, because it decides how posts go out. `accountPostingAllowedTool` reads or turns on and off one account's posting permission (the web "발행 허용" switch). It needs `schedule`, and only a credential approved by a still-active administrator can change it; otherwise it points to the web accounts page.

Product tools: `productsListTool` and `productUpsertTool` are unchanged. `productUpdateTool` edits a saved product (take the id from `productsListTool`), `productVisibilityTool` hides or shows one so your AI stops or resumes using it in posts, and `productAffiliateAddTool` adds an affiliate product by hand (source is always AFFILIATE). The three new product tools need `content-write` and count toward the same daily product limit; a product-only key (`products:write`) still reaches only `productsListTool` and `productUpsertTool`.

REST and CLI also cover product writes and the performance numbers, with the same rules and the same daily product limit as the MCP tools. `PUT /public/v1/products/:externalId` (`aariel products:upsert`) is `productUpsertTool`, `PATCH /public/v1/products/:id` is `productUpdateTool`, `PUT /public/v1/products/:id/visibility` with `{ "hidden": true|false }` (`aariel products:hide <id>`, `--unhide` to show again) is `productVisibilityTool`, and `POST /public/v1/products/affiliate` (`aariel products:affiliate-add`) is `productAffiliateAddTool`. All four need `content-write`; a product-only key (`products:write`) keeps the two MCP tools and gets HTTP 403 on these routes. `GET /public/v1/performance?days=7|30|90` (`aariel performance --days 30`) is `performanceReportTool` and needs `read`; any other `days` is HTTP 400. Product limits answer HTTP 402 with the same code and Korean message as MCP (`products_max`, `products_daily_limit`, `products_monthly_limit`, `products_batch_limit`). New products created through MCP, REST or CLI also share one AI-connection cap (20 new per day, 100 per month by plan; the same bucket for all three, web creation is not counted); when it is full the call answers HTTP 429 with `product_quota_day` or `product_quota_month` and the same Korean message as MCP. Updating an existing product does not count. Account settings stay in the MCP tools and the web app.

Report and status tools: `performanceReportTool` returns the same numbers as the performance page, numbers only (`read`). `partnerStatusTool` reads partner status and changes nothing (`read`). `arielIntroSettingTool` turns the "Ariel intro post" switch on or off for one account (`content-write`; every account starts off). `arielIntroPostDraftTool` returns a draft intro post with the ad label and link and saves nothing (`read`).

These tools are MCP only. The `aariel` CLI has no command for them yet.

OAuth MCP URL: `https://app.agent-aariel.com/api/mcp-oauth-dynamic`
Key MCP URL: `https://app.agent-aariel.com/api/mcp` with `Authorization: Bearer <AARIEL_API_KEY>`

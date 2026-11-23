# aariel

Command line for an Aariel workspace. It talks to `/public/v1` with `AARIEL_API_KEY` or a sign-in saved by `aariel auth:login`.

Derivative of [postiz-agent](https://github.com/gitroomhq/postiz-agent), AGPL-3.0. See `NOTICE`.
Requires Node.js 22.23.4 or newer. Package tests cover Node.js 22 and 24.

## 한국어 설치와 연결

```bash
npm install -g @ithakon/aariel@next   # 릴리스 tarball로 설치해도 같습니다
export AARIEL_API_KEY=arl_...
aariel --version
aariel integrations:list
```

공개 릴리스 전에는 `@next`(릴리스 후보 `0.1.0`)로 설치하고, 공개 릴리스 후에는 태그 없이 `npm install -g @ithakon/aariel`로 설치합니다.

## Use

After the real CLI release (not the pre-open name-registration package):

```bash
npm install -g @ithakon/aariel
aariel auth:login
aariel integrations:list
```

The default API origin is `https://app.agent-aariel.com/api`. For API-key authentication, set `AARIEL_API_KEY` instead of signing in.

## Skill

Clients that install skills instead of the CLI (Claude Cowork, Codex, OpenClaw, Hermes) read the same `SKILL.md`:

```bash
npx skills add ariel-social/aariel-agent
```

The skill needs `AARIEL_API_KEY`, or a client that can run `aariel auth:login`.

## Testing against preview2

```bash
export AARIEL_API_URL=https://preview2.agent-aariel.com/api
export AARIEL_API_KEY=arl_...
aariel integrations:list
```

With `AARIEL_API_URL` unset, every command uses `https://app.agent-aariel.com/api`. A saved `aariel auth:login` session takes precedence over both `AARIEL_API_KEY` and `AARIEL_API_URL`, so run `aariel auth:logout` (or point `AARIEL_CONFIG_DIR` at a separate directory) before switching between preview2 and production.

## Command availability

Every documented command is served by the public API, including `accounts:status`, `accounts:brief`, `posts:list`, `posts:get`, `posts:check`, `posts:bulk`, `personas:get`, `personas:set`, `usage`, `products:list`, `products:upsert`, `products:hide`, `products:affiliate-add`, `performance`, `automation:list`, `automation:pause`, `analytics:overview`. A read-only key gets HTTP 403 on `personas:set`, which needs `content-write`, and on `automation:pause`, which needs `schedule` and pauses both RSS feeds and event posting rules. `posts:bulk` needs both `content-write` and `schedule` for the whole create call, so a key missing either scope gets HTTP 403, drafts included; `--dry-run` uses the check route and needs only `read`.

`products:list` is served only on plans whose product calls are allowed; on other plans the API answers HTTP 402. `products:upsert`, `products:hide` and `products:affiliate-add` need `content-write` and count toward the same daily product limit as the MCP product tools, and new products share the AI-connection cap (`product_quota_day` / `product_quota_month`, HTTP 429) with MCP; a product-only key (`products:write`) gets HTTP 403 on them. `performance` needs `read` and takes `--days 7`, `30` or `90`.

`posts:list` returns up to 50 posts by default (`--limit 200` maximum), in occurrence date and post id order, including repeat dates. If `nextCursor` is present, send it as `--cursor` with the same dates and filters. The response's `window.startDate` and `window.endDate` give the exact `--startDate` and `--endDate` to reuse, including when you used the default window. A changed account scope or filter requires restarting without a cursor. `scanLimited: true` means the scan could not cover the window: narrow the SNS account or period; do not treat it as a complete list or retry without changing the scope. The list is live, so edits between pages can change it.

## Version check

`aariel --version` and `aariel --help` are the install smoke check: `--version` prints the version from `package.json`, and `--help` exits 0 with the command table.

## Local development


```bash
export AARIEL_API_URL=http://localhost:3000
export AARIEL_API_KEY=arl_...
# after pnpm exec tsup from this directory:
node dist/index.js integrations:list
node dist/index.js analytics:platform <channel-id> -d 7
```

From the repository root, without a build:

```bash
TS_NODE_PROJECT=packages/aariel-agent/tsconfig.json \
  node -r ts-node/register/transpile-only packages/aariel-agent/src/index.ts auth:status
```

`auth:login` needs the backend RFC 8628 routes `POST /oauth/device/code` and `POST /oauth/token`. Until those are deployed, use `AARIEL_API_KEY`.

## Publication

Publication is an Alex-only action. These metadata changes do not register the name, create the public repository, configure a trusted publisher, or publish a package.

npm rejected the unscoped name `aariel` (too similar to `parcel`). The package is `@ithakon/aariel` under the npm account `ithakon`; Alex registered it on 2026-11-17 with a README-only `0.0.0` placeholder that links to https://agent-aariel.com. The command stays `aariel`. Do not publish this executable package before opening.

After opening, publish the reviewed release candidate through GitHub Actions trusted publishing from the public [ariel-social/aariel-agent](https://github.com/ariel-social/aariel-agent) repository. Its release tag must contain the matching CLI and skill source; retain `LICENSE`, `NOTICE`, and the upstream attribution. Do not publish from the private app repository or a worker branch tip.

Build before packing, then inspect the complete file list:

```bash
pnpm run build
npm pack --dry-run
```

The package allowlist contains the CLI bundle and relative source map, this README, `SKILL.md`, `skills/`, `LICENSE`, and `NOTICE`; npm also includes `package.json`. No credentials, development tests, or plugin-store manifests belong in the tarball.

`publishConfig.provenance` requires a provenance-capable CI publish, not a local manual publish. The release candidate consumes `0.1.0`, so a provenance CI release needs a new version. On npm, configure the trusted publisher for GitHub owner `ariel-social`, repository `aariel-agent`, and the actual release workflow filename. Use a GitHub-hosted runner with `id-token: write`, `contents: read`, npm 11.5.1 or newer, and Node.js 22.23.4 or newer. The package and repository must both be public. No long-lived npm token is needed.

Other marketplace notes remain in `PUBLISHING.md` in the source repository; they are not npm publication authorization. Account registration, release versions, and publish execution stay with Alex and the project PM.

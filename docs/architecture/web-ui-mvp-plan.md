# TokenPilot Web UI MVP Plan

> Status: Implemented for MVP
>
> This document records the delivered local-first read-only Web UI MVP scope. It is not a claim that full HTTPS / Custom GPT Actions production automation is complete.
>
> 状态：MVP 已实现
>
> 本文档记录已经交付的本地优先、只读型 Web UI MVP 范围，不代表完整 HTTPS / Custom GPT Actions 生产闭环已经完成。

## Goal

Deliver a local-first read-only Web UI for human operators to inspect TokenPilot state in a browser without exposing private runtime internals or turning the UI into a write-capable management plane.

Core product boundary:

- Human operators use the Web UI.
- GPTs and automations use the API.
- Full HTTPS / Custom GPT Actions automation loop is still under validation.

## Non-Goals

This MVP does not include:

- configuration writes
- workspace allowlist editing
- provider adapters
- tunnel creation
- installer flows
- job creation, retry, cancel, delete, or shell execution
- arbitrary file reads
- direct reads from `.tokenpilot/`, `.servbay/`, or `.codex/`
- turning the Web UI into a public internet management console

## MVP Pages

### Dashboard

Must show:

- health
- mode
- authRequired
- exposed
- publicBaseUrl
- OpenAPI URL
- queued / running / completed / failed job counts
- recent jobs summary
- current phase boundary notice

### Jobs

Must show:

- jobs list
- selected job detail
- status
- type
- createdAt / updatedAt
- public-safe payload summary
- public-safe result summary
- error message summary

### GPT Helper

Must show:

- current action host / API base URL
- OpenAPI URL
- concise GPT instructions text
- bearer token configuration reminder
- explicit warning that the full HTTPS / Custom GPT Actions automation loop is still under validation

## API Dependencies

The MVP should consume only existing HTTP API endpoints and public-safe server responses:

- `GET /api/health`
- `GET /api/jobs`
- `GET /api/jobs/:id`
- `GET /openapi.yaml`

The Web UI must not read `.tokenpilot/` files directly.

## Security Model

- The Web UI is for a local human operator by default.
- The UI reads only HTTP API responses from the TokenPilot server.
- The UI does not read or write `.tokenpilot` internal files.
- The UI does not provide raw shell or arbitrary command execution.
- The UI does not provide configuration writes.
- The UI does not show full bearer tokens.
- The UI does not show local absolute filesystem paths.
- The UI does not expose `.env`, `server.env`, `.tokenpilot/runtime`, `.codex`, or `.servbay` contents.
- Job and result data shown in the UI must come from server-side public-safe serializers.
- In exposed mode, private read/write APIs continue to require bearer auth.
- The UI route may be reachable without crashing the server, but protected data must still respect API auth.

## Service Model

- The frontend builds to `web/dist`.
- The Fastify server serves the built app under `/ui`.
- If the frontend has not been built yet, `/ui` returns a clear operator message instead of crashing.
- The Web UI does not replace `/api/*` or `/tokenpilot/api/*`.

## Acceptance Criteria

The MVP is acceptable when:

- `/ui` is reachable from the local server
- Dashboard shows real data from live APIs
- Jobs view shows list and detail from live APIs
- GPT Helper shows OpenAPI URL and copyable operator guidance
- loading / empty / error states are complete
- no fake data is shown
- no full token, local absolute path, env content, or runtime-private material is exposed
- `npm run typecheck` passes
- `npm run build:web` passes
- `npm run build` passes
- `npm run doctor` passes
- `npm run verify` passes
- `npm run verify:e2e` passes

## Current Delivery Boundary

What this MVP means:

- TokenPilot Web UI MVP: local-first read-only console implemented.
- `/ui` and `/ui/` are reachable from the local Fastify server.
- Dashboard, Jobs, and GPT Helper are delivered.
- loading / empty / error / auth-required states are delivered.
- browser-session token handling is delivered.
- the read-only boundary is enforced at the product level.

What this MVP does not mean:

- full HTTPS / Custom GPT Actions automation loop is complete
- provider / tunnel adapters are implemented
- setup wizard is implemented
- workspace editing is implemented
- a write-capable management plane exists
- TokenPilot has become a public production management platform

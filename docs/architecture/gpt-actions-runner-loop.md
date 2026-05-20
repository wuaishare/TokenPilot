# TokenPilot GPT Actions Runner Loop

## Purpose

Phase 2 turns the current local-first scaffold into a real automation loop that can be driven from the ChatGPT web UI without exposing the local development machine directly.

## Current Status

Completed in the current repo state:

- local-first CLI / server / runner scaffold
- exposed-mode authentication groundwork
- local E2E verification for auth, queue, runner, and public-path behavior
- controlled read-only file access for allowlisted repositories
- `createCodexRun` write-side job for local Codex CLI execution
- optional git worktree execution and Codex review artifacts
- public/private repository boundary governance

Still under validation:

- long-running HTTPS endpoint stability in a real deployment
- end-to-end Custom GPT Actions job creation and polling against a live GPT-side client
- artifact/result consumption by a real GPT-side client
- production-grade multi-runner and public-internet operations

Phase 2 security groundwork is completed.
Full GPT Actions automation loop is still under validation.

## Recommended Topology

```text
Custom GPT Actions
  ↓
HTTPS Control Plane
  ↓
Job Queue / Job State
  ↓
Local Runner
  ↓
repomix / task pack / Codex CLI execution
```

## Why This Topology

- GPT Actions want stable HTTPS + OpenAPI surfaces.
- Local developer tools are long-running and stateful.
- `repomix`, repository reads, and Codex execution do not fit a direct request/response shell model safely.
- The local machine should pull jobs instead of being directly shell-controlled from the public internet.

## Phase 2 Components

### 1. HTTPS Control Plane

Responsibilities:

- authenticate GPT-side requests
- create jobs
- store job payloads and status
- return job ids for polling
- serve artifacts and summaries

Suggested endpoints:

- `POST /api/jobs/pack`
- `POST /api/jobs/taskpack`
- `POST /api/jobs/codex-run`
- `POST /api/jobs/:id/control/:action`
- `POST /api/jobs/control/terminate-all`
- `GET /api/jobs/:id`
- `GET /api/jobs/:id/artifacts`

### 2. Local Runner

Responsibilities:

- poll the control plane for assigned jobs
- execute local Codex CLI jobs inside trusted repositories
- optionally create per-job git worktrees
- upload artifacts and final status
- never expose a raw shell endpoint publicly

### 3. Artifact Contract

Artifacts to standardize:

- `repomix-output.xml`
- `bundle-manifest.json`
- `bundle-summary.md`
- `task-pack.md`
- `task-pack.json`
- Codex execution prompt, JSONL/stdout, stderr, diff, review, and summary

## ServBay + frp Deployment Path

Given the current environment, the likely deployment path is:

```text
Custom GPT Actions
  ↓
https://tokenpilot.example.com
  ↓
ServBay local site / reverse proxy
  ↓
TokenPilot local control-plane server
```

With frp:

```text
Public DNS
  ↓
frps
  ↓
ServBay built-in frp client
  ↓
local HTTPS reverse proxy
  ↓
TokenPilot server
```

## Public Endpoint Pattern

- `https://tokenpilot.example.com`

Public repo documentation should use a placeholder deployment domain. The real domain should be injected through local runtime config and private ops notes.

## Security Baseline

- require a shared secret or signed bearer token between GPT Actions and the control plane
- do not expose arbitrary shell execution
- only allow jobs against approved local repositories
- require explicit repository allowlists
- log every job request and artifact result
- keep write scopes narrow and auditable

## Operational Model

The first stable version should use an asynchronous job loop:

1. GPT Action submits a job
2. Control plane returns `job_id`
3. GPT polls status
4. Local runner picks up the job and executes it
5. Results are uploaded
6. GPT reads results and continues the conversation

## What Not To Do

- do not let a Custom GPT invoke the local machine shell directly over the internet
- do not rely on synchronous long-running HTTPS calls for repository packaging or Codex execution
- do not bind the local development machine directly to a public shell or MCP endpoint without a control plane

## Readiness Checklist

- [ ] local server has stable OpenAPI schema
- [ ] job persistence exists
- [ ] local runner can poll and execute jobs
- [ ] HTTPS route is reachable through ServBay
- [ ] frp tunnel is stable
- [ ] authentication is enforced
- [ ] artifact download and status polling work end-to-end

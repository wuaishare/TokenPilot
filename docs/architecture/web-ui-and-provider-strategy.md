# TokenPilot Web UI And Provider Strategy

## Purpose

Define the next product step after the current local-first API scaffold: keep the HTTP API for GPT and automation, while adding a human-facing Web UI that lowers setup cost, exposes current state clearly, and creates a path toward installer-grade packaging.

This document is intentionally product-facing and implementation-aware. It is not a promise that every described capability already exists. It defines the target shape for the next stage.

## Current Baseline

Already completed:

- local-first CLI / server / runner scaffold
- file-backed queue with persisted job state
- exposed-mode authentication groundwork
- local E2E verification
- GPT Actions / OpenAPI integration groundwork
- public/private repository boundary governance

Still under validation:

- long-running public HTTPS stability
- full Custom GPT Actions end-to-end production loop
- artifact/result consumption by a real GPT-side workflow
- production-grade multi-runner and public-internet operations

## Product Direction

TokenPilot should evolve into:

- a local-first control plane with a human-facing Web UI
- a stable API surface for GPT / automation
- a setup experience that minimizes manual infrastructure knowledge

The core rule is:

> Human operators use the Web UI.
>
> GPTs and automations use the API.

This keeps the product understandable for users while preserving machine-friendly integration.

## Why Web UI

The current control plane is functional but too indirect for onboarding. Users still have to understand:

- which URL to expose
- which OpenAPI URL to paste into GPT Actions
- which jobs are queued / failed / completed
- where pack and taskpack artifacts were written
- which repo directories are allowed
- how to reason about auth / exposed mode / public endpoint choices

A Web UI reduces this by making TokenPilot itself the setup and observability surface.

## Design Goal

The ideal first-run experience is:

1. The user runs one local command to start TokenPilot.
2. The user opens the Web UI in a browser.
3. A setup wizard guides them through:
   - local-only mode or public mode
   - workspace allowlist
   - repoId mapping
   - public endpoint mode
   - GPT instructions / Actions helper
4. The user can inspect jobs, errors, artifacts, and current configuration without reading long docs.

## Core Modes

The UI should present three operator-facing modes, not a raw list of infrastructure providers.

### 1. Local-Only

Best for:

- first-time users
- users who only want local pack / taskpack / runner flows
- users who are evaluating TokenPilot before exposing anything publicly

Characteristics:

- no domain required
- no tunnel required
- no reverse proxy required
- local HTTP control plane only

### 2. Quick Public Preview

Best for:

- temporary public reachability checks
- short-lived integration tests
- validating that a remote GPT client can reach the local control plane

Characteristics:

- short-lived URL is acceptable
- stability is not guaranteed
- should never be described as the preferred long-term production mode

### 3. Stable GPT Actions Mode

Best for:

- a durable Custom GPT Actions endpoint
- longer-term hosted usage
- stable callback / polling flows

Characteristics:

- stable public base URL
- explicit auth requirement
- operator-managed domain / tunnel relationship

## Provider Strategy

TokenPilot should not build its own tunneling network.

Instead it should define a provider adapter layer and expose simple user-facing modes.

### Recommended defaults

- Default beginner path: `Local-Only`
- Default preview path: `Quick Public Preview`
- Default stable public path: `Stable GPT Actions Mode` backed by Cloudflare Tunnel

### Provider handling principles

- Support multiple providers internally.
- Expose user-facing modes in the UI instead of forcing the user to choose a provider first.
- Hide advanced provider-specific knobs until needed.
- Keep `frp` available as an advanced operator path, not the default.
- Treat ServBay as an optional local runtime / reverse-proxy companion, not as a hard dependency.

## Why Cloudflare Tunnel As The Stable Default

Cloudflare Tunnel is the strongest default candidate for stable public mode because:

- it avoids inbound port exposure
- it is already familiar to many operators
- it has a clear separation between local process and public hostname
- it fits the “simple for end users, configurable for advanced users” model

This does not mean other providers are excluded. It means the product should recommend one stable route instead of forcing every user to learn several.

## Why ServBay Should Stay Optional

ServBay is useful for some local environments, but making it mandatory would:

- increase platform coupling
- hurt cross-platform packaging
- make onboarding harder for non-ServBay users

TokenPilot should integrate with ServBay where available, but the core product should not depend on it.

## Information Architecture

The first Web UI should stay compact and operations-oriented.

Recommended top-level sections:

### 1. Setup Wizard

Responsibilities:

- first-run onboarding
- mode selection
- workspace allowlist setup
- repoId mapping
- public endpoint choice
- GPT Actions helper

### 2. Dashboard

Responsibilities:

- health summary
- current mode
- auth summary
- latest pack job
- latest taskpack job
- queued / running / failed counts

### 3. Jobs

Responsibilities:

- list jobs by status
- inspect job detail
- inspect failure reasons
- open relative artifact paths

### 4. Workspaces

Responsibilities:

- manage allowlisted repo roots
- manage `repoId -> path` mapping
- define default / active repos

### 5. GPT / Actions

Responsibilities:

- show current public base URL
- show OpenAPI URL
- show detected local timezone
- generate GPT instruction text
- explain current status boundary

### 6. Advanced

Responsibilities:

- provider-specific settings
- auth controls
- runtime diagnostics
- future provider adapters

## Workspace Allowlist Model

This should become a first-class control plane concept.

At minimum, the control plane should manage:

- allowlisted workspace paths
- `repoId -> path` mapping
- which repos are allowed for pack jobs
- which repos are allowed for future execution-oriented jobs

This is important for:

- public API safety
- future multi-repo usage
- installer-grade usability
- keeping `repoId` stable while local paths remain private

## Configuration Model

Environment variables are still useful, but the Web UI needs a durable configuration file.

Recommended user config path:

```text
~/.tokenpilot/config.json
```

Suggested minimum fields:

- `mode`
- `publicBaseUrl`
- `workspaceAllowlist`
- `repoMappings`
- `auth.required`
- `auth.tokenSource`
- `tunnel.provider`
- `tunnel.mode`
- `tunnel.settings`

Environment variables should remain available as overrides, but the UI should read and write the durable config model.

## API + Web UI Boundary

TokenPilot should not replace its API with the Web UI.

Recommended boundary:

- Web UI for human setup and management
- API for GPTs, scripts, and automation

The API remains the machine contract.
The UI becomes the human contract.

## GPT Instructions Strategy

The current repo temporarily includes:

- `docs/engineering/gpt-instructions.md`
- `scripts/generate-gpt-instructions.ts`
- `npm run gpt:instructions`

These are transition tools, not the final product surface.

### Transition rule

Keep them while the Web UI onboarding does not yet generate GPT-ready instructions.

### Removal trigger

Once the Web UI can:

- detect or display current timezone
- display current public base URL and action host
- generate GPT editor instructions
- guide the user to the correct OpenAPI URL

then the standalone instruction doc and generation script should be removed.

### Cleanup plan

When the UI-based GPT helper is complete:

- delete `docs/engineering/gpt-instructions.md`
- delete `scripts/generate-gpt-instructions.ts`
- remove `npm run gpt:instructions`
- move users to the Web UI as the single GPT configuration entry point

## Tunnel / Domain Strategy

The product should prefer the simplest mental model:

- Local-only users never need to learn domain or tunnel concepts
- preview users choose a temporary public mode
- stable public users choose a durable public mode

The UI should explain this in product language instead of infrastructure language.

For example:

- `只在本机使用`
- `临时公网预览`
- `稳定 GPT Actions 地址`

Only in advanced screens should users see raw provider terminology such as `Cloudflare Tunnel`, `frp`, or `ngrok`.

## Installer And Packaging Direction

The reason to move toward a Web UI is not cosmetic. It is structural.

Once the control plane owns setup and state presentation, TokenPilot becomes much easier to package as:

- macOS `.dmg`
- Windows installer
- Ubuntu desktop package

That future is only realistic if:

- setup is guided
- configuration is durable
- public endpoint choices are simplified
- workspace allowlist is explicit
- GPT configuration is visible and reproducible

## Testing Strategy

The Web UI project should not weaken the current verification discipline.

### Existing tests to keep

- `verify:smoke`
- `verify:e2e`

### New tests to add when implementation starts

- setup wizard load / save behavior
- workspace allowlist CRUD
- repoId mapping validation
- mode switching
- GPT helper generation
- dashboard rendering from real control-plane state
- provider adapter mock verification

### Provider adapter testing rule

Do not require real public tunnels in CI or default local verification.

Provider adapters should be testable with:

- fixtures
- mocks
- dry-run command validation

The stable contract is the adapter interface, not an always-live external tunnel.

## Recommended Implementation Order

1. Define durable config model
2. Add workspace allowlist / repoId management endpoints
3. Add minimal Web UI shell
4. Add setup wizard
5. Add dashboard + jobs views
6. Add GPT / Actions helper view
7. Add provider adapter abstraction
8. Add preview and stable public mode helpers
9. Remove transitional GPT instruction script and doc

## Recommended Product Wording

Use these wording principles consistently:

- say `local-first` instead of `self-hosted` when the feature is still local-only
- say `under validation` instead of implying production closure
- say `stable public mode` instead of surfacing provider jargon too early
- say `workspace allowlist` instead of vague “trusted repos”

## Summary

TokenPilot should become:

- a local-first control plane
- with a human-facing Web UI
- and a machine-facing API
- using third-party tunnel providers behind a simplified mode model

The right short-term move is not to build every piece of infrastructure ourselves.
The right move is to simplify the user experience while keeping the system architecture honest.

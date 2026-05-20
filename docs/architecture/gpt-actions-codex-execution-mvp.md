# GPT Actions to Codex Execution MVP

This route prioritizes the shortest useful ChatGPT + local Codex workflow while keeping a small set of hard safety gates.

## First Principle

TokenPilot should not become a large security platform before the core workflow works. The MVP is:

```text
ChatGPT / GPT Actions
  -> TokenPilot job API
  -> local allowlisted repo
  -> optional git worktree
  -> codex exec
  -> codex review
  -> diff / review / commit artifacts
  -> ChatGPT result review
```

## Read / Write Split

- Read-side APIs stay narrow: health, GPT config, recent commits, allowlisted file reads, job status, job artifacts.
- Write-side APIs do not expose raw shell. They create explicit jobs that the local runner consumes.
- Codex CLI owns code changes, verification, and review inside the selected local repo.
- TokenPilot owns queueing, process control, artifact capture, and public-safe status.

## MVP Repo Governance

The default local config may auto-discover these repo ids when they exist beside the TokenPilot checkout:

- `tokenpilot`
- `sourceflow-refactor`
- `ai-wuaishare-cn`

Public docs must not commit machine-specific absolute paths. Runtime config may store local absolute paths in the operator's private TokenPilot config file.

## Worktree Policy

Worktrees are optional, not mandatory.

- `always`: create an isolated worktree and branch for the job.
- `never`: run in the mapped repo checkout.
- `auto`: use the job's execution mode to decide. Development tasks should prefer worktrees; read-only review/planning tasks can run in the current checkout.

The GPT should recommend a policy, but the operator remains the final decision maker.

## Codex Run Contract

`createCodexRun` is the first write-side job. It accepts:

- `repoId`
- `title`
- `instructions`
- `executionMode`: `plan`, `review`, or `develop`
- `worktreePolicy`: `auto`, `always`, or `never`
- `approvalPolicy`: Codex CLI approval policy, default `never`
- `sandbox`: Codex CLI sandbox, default `workspace-write`
- `verificationCommands`
- `commitPolicy`: `none`, `propose`, or `commit`
- optional commit title/body

The runner captures:

- execution prompt
- Codex JSONL/stdout/stderr
- git status
- diff patch
- Codex review output
- execution summary
- optional commit result

## Hard Safety Gates

Keep this list short and enforceable:

1. Bearer auth for exposed write-side APIs.
2. `repoId` must resolve through the local allowlist.
3. No raw shell HTTP endpoint.
4. Non-read jobs run through the local runner, not synchronously in the public request.
5. Worktree and commit behavior are explicit job fields and are recorded in artifacts.

## Task Control

The Web UI and GPT Actions should be able to:

- pause a running Codex process
- resume it
- terminate one job
- terminate all tracked Codex job processes

This is process control, not a general shell surface.

## Carry Forward From The Larger Goal

Keep these useful constraints from the previous A-E goal document:

- Do not overstate completion.
- Do not leak tokens, env contents, local absolute paths, or runtime-private files.
- Do not treat skipped verification as passed.
- Keep public-safe artifacts readable by GPT.
- Record remaining gaps clearly when a blocker prevents full closure.

The broader Beta hardening items still matter, but they should not block the first real GPT Actions + local Codex execution loop.

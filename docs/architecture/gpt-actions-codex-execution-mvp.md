# GPT Actions Dual-Mode Architecture

TokenPilot now supports two execution modes: ChatGPT direct-drive for simple edits, and Codex async jobs for complex tasks.

## Dual-Mode Overview

```text
                    ChatGPT / GPT Actions
                    /                    \
           Direct-Drive                  Codex Async
           (简单修改)                     (复杂任务)
           /        \                   /          \
   writeFile/editFile  runShell   createCodexRun   job artifacts
   searchCode           git ops   codex exec       codex review
   listDirectory        git cmt   worktree         diff/commit
           \        /                   \          /
            TokenPilot API  ←──→  TokenPilot Runner
                    \                    /
                 local allowlisted repo
```

## Mode Selection

| 场景 | 模式 | 延迟 | 隔离 |
|---|---|---|---|
| 改一行文案、修 typo | 直驱 editFile | 实时 1-3s | 无 |
| 单文件小改动 + verify | 直驱 editFile + runShell | 5-15s | 无 |
| 多文件编辑 + 验证 | 直驱（可自己编排） | 15-60s | 无 |
| 跨文件重构、深度探索 | Codex createCodexRun | 分钟级 | 可选 worktree |
| 需要自动审查 | Codex createCodexRun | 分钟级 | 可选 worktree |

## ChatGPT Direct-Drive Mode

ChatGPT calls TokenPilot APIs directly without going through Codex CLI:

- `writeFile` — create/overwrite text files (512 KB max)
- `editFile` — precise search-and-replace (search must be unique)
- `listDirectory` — list directory contents
- `searchCode` — ripgrep code search (40 results max)
- `runShell` — whitelisted command execution (25s timeout, 64 KB output cap)
- `getGitDiff` / `getGitStatus` — view changes
- `gitCommit` — stage and commit

Safety: all paths validate through repo allowlist + blocked segments. `runShell` uses a command whitelist, not raw shell, but it is still a high-trust local command execution API and should only be exposed behind bearer auth in an operator-controlled environment.

## Codex Async Mode (createCodexRun)

```text
ChatGPT / GPT Actions
  -> TokenPilot job API (createCodexRun)
  -> local allowlisted repo
  -> optional git worktree
  -> codex exec (Codex uses its own model_provider)
  -> codex review
  -> diff / review / commit artifacts
  -> ChatGPT result review via getJob + artifacts
```

## Read / Write Split

- Read-side APIs: health, GPT config, recent commits, file reads, directory listing, code search, job status, job artifacts.
- Write-side APIs — direct-drive: `writeFile`, `editFile`, `runShell` (whitelisted, 25s cap), `gitCommit` (public-safe paths only).
- Write-side APIs — async: `createCodexRun` (delegates to Codex CLI).
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
- public-safe diff patch
- Codex review output
- execution summary
- optional commit result

## Hard Safety Gates

Keep this list short and enforceable:

1. Bearer auth for exposed write-side APIs.
2. `repoId` must resolve through the local allowlist.
3. No raw shell HTTP endpoint.
4. Non-read jobs run through the local runner, not synchronously in the public request.
5. Git diff / commit / Codex artifacts must omit public-unsafe paths such as env files, TokenPilot runtime state, logs, and local agent state.
6. Worktree and commit behavior are explicit job fields and are recorded in artifacts.

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

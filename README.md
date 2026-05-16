# TokenPilot

![TokenPilot 项目海报](./docs/assets/tokenpilot-hero.webp)

> 用 ChatGPT 做需求澄清、上下文压缩、任务规划与结果复盘；用 Codex 进入仓库执行修改、运行验证、产出 Diff / PR。  
> 目标：减少 AI 编程中的重复读取、盲目读取和无效上下文消耗，把 Token 花在真正产生价值的地方。

[💬 参与讨论](https://github.com/wuaishare/TokenPilot/discussions) · [🐛 提交问题](https://github.com/wuaishare/TokenPilot/issues)

---

## TokenPilot 是什么？

**TokenPilot** 是一个开源方法论与工具化实验项目，目标是探索一套更省 Token、更稳定、更适合长期项目开发的 **ChatGPT + Codex 协同工作流**。

它不试图让 ChatGPT 替代 Codex，也不试图让 Codex 从零承担所有规划、审查和上下文整理工作，而是把 AI 编程过程拆成两端：

```text
ChatGPT：规划者 / 审查者 / 上下文压缩器
Codex：执行者 / 编码者 / 验证者
```

更具体地说：

```text
ChatGPT 负责：
- 理解需求
- 分析问题
- 压缩上下文
- 生成 Codex 任务包
- 审查 Codex 执行结果
- 沉淀项目状态与经验

Codex 负责：
- 进入真实仓库
- 阅读关键代码
- 修改文件
- 运行测试 / lint / build
- 输出修改摘要、验证结果和风险
```

TokenPilot 的核心目标是：

> 让 AI 少读重复上下文，多做有效开发。

---

## 为什么需要 TokenPilot？

在很多 AI 编程工作流中，真正昂贵的部分往往不是“模型写了多少代码”，而是模型为了写对代码，反复消耗大量 Token 去读取：

- 仓库文件；
- 历史对话；
- 项目文档；
- 报错日志；
- 依赖关系；
- 测试输出；
- 需求背景；
- API 契约；
- 组件结构；
- 状态机逻辑。

这会导致一种典型问题：

```text
Token 花了很多，但真正用于修改代码和验证结果的比例并不高。
```

TokenPilot 借鉴一种类似工程性能优化的思路：

```text
网站慢，不一定是写入慢，很多时候是读取太多、重复读取、没有缓存、没有索引。
AI 编程贵，也不一定是生成代码贵，很多时候是上下文读取太多、重复扫描、任务边界不清。
```

所以，TokenPilot 把 AI 开发过程视为一种 **读写分离** 工作流：

```text
读：理解项目、读取上下文、分析问题、确定范围
写：修改代码、写测试、运行验证、产出结果
```

优化重点不是“让模型少思考”，而是：

```text
少读无关内容；
少重复读同一批材料；
少让 Codex 每次从零理解项目；
多使用项目索引、任务包和执行摘要复用上下文。
```

---

## 解决什么问题？

### 1. Codex 上下文消耗过快

很多任务一开始就让 Codex 全仓探索：

```text
请全面审查这个项目，找出所有问题并修复。
```

这种输入很容易让 Codex 把大量 Token 花在搜索、理解、推断、确认上，而不是直接解决一个清晰问题。

TokenPilot 的做法是先让 ChatGPT 生成明确任务包：

```text
目标是什么？
范围在哪里？
优先读取哪些文件？
不要改哪些模块？
怎么验证？
什么结果才算完成？
```

然后再交给 Codex 执行。

### 2. ChatGPT 端能力无法充分进入 Codex 工作流

一些用户拥有 ChatGPT Free / Plus / Pro，但 Codex 工作流里不一定能直接使用 ChatGPT 端的全部模型、项目空间、文件分析、Web 调研、GitHub 检索或深度推理能力。

TokenPilot 的思路不是让 Codex “绕过限制” 使用某个模型，而是把 ChatGPT 端作为 **Planner / Reviewer**：

```text
ChatGPT 端：
- 做任务规划；
- 做上下文压缩；
- 使用项目空间保存长期上下文；
- 使用文件分析、Web 调研、GitHub 检索辅助理解；
- 生成更清晰的 Codex Task Pack；
- 审查 Codex 输出是否真正达标。

Codex 端：
- 负责进入仓库执行；
- 负责真实改代码；
- 负责跑验证；
- 负责交付可检查结果。
```

这样，用户可以把 ChatGPT 端能力纳入 Codex 工作流，而不是让 Codex 独自承担所有上下文读取和任务规划成本。

### 3. 长任务容易变成“边做边猜”

当任务描述不清晰时，AI 编程工具容易出现：

- 读了很多无关文件；
- 改了不该改的模块；
- 生成了看似合理但不符合项目真实约束的方案；
- 没有运行必要验证；
- 最终总结看起来完整，但实际问题没有解决。

TokenPilot 要求在进入 Codex 前先生成任务契约：

```text
任务目标
背景摘要
允许读取范围
禁止修改范围
推荐检索关键词
执行步骤
验证命令
验收标准
风险点
完成后需要回填的信息
```

这样可以显著减少“边做边猜”。

---

## 核心理念

### 1. 读写分离

一次 AI 编程任务可以拆成两类 Token：

```text
读 Token：
用于理解仓库结构、源码文件、文档、报错、日志、测试结果、历史决策和需求背景。

写 Token：
用于生成代码、测试、文档、迁移脚本、提交说明和复盘摘要。
```

TokenPilot 的判断是：

> 多数复杂 AI 编程任务中，读 Token 往往比写 Token 更值得优化。

### 2. 先压缩，再执行

不要把原始、模糊、开放式需求直接丢给 Codex。

推荐流程：

```text
原始需求
  ↓
ChatGPT 理解与追问
  ↓
ChatGPT 整理背景
  ↓
ChatGPT 定义范围与验收标准
  ↓
生成 Codex Task Pack
  ↓
Codex 执行
```

### 3. 任务包优先于提示词

TokenPilot 不追求一个“万能 Prompt”，而是强调稳定的任务包格式。

一个好的任务包应该让 Codex 明确知道：

```text
我要解决什么？
应该先看哪里？
哪些地方不要碰？
怎么判断完成？
验证命令是什么？
完成后要汇报哪些证据？
```

### 4. 结果要反哺为项目记忆

每次 Codex 执行完成后，都应该沉淀成可复用记录：

```text
任务名称
修改文件
根因
关键改动
验证命令
验证结果
遗留风险
下一步建议
```

这些记录可以减少下一次任务的重复读取。

---

## 推荐工作流

```text
用户提出需求
  ↓
ChatGPT：澄清目标、分析背景、压缩上下文
  ↓
ChatGPT / GitHub App：定向查证相关代码与文档
  ↓
ChatGPT：生成 Codex Task Pack
  ↓
Codex：按任务包读取关键文件、修改代码、运行验证
  ↓
Codex：输出修改摘要、验证结果、风险说明
  ↓
ChatGPT：二次审查 Codex 输出
  ↓
写入项目状态、决策日志、Token 优化记录
```

---

## ChatGPT 与 Codex 的分工

| 任务 | ChatGPT 更适合 | Codex 更适合 |
|---|---:|---:|
| 需求澄清 | ✅ | △ |
| 技术方案对比 | ✅ | △ |
| 项目上下文压缩 | ✅ | △ |
| Web / 文档调研 | ✅ | △ |
| GitHub 片段检索与解释 | ✅ | △ |
| 生成任务包 | ✅ | △ |
| 进入仓库改代码 | ❌ | ✅ |
| 运行测试 / lint / build | ❌ | ✅ |
| 输出 diff / PR | ❌ | ✅ |
| 审查执行结果 | ✅ | △ |
| 维护决策日志 | ✅ | △ |

---

## 快速开始

### 第 1 步：建立 ChatGPT 项目空间

建议为每个重要代码项目建立一个 ChatGPT Project，并放入少量高密度上下文文件，而不是直接上传整个仓库。

推荐文件：

```text
00_START_HERE.md
01_PROJECT_BRIEF.md
02_ARCHITECTURE_MAP.md
03_REPO_INDEX.md
04_API_CONTRACTS.md
05_CURRENT_STATUS.md
06_DECISION_LOG.md
07_CODEX_TASK_TEMPLATE.md
08_CODEX_RESULT_REVIEW_TEMPLATE.md
09_TOKEN_OPTIMIZATION_LOG.md
```

这些文件的作用是让 ChatGPT 快速理解项目，而不是每次从零猜测项目结构。

### 第 2 步：维护 Repo Index

`03_REPO_INDEX.md` 是给 AI 看的仓库地图。

示例：

```md
# Repo Index

## Frontend
- apps/web/src/pages：页面入口
- apps/web/src/components：通用组件
- apps/web/src/hooks：React hooks
- apps/web/src/services：API 调用封装

## Backend
- apps/api/src/routes：API 路由
- apps/api/src/services：业务服务
- apps/api/src/db：数据库访问层

## High-risk Areas
- 状态机
- 权限判断
- 数据迁移
- 批量操作
- 发布流程
```

Repo Index 的目标不是替代源码，而是减少 AI 的盲目搜索成本。

### 第 3 步：让 ChatGPT 生成 Codex Task Pack

可以使用这个提示词：

```text
请把下面的问题整理成 Codex Task Pack。

要求：
1. 任务目标必须明确；
2. 背景摘要不要超过 600 字；
3. 指定优先读取的文件或目录；
4. 明确禁止修改的范围；
5. 给出推荐检索关键词；
6. 给出执行步骤；
7. 给出必须运行的验证命令；
8. 给出验收标准；
9. 最终输出必须可以直接复制给 Codex 使用。

问题：
<在这里粘贴你的需求、报错、截图描述或现象>
```

### 第 4 步：把 Task Pack 交给 Codex 执行

给 Codex 的提示：

```text
请严格按下面的 Codex Task Pack 执行。

要求：
1. 先读取任务包指定的相关文件；
2. 如果需要读取额外文件，先说明原因；
3. 不要修改任务包禁止修改的范围；
4. 优先做最小可验证改动；
5. 修改后运行指定验证命令；
6. 输出修改文件列表、验证结果和遗留风险。

<粘贴 Codex Task Pack>
```

### 第 5 步：让 ChatGPT 审查 Codex 结果

Codex 完成后，把输出贴回 ChatGPT：

```text
这是 Codex 的执行结果。请做二次审查：

1. 是否真正解决原始问题；
2. 是否超出任务边界；
3. 是否存在无关 diff；
4. 是否存在潜在回归；
5. 验证命令是否足够；
6. 是否应该继续补一轮 Codex 任务；
7. 请生成 Codex Run Summary；
8. 请生成应该写入 DECISION_LOG 和 CURRENT_STATUS 的内容。

Codex 输出：
<粘贴 Codex 总结、diff 摘要、验证输出>
```

---

## Codex Task Pack 精简模板

> 完整模板后续会放入 `templates/codex-task-pack.md`。README 只保留最小可用版本。

```md
# Codex Task Pack

## 1. 任务目标

用一句话说明这次要完成什么。

## 2. 背景摘要

只保留当前任务必要背景，避免把整个项目历史塞进去。

## 3. 任务范围

### 必须检查

- path/to/file-a
- path/to/directory-b

### 必要时可以检查

- path/to/related-module

### 禁止修改

- path/to/unrelated-module
- package manager config
- database schema
- global theme tokens

## 4. 执行要求

1. 先确认真实根因。
2. 采用最小可验证改动。
3. 不引入新依赖，除非必须。
4. 不修改无关模块。
5. 保持现有代码风格。

## 5. 验证命令

```bash
npm run lint
npm run build
npm run test
```

请根据项目实际情况替换验证命令。

## 6. 验收标准

- 问题现象消失；
- 验证命令通过；
- 没有无关 diff；
- 不破坏现有功能；
- 输出修改文件列表与验证结果。

## 7. 完成后必须输出

- 根因；
- 修改文件；
- 关键改动；
- 验证结果；
- 遗留风险；
- 后续建议。
```

---

## Token Optimization Log 精简模板

> 完整模板后续会放入 `templates/token-optimization-log.md`。

```md
# Token Optimization Log

## YYYY-MM-DD / Task Name

- 执行端：ChatGPT / Codex
- 任务类型：Bugfix / Refactor / UI / Docs / Research / Review
- 总 Token：
- 输入 Token：
- 输出 Token：
- 读取占比判断：
- 主要读取来源：
  - 仓库文件
  - 日志
  - 文档
  - 历史对话
  - 测试输出
- 是否存在重复读取：
- 可沉淀为缓存的内容：
- 下次优化动作：
```

---

## 可选增强

### Context Pack

当项目变大后，可以用 Repomix、Gitingest 等工具，把仓库按主题打包成 AI 友好的上下文包。

推荐切片：

```text
context-pack-frontend.md
context-pack-backend.md
context-pack-api-contracts.md
context-pack-state-machine.md
context-pack-ui-components.md
context-pack-tests.md
context-pack-docs.md
```

每个 Context Pack 都应该记录生成时间、分支、commit、包含范围、排除范围和适用场景。

### MCP App

后续可以把 TokenPilot 做成一个 ChatGPT Developer Mode / MCP App，用结构化工具帮助 ChatGPT 读取项目状态和生成任务包。

MVP 阶段建议只做只读能力：

```text
get_project_manifest
get_repo_index
get_current_status
get_recent_decisions
get_known_issues
get_recent_codex_runs
generate_codex_task_pack
summarize_codex_result
```

写操作必须非常谨慎，建议遵守：

```text
dry-run → preview → human confirmation → write → audit log
```

---

## 适合什么人？

TokenPilot 适合所有正在使用 ChatGPT 与 Codex 进行 AI 编程的人。

无论是 Free、Plus 还是 Pro 用户，都可以使用 TokenPilot 的基本思路：先用 ChatGPT 做需求澄清、上下文压缩和任务包整理，再把明确任务交给 Codex 执行。

区别主要在于：

- Free 用户更适合轻量任务规划、短上下文整理和基础任务包生成；
- Plus 用户适合更频繁的项目分析、任务拆解和结果复盘；
- Pro 用户更适合把 ChatGPT 端作为高强度 Planner / Reviewer，用更强模型能力辅助 Codex 工作流。

TokenPilot 尤其适合：

- 经常觉得 Codex 上下文消耗太快的人；
- 维护中大型仓库、多模块项目、长期任务的人；
- 想实践 “Planner + Coder + Reviewer” 工作流的人；
- 想减少 Codex 盲读、重复读、全仓读和返工的人；
- 想把 ChatGPT 端模型、项目空间、文件分析、Web 调研等能力纳入 Codex 工作流的人。

不太适合：

- 很小的一次性脚本；
- 不需要上下文沉淀的临时任务；
- 只想找一个自动写完整项目的黑盒工具；
- 不愿意维护项目索引、状态日志、任务包的人。

---

## 项目路线图

### Phase 0：方法论文档

- [x] 定义 TokenPilot 工作流
- [x] 提供 Codex Task Pack 精简模板
- [x] 提供 Token Optimization Log 精简模板

### Phase 1：模板库

- [ ] `templates/codex-task-pack.md`
- [ ] `templates/codex-result-review.md`
- [ ] `templates/repo-index.md`
- [ ] `templates/current-status.md`
- [ ] `templates/decision-log.md`
- [ ] `templates/token-optimization-log.md`

### Phase 2：示例库

- [ ] `examples/simple-bugfix-task-pack.md`
- [ ] `examples/ui-review-task-pack.md`
- [ ] `examples/refactor-task-pack.md`
- [ ] `examples/codex-run-summary.md`

### Phase 3：CLI 工具

计划提供一个轻量 CLI：

```bash
tokenpilot init
tokenpilot pack
tokenpilot task
tokenpilot review
tokenpilot log
```

可能能力：

- 初始化项目文档；
- 生成 Repo Index 草稿；
- 从 Codex 输出生成 Run Summary；
- 从 issue / markdown / error log 生成 Task Pack；
- 维护 Token Optimization Log。

### Phase 4：MCP App

提供 ChatGPT 可调用的 MCP 工具：

- 读取项目状态；
- 检索项目文档；
- 生成 Codex Task Pack；
- 复盘 Codex 结果；
- 追加决策日志。

---

## 建议仓库结构

```text
TokenPilot/
├── README.md
├── LICENSE
├── docs/
│   ├── philosophy.md
│   ├── workflow.md
│   ├── chatgpt-codex-collaboration.md
│   ├── token-read-write-split.md
│   └── mcp-app-design.md
├── templates/
│   ├── codex-task-pack.md
│   ├── codex-result-review.md
│   ├── repo-index.md
│   ├── current-status.md
│   ├── decision-log.md
│   └── token-optimization-log.md
├── examples/
│   ├── simple-bugfix-task-pack.md
│   ├── ui-review-task-pack.md
│   └── refactor-task-pack.md
└── packages/
    └── cli/
```

---

## 参与讨论

TokenPilot 仍然是一个实验性开源项目。如果你正在探索 ChatGPT + Codex 协同、节省 Token 的 AI 编程工作流，欢迎参与讨论：

- 💬 GitHub Discussions：<https://github.com/wuaishare/TokenPilot/discussions>
- 🐛 GitHub Issues：<https://github.com/wuaishare/TokenPilot/issues>
- 🔀 Pull Requests：欢迎贡献模板、文档、示例和工具代码。

建议使用方式：

```text
Discussions：开放讨论、经验分享、Q&A、工作流案例
Issues：明确 bug、文档错误、可执行任务
Pull Requests：模板、文档、示例、工具代码贡献
```

---

## 免责声明

TokenPilot 是一个社区实验项目 / 开源方法论，不隶属于 OpenAI，也不是 Codex、ChatGPT 或 GitHub 的官方功能。

本项目不会帮助绕过任何平台限制。它关注的是：

```text
更合理地分配现有工具能力；
更少重复读取；
更清晰地描述任务；
更稳定地交付代码；
更系统地复盘结果。
```

---

## 参考资料

- OpenAI Codex Web  
  https://developers.openai.com/codex/cloud

- OpenAI Codex Models  
  https://developers.openai.com/codex/models

- Connecting GitHub to ChatGPT  
  https://help.openai.com/en/articles/11145903-connecting-github-to-chatgpt

- ChatGPT Developer Mode  
  https://developers.openai.com/api/docs/guides/developer-mode

- Using Codex with your ChatGPT plan  
  https://help.openai.com/en/articles/11369540-using-codex-with-your-chatgpt-plan

- 相关讨论：有人实践过“规划模型 + 编码模型”的 AI 编程工作流吗？  
  https://linux.do/t/topic/2185954

- Repomix  
  https://github.com/yamadashy/repomix

- Gitingest  
  https://gitingest.com/

---

## License

待定。建议使用 MIT License，方便社区采用和二次扩展。

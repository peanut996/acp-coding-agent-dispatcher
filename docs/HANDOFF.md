# Agent Router Handoff

Last updated: 2026-06-20, Asia/Shanghai

## Snapshot

- Product name: Agent Router
- Plugin skill name: Coding Agent Dispatch
- Repo: `https://github.com/peanut996/codex-agent-router`
- Local repo: `/Users/peanut996/Workspace/codex-agent-router`
- Default branch: `master`
- Current HEAD: `33e67f1 Update marketplace ref to v0.6.8`
- Current release: `v0.6.8`
- Release URL: `https://github.com/peanut996/codex-agent-router/releases/tag/v0.6.8`
- Local installed plugin: `agent-router@codex-agent-router`, installed and enabled, version `0.6.8`
- Installed cache: `/Users/peanut996/.codex/plugins/cache/codex-agent-router/agent-router/0.6.8`

## PRD And Source Inventory

The original product PRD was written in the conversation as `ACP Coding Agent Dispatcher 产品需求文档`, draft PRD v0.1. It defines the V1 product boundary: a Codex Plugin with a bundled MCP server, local dispatcher registry, ACP/CLI adapters, session/job management, safety controls, and no native Codex sidebar/settings UI changes in V1.

Current checked-in product docs:

- `README.md`: public product overview, current capability status, validation commands, and adapter acceptance matrix.
- `docs/USAGE.md`: English usage guide.
- `docs/USAGE.zh-CN.md`: Chinese usage guide.
- `skills/agent-router/SKILL.md`: bundled Codex skill instructions for when and how to use Agent Router MCP tools.
- `.codex-plugin/plugin.json`: plugin manifest and marketplace-facing metadata.
- `.agents/plugins/marketplace.json`: repo marketplace entry, currently pinned to `v0.6.8`.

There is not yet a checked-in standalone PRD file. If the product needs a canonical repo artifact, add `docs/PRD.md` by converting the original conversation PRD and then keep this handoff document as the operational status companion.

## Product Boundary

V1 remains a Codex Plugin plus bundled local MCP server.

In scope:

- Discover installed local coding agents.
- Read ACP Registry metadata for known adapters.
- Configure defaults and safety flags.
- Run external agents against an existing absolute worktree.
- Track jobs, sessions, event logs, process PID metadata, changed files, and failure reasons.
- Continue, list, cancel, archive, and tail jobs/sessions through MCP tools.
- Prefer ACP adapters when available and fall back to CLI adapters.

Out of scope for V1:

- Native Codex sidebar/session-list UI.
- Native Codex settings-page agent picker.
- Message avatar/logo customization.
- Automatic external agent installation.
- Automatic commit, push, PR, or conflict resolution.
- Cloud or multi-tenant scheduling.

## Implemented MCP Tools

- `discover_coding_agents`
- `get_coding_agent_dispatcher_config`
- `configure_coding_agent_dispatcher`
- `run_coding_agent`
- `list_coding_agent_jobs`
- `get_coding_agent_job`
- `tail_coding_agent_job_events`
- `cancel_coding_agent_job`
- `list_coding_agent_sessions`
- `continue_coding_agent_session`
- `archive_coding_agent_session`

## Current Capability Progress

| Area | Status | Notes |
| --- | --- | --- |
| Plugin packaging | Done | Manifest, skill, bundled MCP server, repo marketplace, GitHub releases are working. |
| Agent discovery | Done | Finds `opencode`, Cursor Agent `agent`, `claude`, `codex`, plus installed ACP adapter commands. |
| ACP Registry metadata | Done | Reads registry metadata, caches it locally, exposes ids/icons/versions/install hints. No auto-install. |
| Default config | Done | `launchExternalAgents=true` and `inheritEnvironment=true` by default; per-call overrides are available. |
| Run jobs | Done | Sync and async runs are supported. Worktree must be existing absolute path. |
| Job tracking | Done | Registry records job/session state, process PID metadata, status, changed files, logs, and errors. |
| Event tailing | Done | `tail_coding_agent_job_events` supports polling with `nextEventIndex`. |
| Cancellation | Done | Active child processes can be cancelled; persisted process metadata records kill attempts. |
| Restart recovery | Done | Orphaned running jobs are marked and recorded child PIDs are best-effort terminated. |
| Session list/continue/archive | Mostly done | Dispatcher sessions work. OpenCode native ACP session list/continue is validated. Multi-agent native ACP aggregation exists; more real providers need acceptance. |
| Result/failure surfacing | Done | Provider errors are returned in `failureReason` and `agentErrors`; OpenCode model config options are surfaced in `availableModels`. |
| Safety | Mostly done | Worktree absolute-path requirement, per-worktree lock, permission profiles, no auto commit/push. More policy polish can be added later. |
| Native Codex UI integration | Not planned for V1 | Use MCP tool responses instead. |

## Adapter Status

| Agent | Preferred adapter | Fallback | Current validation |
| --- | --- | --- | --- |
| OpenCode | `opencode acp` | None needed | Real ACP E2E passed; real no-model handshake and session-list smoke passed. |
| Claude Code | `claude-agent-acp` | `claude -p --output-format stream-json` | CLI real E2E passed; ACP no-model handshake smoke added in `v0.6.8`. |
| Cursor Agent | None yet | Official `agent` CLI | Real CLI E2E passed. |
| Codex CLI | `codex-acp` | `codex exec` | CLI real E2E passed; ACP no-model handshake smoke added in `v0.6.8`. |

## Release Timeline

- `v0.6.6`: added `tail_coding_agent_job_events` for near-real-time polling.
- `v0.6.7`: added ACP Registry metadata/cache, ACP-first routing for OpenCode/Claude/Codex, generalized ACP stdio execution and native session-list aggregation.
- `v0.6.8`: added real no-model ACP adapter handshake smoke for `claude-agent-acp` and `codex-acp`, registry version fallback for ACP adapters that do not support version probing, and PATH-priority fixes for equal/unknown versions.

## Latest Validation Evidence

No-model validation listed in `README.md`:

```bash
npm run check
npm run smoke
npm run smoke:sessions
npm run smoke:opencode
npm run smoke:opencode:sessions
npm run smoke:acp:handshake
npm run e2e:restart-recovery
```

`v0.6.8` release notes state the following validation passed:

- `npm run check`
- `npm run smoke`
- `npm run smoke:newline`
- `npm run smoke:sessions`
- `npm run smoke:opencode`
- `npm run smoke:acp:handshake`
- `git diff --check`
- Real discovery confirmed Codex ACP `acp.version=0.16.0` and Claude ACP `acp.version=0.48.0` without misleading ACP version probe failure notes.

Known real E2E acceptance from the docs:

- OpenCode ACP file-edit E2E passed with model `opencode-go/glm-5.2`.
- OpenCode session lifecycle E2E passed.
- Claude CLI fallback E2E passed.
- Cursor Agent CLI fallback E2E passed through official `agent`.
- Codex CLI fallback E2E passed.

## Important Runtime Paths

- Config: `~/.codex/agent-router/config.json`
- Job/session registry: `~/.codex/agent-router/registry.json`
- Logs: `~/.codex/agent-router/logs/`
- ACP Registry cache: `~/.codex/agent-router/acp-registry-cache.json`
- Installed plugin cache: `~/.codex/plugins/cache/codex-agent-router/agent-router/0.6.8`

## Current Defaults

- `launchExternalAgents=true`
- `inheritEnvironment=true`
- `allowCurrentDirectory=false`
- `requireAbsoluteWorktree=true`
- Registry enabled by default.
- Registry URL: `https://cdn.agentclientprotocol.com/registry/v1/latest/registry.json`
- Registry cache TTL: 86400 seconds.

## Known Gaps

1. Real ACP prompt/file-edit E2E for `claude-agent-acp` and `codex-acp` is not yet documented as passed. `v0.6.8` validates no-model handshake/session creation only.
2. Native ACP session-list behavior is proven for OpenCode. Other ACP adapters need real session-list/continue acceptance.
3. Registry-driven adapter expansion is still manual. Only known local routes are mapped into Agent Router profiles today.
4. No automatic adapter installation. Registry install hints are surfaced, but users install tools themselves.
5. No standalone checked-in PRD. The original PRD exists in conversation context; convert it to `docs/PRD.md` if needed.
6. No native Codex UI extension. This is intentionally out of V1 scope unless official UI extension APIs become available.

## Recommended Next Work

1. In a fresh Codex top-level thread, confirm Agent Router tools are injected:

   ```text
   使用 Agent Router / Coding Agent Dispatch。先调用 discover_coding_agents。
   不要 fallback 到 Cursor CLI；如果看不到 run_coding_agent，直接报告工具未注入。
   ```

2. Run `discover_coding_agents` and check:

   - OpenCode is available through ACP.
   - Claude has registry id `claude-acp` and uses `claude-agent-acp` if installed.
   - Codex has registry id `codex-acp` and uses `codex-acp` if installed.
   - Cursor Agent uses official `agent` CLI fallback.

3. Run a small dispatch smoke through Agent Router with an isolated worktree. Use `tail_coding_agent_job_events` for async polling.

4. Add or run real ACP E2E for Claude and Codex adapters:

   - no prompt/model call smoke is already covered by `npm run smoke:acp:handshake`;
   - next acceptance should prove prompt/file edit, provider session id, changed file, log path, and failure surfacing.

5. Convert the original PRD into a checked-in `docs/PRD.md`, then keep `docs/HANDOFF.md` as the latest progress ledger.

6. Start registry expansion work:

   - add a small registry-id-to-router-profile mapping layer;
   - keep no-auto-install policy;
   - prefer ACP transports when executable is installed;
   - retain CLI fallback only for known stable local commands.

## Fresh Session Prompt

Copy this into a new Codex session:

```text
继续 Agent Router / codex-agent-router 开发。

Repo: /Users/peanut996/Workspace/codex-agent-router
GitHub: https://github.com/peanut996/codex-agent-router
Current release: v0.6.8
Current local plugin: agent-router@codex-agent-router 0.6.8

先做这些：
1. git -C /Users/peanut996/Workspace/codex-agent-router fetch origin
2. 确认 master 与 origin/master 状态。
3. 使用 Agent Router / Coding Agent Dispatch，先调用 discover_coding_agents。
4. 不要 fallback 到 Cursor CLI；如果看不到 discover_coding_agents 或 run_coding_agent，直接报告工具未注入。

当前产品状态：
- Plugin/MCP/job/session/config/log/event tailing 已可用。
- launchExternalAgents 默认 true。
- inheritEnvironment 默认 true。
- OpenCode 走 native ACP。
- Claude 优先 claude-agent-acp，fallback claude CLI。
- Codex 优先 codex-acp，fallback codex exec。
- Cursor Agent 走官方 agent CLI fallback。
- ACP Registry metadata/cache 已接入。
- v0.6.8 已加入 claude-agent-acp / codex-acp 的真实 no-model handshake smoke。

下一步优先：
- 验证新 thread 中 Agent Router tools 是否注入。
- 用 discover_coding_agents 看本机 ACP adapter 状态。
- 设计/运行 Claude ACP 和 Codex ACP 的真实 prompt/file-edit E2E。
- 把原始 PRD 转成 docs/PRD.md。
```

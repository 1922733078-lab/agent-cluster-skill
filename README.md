# agent-cluster Codex Skill

`agent-cluster` is a Codex skill for coordinating a dynamic team of specialist subagents on complex tasks. It is designed for orchestrator-only workflows where the main agent delegates substantive work, tracks dependencies, spawns new subagents as needs emerge, routes iterative follow-up rounds, and synthesizes the final result.

The skill is especially useful when a request explicitly calls for an agent team, subagents, multi-agent collaboration, parallel agents, `agent集群`, `多代理`, `多智能体`, or a complex task that should be split across specialist agents.

## What This Skill Does

- Coordinates explorer, worker, reviewer, and verifier subagents.
- Makes the main agent act as an orchestrator instead of the primary worker.
- Delegates substantive exploration, implementation, drafting, review, research, and verification to subagents when possible.
- Keeps subagent prompts scoped to concrete files, modules, or questions.
- Requires compressed subagent summaries instead of long logs or full-file excerpts.
- Uses iterative follow-up rounds instead of one-shot delegation.
- Tracks task dependencies with `TASK_ID`, `DEPENDS_ON`, status, and next decision.
- Continues a branch as soon as its required dependency returns.
- Dynamically spawns new subagents when returned results reveal new independent scopes, specialist needs, or verification needs.
- Avoids waiting for unrelated running subagents.
- Keeps the main agent responsible for dispatch, dependency routing, quality triage, integration, and final delivery.

## Current Behavior

This version uses orchestrator-only, dependency-aware early-return orchestration.

Instead of deciding the final team size at the beginning, the main agent starts with the first delegable units and expands the team as the task changes. Instead of waiting for a full round of subagents to finish, the main agent reacts whenever any relevant subagent returns:

1. Inspect the returned compressed result.
2. Decide whether the branch is ready, blocked, or needs follow-up.
3. If no other result is needed, immediately send a follow-up to the same subagent.
4. If a specific dependency is needed, wait only for that dependency.
5. If the result exposes a new independent scope, specialist need, or verification need, spawn a new subagent.
6. Do not wait for unrelated subagents unless final integration depends on them.

Example:

```text
Worker A returns.
Worker A's next step does not depend on Worker B or C.
The main agent immediately continues Worker A's branch.

Worker D returns.
Worker D needs Reviewer E's result before continuing.
The main agent marks Worker D as blocked on Reviewer E and waits only for Reviewer E.

Worker F returns.
Worker F exposes a new independent compatibility risk.
The main agent spawns Verifier G with a bounded verification task instead of checking it locally.
```

## Repository Layout

```text
.
├── README.md
└── agent-cluster/
    ├── SKILL.md
    └── agents/
        └── openai.yaml
```

The `agent-cluster/` directory is the actual skill folder. The root `README.md` is documentation for sharing on GitHub and is not required inside the skill folder.

## Installation

Copy the `agent-cluster/` folder into your Codex skills directory.

Common locations:

```bash
mkdir -p "${CODEX_HOME:-$HOME/.codex}/skills"
cp -R agent-cluster "${CODEX_HOME:-$HOME/.codex}/skills/"
```

Then restart or refresh Codex so the skill can be discovered.

## Usage

Ask Codex to use the skill explicitly:

```text
Use $agent-cluster to coordinate subagents for this task.
```

Or use natural language triggers:

```text
Use an agent team to inspect this codebase and implement the fix.
```

```text
请使用 agent集群 / 多智能体 协作完成这个复杂任务。
```

## Delegation Model

The main agent should:

- Clarify objective, deliverable, workspace, constraints, and verification target.
- Create and maintain a task ledger instead of a fixed team plan.
- Spawn the first necessary subagents, then dynamically add more as the task evolves.
- Assign each subagent one owned scope.
- Track each delegated task in a lightweight ledger.
- Inspect returned summaries immediately.
- Send follow-up work only when dependencies are satisfied.
- Request revision or verification from subagents.
- Integrate final results and produce the final user-facing answer.

The main agent should not personally perform exploration, implementation, drafting, review, research, or verification when a subagent can reasonably own it.

Subagents should:

- Work only within their owned scope.
- Avoid reverting unrelated changes.
- Save long notes, logs, and evidence to workspace files.
- Return a compressed result with exact file paths, line references, commands, artifacts, risks, and next dependencies.

## Subagent Prompt Shape

The skill includes this prompt structure:

```text
TASK_ID: <stable id>
TASK_TYPE: <explore | implement | draft | review | verify | synthesize | artifact>
SPAWN_REASON: <why this subagent is needed now>
DEPENDS_ON: <task ids, agents, or "none">
EXPECTED_OUTPUT: <exact output expected>
Your owned scope: <files/modules/questions>.

RESULT: <1-3 sentence conclusion>
CHANGED: <files changed, or "none">
EVIDENCE: <exact file:line pointers, test commands, artifact paths>
RISKS: <remaining risks or "none">
NEXT_DEPENDS_ON: <specific task ids/agents needed before follow-up, or "none">
SPAWN_RECOMMENDATION: <new subagent scope/type if needed, or "none">
NEXT: <specific follow-up request if more work is needed, or "none">
```

## Validation

This skill was validated with the Codex skill creator validator:

```text
Skill is valid!
```

## Notes

- The skill does not set a hard maximum number of subagents.
- It does not decide the final team size upfront.
- It uses an elastic team that grows or shrinks with the task.
- Additional subagents should be created for genuinely independent scopes, specialist needs, or verification needs.
- Follow-up to an existing subagent is preferred when its retained context is useful.
- Every active subagent should have a clear owner scope, task ID, dependency entry, expected output, and spawn reason.

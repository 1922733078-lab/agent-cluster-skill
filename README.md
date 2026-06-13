# agent-cluster Codex Skill

`agent-cluster` is a Codex skill for coordinating a small team of specialist subagents on complex tasks. It is designed for multi-agent workflows where the main agent delegates bounded work, tracks dependencies, iterates through follow-up rounds, and integrates the final result.

The skill is especially useful when a request explicitly calls for an agent team, subagents, multi-agent collaboration, parallel agents, `agent集群`, `多代理`, `多智能体`, or a complex task that should be split across specialist agents.

## What This Skill Does

- Coordinates explorer, worker, reviewer, and verifier subagents.
- Keeps subagent prompts scoped to concrete files, modules, or questions.
- Requires compressed subagent summaries instead of long logs or full-file excerpts.
- Uses iterative follow-up rounds instead of one-shot delegation.
- Tracks task dependencies with `TASK_ID`, `DEPENDS_ON`, status, and next decision.
- Continues a branch as soon as its required dependency returns.
- Avoids waiting for unrelated running subagents.
- Keeps the main agent responsible for validation, integration, and final delivery.

## Current Behavior

This version uses dependency-aware early-return orchestration.

Instead of waiting for a full round of subagents to finish, the main agent reacts whenever any relevant subagent returns:

1. Inspect the returned compressed result.
2. Decide whether the branch is ready, blocked, or needs follow-up.
3. If no other result is needed, immediately send a follow-up to the same subagent.
4. If a specific dependency is needed, wait only for that dependency.
5. Do not wait for unrelated subagents unless final integration depends on them.

Example:

```text
Worker A returns.
Worker A's next step does not depend on Worker B or C.
The main agent immediately continues Worker A's branch.

Worker D returns.
Worker D needs Reviewer E's result before continuing.
The main agent marks Worker D as blocked on Reviewer E and waits only for Reviewer E.
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
- Create a short orchestration plan.
- Spawn the smallest useful team.
- Assign each subagent one owned scope.
- Track each delegated task in a lightweight ledger.
- Inspect returned summaries immediately.
- Send follow-up work only when dependencies are satisfied.
- Integrate final results and run verification.

Subagents should:

- Work only within their owned scope.
- Avoid reverting unrelated changes.
- Save long notes, logs, and evidence to workspace files.
- Return a compressed result with exact file paths, line references, commands, artifacts, risks, and next dependencies.

## Subagent Prompt Shape

The skill includes this prompt structure:

```text
TASK_ID: <stable id>
DEPENDS_ON: <task ids, agents, or "none">
Your owned scope: <files/modules/questions>.

RESULT: <1-3 sentence conclusion>
CHANGED: <files changed, or "none">
EVIDENCE: <exact file:line pointers, test commands, artifact paths>
RISKS: <remaining risks or "none">
NEXT_DEPENDS_ON: <specific task ids/agents needed before follow-up, or "none">
NEXT: <specific follow-up request if more work is needed, or "none">
```

## Validation

This skill was validated with the Codex skill creator validator:

```text
Skill is valid!
```

## Notes

- The skill does not set a hard maximum number of subagents.
- It recommends the smallest useful team.
- Medium tasks typically use an explorer plus the main agent.
- Large tasks typically use three to five specialist agents.
- Additional subagents should be created only for genuinely independent scopes.
- Follow-up to an existing subagent is preferred when its retained context is useful.


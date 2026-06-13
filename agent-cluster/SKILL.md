---
name: agent-cluster
description: Coordinate an agent team for complex tasks, prioritizing dependency-aware early returns, iterative follow-up rounds, and waiting only for specific required subagent results. Use when the user explicitly asks for agent集群, agent team, subagents, multi-agent work, multi-agent collaboration, 多代理, 多智能体, 并行代理, or wants a task decomposed across specialist agents and integrated into one final result.
---

# agent集群

## Operating Rule

Use an agent team only when the user explicitly requests this skill or asks for subagents, delegation, parallel agent work, agent team, 多代理, or 多智能体协作. If the current task is simple or has no meaningful parallelism, state that briefly and complete it locally.

## Context Budget Rule

When preserving the main agent's context is important, use context-sparing orchestration by default.

The main agent should not ingest full exploratory notes, long logs, full-file excerpts, or complete intermediate reasoning from subagents. Subagents must keep detailed notes in workspace artifacts and return only compressed, decision-ready summaries with precise file paths, line references, commands, and artifact locations.

Prefer:

- file paths over pasted content
- diffstats over full diffs
- exact line pointers over long excerpts
- saved logs over pasted logs
- concise risk lists over narrative analysis
- follow-up questions to the same subagent over spawning a new subagent with duplicated context

## Delegation Ownership Rule

After the main agent delegates a task to a subagent, the main agent should not perform the same task in parallel. While subagents run, the main agent should react only to returned results, explicit dependencies, user status updates, subagent coordination, tool recovery, or integration scaffolding that does not duplicate delegated work. Do not start speculative side tasks while subagents are running.

Keep subagents open after one completed round when their local context may still be useful. Prefer sending focused follow-up work to the same subagent before integrating or spawning a new subagent, unless the current result is already sufficient or the next question is genuinely independent. Close a subagent only when its context no longer has value for likely follow-up work, verification, or integration.

## Multi-Round Priority Rule

Treat subagent work as an iterative delegation loop by default:

- Expect at least one follow-up decision after every subagent result: continue with the same subagent, delegate a new independent scope, integrate, or close.
- Prefer same-subagent follow-ups when the subagent has useful local context and the next task refines, verifies, extends, or fixes its prior result.
- Run another subagent round before final integration when the returned result is incomplete, weakly evidenced, contradictory, unverified, or exposes a clear next step.
- Stop the loop only when additional rounds would duplicate work, add no meaningful evidence, or delay a deliverable that is already ready to verify and return.

## Dependency-Aware Early Return Rule

Treat subagent orchestration as an event-driven dependency loop, not a full-round barrier:

- Maintain a lightweight task ledger with `TASK_ID`, owner, scope, `DEPENDS_ON`, status, and next decision.
- When any subagent returns, immediately inspect its compressed result enough to decide the next step.
- If the next useful follow-up depends only on that subagent's result, send the follow-up to the same subagent immediately.
- If the next step depends on another specific subagent result, mark that branch as blocked on that dependency and wait only for the required dependency.
- Do not wait for all running subagents unless final integration genuinely depends on all of them.
- Do not spawn unrelated new work while waiting; only continue, unblock, verify, or integrate branches whose dependencies are satisfied.

## Workflow

1. Clarify the objective, deliverable, workspace, constraints, and verification target from the current conversation and files.
2. Make a short orchestration plan before spawning agents:
   - Keep the immediate critical-path task local.
   - Split only independent sidecar work or disjoint implementation areas.
   - Assign each agent one concrete output and, for code edits, one clear ownership boundary.
   - Record each delegated task's ID, owner, dependencies, and status so the main agent does not duplicate it.
3. Discover the available subagent tools if they are not already visible. Prefer the local multi-agent tools when available, especially `spawn_agent`, `wait_agent`, `send_input`, and `close_agent`.
4. Spawn the smallest useful team:
   - Use explorer-style agents for bounded codebase or research questions.
   - Use worker-style agents for direct implementation in disjoint files or modules.
   - Use reviewer/verifier agents only when they can check a concrete risk in a later round or independent scope.
5. Give each agent a self-contained, context-budgeted prompt with:
   - A task ID and explicit dependencies, or `DEPENDS_ON: none`.
   - The task objective and expected output format.
   - The files, modules, or question it owns.
   - Any constraints, tests, style rules, and sources it must respect.
   - A strict return budget, usually no more than 10-20 bullets.
   - An instruction to save detailed notes, long evidence, command logs, and draft artifacts to workspace files instead of pasting them back.
   - A reminder not to revert or overwrite unrelated work.
6. Wait for the next relevant subagent event, not necessarily the whole active set. While waiting, limit main-agent activity to user status updates, subagent coordination, tool recovery, or preparing integration scaffolding that does not duplicate delegated work.
7. When any subagent returns, immediately decide whether to:
   - send a focused follow-up to the same subagent when its retained context is useful and dependencies are satisfied;
   - mark the branch blocked on the specific subagent result it needs, then wait only for that dependency;
   - spawn a new subagent only for a genuinely new independent scope whose prerequisites are already satisfied;
   - keep the subagent open if its context may still help later;
   - close the subagent only when its context no longer has value;
   - integrate that branch when the result is sufficiently complete and verified.
8. Repeat steps 6-7 until all deliverable-required branches are complete or explicitly blocked, then integrate results yourself. Review patches or findings, resolve conflicts, discard weak suggestions, run appropriate verification, and produce one coherent final answer. Open only the referenced files, line ranges, logs, or artifacts needed for integration and verification.

## Team Patterns

Use a two-agent pattern for medium tasks:

- Explorer: inspect the relevant code or documents and return risks, file pointers, or a minimal plan.
- Main agent: wait for the explorer result, decide any follow-up round, then implement, verify, and integrate when the delegated result is ready.

Use a three-to-five-agent pattern for large tasks:

- Planner or explorer: map architecture, assumptions, and ownership boundaries.
- Worker A/B/C: implement disjoint slices or produce independent artifacts.
- Reviewer or verifier: run focused checks, reproduce a bug, inspect generated output, or audit the final patch.

Avoid spawning multiple agents for the same unresolved question. Reuse an existing agent with follow-up input when its context is still relevant. Prefer one well-scoped follow-up round over immediate local completion when a subagent's retained context can improve correctness, evidence, or implementation quality.

Do not treat a multi-agent team as a mandatory barrier. If Worker A returns and its next step does not depend on Worker B or C, continue Worker A's branch immediately. If Worker A depends on Worker B, wait only for Worker B, not for unrelated workers.

## Prompt Template

```text
You are part of an agent team working on: <objective>.

TASK_ID: <stable id>
DEPENDS_ON: <task ids, agents, or "none">
Your owned scope: <files/modules/questions>.
Do not edit outside this scope unless required to complete the task; if required, report it first.
Other agents or the main agent may be editing nearby files. Do not revert unrelated changes.

Context budget:
- Do not paste full files, long logs, or broad notes into your final response.
- Save detailed notes to: <artifact-path>.
- Save long command output to files when useful.
- Return only a compressed summary.

Deliverable format:
RESULT: <1-3 sentence conclusion>
CHANGED: <files changed, or "none">
EVIDENCE: <exact file:line pointers, test commands, artifact paths>
RISKS: <remaining risks or "none">
NEXT_DEPENDS_ON: <specific task ids/agents needed before follow-up, or "none">
NEXT: <specific follow-up request if more work is needed, or "none">
```

## Integration Checklist

- Confirm every delegated result maps back to the user's requested deliverable.
- Keep a lightweight task ledger and update status/dependencies after each returned result.
- Inspect code changes before relying on them.
- Do not repeat the subagent's delegated search, implementation, or analysis unless a concrete reliability issue requires it.
- Do not read every subagent artifact by default; inspect only artifacts needed to make or verify a decision.
- Prefer targeted `rg`, `git diff --stat`, `git diff -- <file>`, and line-range reads over full-file reads.
- Ask the same subagent for a compressed follow-up when its local context is still useful.
- Continue branches as soon as their dependencies are satisfied; do not wait for unrelated running subagents.
- Require subagents to point to evidence instead of replaying evidence.
- Keep useful subagents open across rounds and close them only when their retained context has no expected value.
- Run targeted tests, builds, render checks, or source verification appropriate to the task.
- Mention agent-derived findings only after validating or clearly labeling them as unverified.
- Keep the final response concise: what changed, where, and how it was verified.

## Fallback

If subagent tooling is unavailable, simulate the team as sequential specialist passes in one agent: planner, implementer, reviewer, verifier. Tell the user that live subagents were unavailable and complete the task with the local workflow.

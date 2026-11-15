# Agent Architecture Skill

A Claude skill that evaluates, evolves and runs any project end to end — a business idea, a research paper, a piece of software — using a team of AI agents with built-in quality control.

It combines two systems I built:

- **[Idea Evolution Engine](https://github.com/souzaeliasrafael/idea-evolution-engine)** — agents that turn a raw idea into an auditable decision through bounded rounds of research, building, criticism and scoring.
- **A long-running research pipeline** (a neighborhood-boundary study for a Brazilian city) that runs for days in the cloud on parallel workstreams, with a blind reviewer checking every delivery and an arbiter deciding gates without waiting for me.

The skill itself is written in Portuguese (`skills/arquitetura-agentes/SKILL.md`).

## How it works

```
Spec  ->  Structurer -> Researcher -> Builder -> Critic -> Verification -> Judge -> Reviewer -> Arbiter
                                                                                        |
                                    next round / test / pivot / approve / discard  <----+
```

| Role | What it does |
|---|---|
| **Spec** | Written first: goal, ground truth, judging criteria, gates, constraints, stop condition. Everything project-specific lives here, never in the skill. |
| **Researcher** | Picks the highest-impact open hypothesis and looks for sourced evidence. Facts, assumptions and inferences are always labeled. |
| **Builder** | Produces a new version. Previous versions are immutable. |
| **Critic** | Lists testable flaws, each with the condition that would confirm it. |
| **Judge** | Scores each criterion 0–4 on a fixed rubric (no source → max 1). The final 0–100 score is computed in code, not estimated. |
| **Reviewer** | A separate sub-agent that sees only the spec and the delivery, not the reasoning behind it, and checks it item by item. |
| **Arbiter** | Decides gates on its own and writes down the reasoning for the human to review later. Human decisions always override it. |

### Running unattended, in parallel

- **Workstreams** run in parallel, each on its own git branch, coordinated through files in a private repo (`SPEC`, `STATUS`, decisions, sources, ablation table).
- **Lock + heartbeat:** each workstream takes a lock before working and commits a heartbeat every 10 minutes; a stale lock (20 min) can be taken over.
- **Dependencies:** a workstream that needs another one's result releases its lock as "waiting for X" and resumes only when `X: approved` appears in the status file.
- **Watchdog:** a small, cheap agent runs several times an hour, frees stale locks and re-fires stalled workstreams.
- **Daily report:** progress, score changes, arbiter decisions to review, items waiting on the human, stagnation detection.

### Rules that never change

- Write down the reason for every decision.
- Never overwrite, move or delete earlier work.
- Ablation: a new feature stays only if it measurably improves the metric in the spec.
- Validate on the full dataset, not just a sample, when cost allows.
- Irreversible actions (deleting data, publishing, spending money, contacting third parties) stop that workstream and wait for the human.
- Nothing invented: every data point and reference needs a verified source.

## Modes

- **Evaluate** — runs the evaluation rounds in a single conversation.
- **Run** — creates scheduled tasks that develop and monitor the project until the stop condition.
- **Take over** — adopts a project already in progress without redoing or overwriting anything.

## Installation

**Claude app:** upload the `skills/arquitetura-agentes` folder as a skill, then say *"usa a arquitetura de agentes"*.

**Claude Code:** copy the folder to `~/.claude/skills/arquitetura-agentes/` (all projects) or to `.claude/skills/` inside a repository (that project only).

## Compatibility note

The evaluation rounds work anywhere. The unattended mode refers to tools that exist only in the Claude app with scheduled tasks (`create_trigger`, `update_trigger`, `fire_trigger`, `list_triggers`, `add_repo`). In other environments, replace them with an equivalent scheduler such as cron or GitHub Actions.

## Models

Workstreams, reviewer and report run on a mid-size model to fit usage limits; the arbiter uses a larger model with extended reasoning; the watchdog uses the smallest model; a final independent review uses the most capable model available.

## License

MIT

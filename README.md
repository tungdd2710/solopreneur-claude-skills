# standard-process — Claude Code Skill

> **Stop Claude from diving straight into code before it understands the problem.**

A structured triage-and-research workflow for Claude Code. Every request gets classified first, and substantive work follows a 6-step process that eliminates the most common Claude failure modes: scope drift, lost research, and code written without approval.

---

## Why you need this

Without a workflow skill, Claude Code tends to:

- **Over-execute trivial requests** — asks clarifying questions or writes a full plan for a 2-line fix
- **Under-execute complex requests** — jumps to code before fully understanding the problem
- **Lose research to context compaction** — you got great analysis in turn 3 and it's gone by turn 20
- **Write code before approval** — surprises you with sweeping changes when you asked a question
- **Re-research the same topics** — every session starts from zero, even on well-explored problems

This skill adds a lightweight decision layer that runs before every response. The overhead is one sentence of triage. The payoff is Claude that matches effort to complexity, saves its work, and never executes without a green light.

---

## What it does

### Step 0 — Triage (always runs first, ~1 sentence)

| Type | Definition | What happens |
|------|------------|--------------|
| **Trivial** | Yes/no, definition, single lookup | Answer directly — no overhead |
| **Micro** | 1–3 line edit, obvious typo | State assumption inline, execute |
| **Continuation** | Follow-up in an ongoing task | Inherit parent triage — no re-triage |
| **Substantive** | Multi-step, decisions, multiple files | Run full 6-step process |

### Steps 1–6 (substantive only)

```
1. ANALYSE   — decompose into sub-questions; classify each as REUSE/REFRESH/NEW
               name research surfaces upfront — sub-agent can't decide its own scope
2. CHECK     — hit existing docs, progress tracker, memory before any new search
3. RESEARCH  — exhaustively cover every named surface; save findings to files
4. BRAINSTORM — 2–3 options with tradeoffs; recommend one
5. APPROVE   — no code without explicit confirmation
6. EXECUTE   — update docs first, then code, then commit
```

**The key constraint:** research scope is declared in step 1, not discovered during step 3. This means you can audit what Claude looked at, and Claude can't quietly skip surfaces that would change the answer.

---

## Real-world effect

| Scenario | Without skill | With skill |
|----------|---------------|------------|
| "How should we handle auth?" | Claude picks one approach, starts coding | Triaged as substantive → ANALYSE → 3 options presented for approval |
| "Fix this typo in the README" | May ask clarifying questions | Triaged as micro → fixed inline in one turn |
| "What did we decide about X?" | Searches from scratch | CHECK step hits memory + docs first |
| Session continues after compaction | Research from 20 turns ago is gone | Saved to `docs/research/` — survives compaction |
| "Add feature Y" | Claude designs Y, then asks if it's right | BRAINSTORM presents options before any code is written |

---

## Install

```bash
git clone https://github.com/tungdd2710/standard-process-claude-skill
cp -r standard-process-claude-skill/skills/standard-process ~/.claude/skills/
```

Then in any Claude Code session:

```
/standard-process
```

The skill is also designed to auto-trigger when you add this line to your `CLAUDE.md`:

```markdown
## Skill routing

When the user's request matches an available skill, invoke it via the Skill
tool as your FIRST action. Key routing rules:
- ALL requests → invoke standard-process as first action for triage
```

---

## Customize for your project

Override the internal paths in your `CLAUDE.md` to match your repo structure:

```markdown
## Standard Process — path overrides

When running standard-process steps, use these project-specific paths:
- Research directory: `notes/research/`
- Project index: `docs/INDEX.md`
- Progress tracker: `.claude/progress.md`
- Memory: `.claude/memory/MEMORY.md`
```

If you have a domain-specific research surface (e.g. a Linear project, a Confluence space, a specific Slack channel), add it to the ANALYSE surface menu via `CLAUDE.md` so the skill always checks it.

---

## Design principles

- **No overhead on simple requests.** Trivial and micro triage adds zero friction.
- **Scope is declared, not discovered.** ANALYSE names surfaces before research starts — makes Claude auditable.
- **Work survives context compaction.** All research goes to files in step 3. The session can compress; the knowledge doesn't.
- **Approval is a hard gate.** Step 5 is not a suggestion. No execution without an explicit yes.
- **Portable.** Nothing in the skill is project-specific. Customize via `CLAUDE.md` overrides, not by editing the skill.

---

## Contributing

Issues and PRs welcome. The SKILL.md is intentionally generic — project-specific conventions belong in `CLAUDE.md`, not here.

## License

MIT

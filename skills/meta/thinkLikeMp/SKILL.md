---
name: thinkLikeMp
description: Meta-skill that reads the user's instruction, scans available skills, and routes to the right ones in sequence. Combines Skills for Real Engineers intelligently — diagnose bugs, write tests, improve architecture — based on what the problem actually needs. Use when you want the agent to decide which skill(s) to apply, or when the problem spans multiple skill domains.
---

# thinkLikeMp

You are a meta-skill. You do not do engineering work directly. You read the user's instruction, decide which skills from this repo are needed, and orchestrate them in the right order.

## Before anything else — open the session log and terminal

Run these commands:

```bash
mkdir -p .thinkLikeMp/logs
THINKLIKEMP_LOG=".thinkLikeMp/logs/session-$(date +%Y%m%d-%H%M%S).log"
echo "=== thinkLikeMp session started $(date) ===" >> "$THINKLIKEMP_LOG"
echo "" >> "$THINKLIKEMP_LOG"
echo "$THINKLIKEMP_LOG"
```

Save the log path output — use it for all subsequent log writes.

Then open a read-only terminal pane by running:

```bash
# On Windows (Cursor/VS Code terminal)
start cmd /k "powershell -Command \"Get-Content -Path '$THINKLIKEMP_LOG' -Wait\""
```

```bash
# On Mac/Linux
osascript -e "tell application \"Terminal\" to do script \"tail -f $THINKLIKEMP_LOG\""
```

Then output ONE line in chat:
```
thinkLikeMp activated. Watch the new terminal pane for routing decisions.
```

Nothing else in chat until routing is decided.

## Phase 1 — Build the skills index (first run only)

Check if `SKILLS-INDEX.md` exists in the same directory as this skill.

If it does NOT exist, scan these bucket folders and build it:
- `skills/engineering/`
- `skills/productivity/`
- `skills/misc/`
- `skills/personal/`

For each folder, read the `SKILL.md` inside each subfolder and extract the `name:` and `description:` from the frontmatter.

Write `SKILLS-INDEX.md` in the same directory as this skill:

```markdown
# Skills Index

Auto-generated. Rebuild after pulling upstream changes with `/thinkLikeMp rebuild-index`.

## Engineering
- **skill-name** — description

## Productivity
- **skill-name** — description

## Misc
- **skill-name** — description

## Personal
- **skill-name** — description
```

Log to session file:
```
[INDEX] Generated SKILLS-INDEX.md from X skills
```

If `SKILLS-INDEX.md` already exists, skip this phase and read it directly.

## Phase 2 — Understand the instruction

Read the user's instruction and the full conversation context.

If the instruction is clear enough to route → proceed to Phase 3.

If genuinely ambiguous → ask ONE focused question in chat, wait for answer, then proceed.

Log to session file:
```
[INSTRUCTION] <one-line summary of what user wants>
```

## Phase 3 — Route to skills

Read `SKILLS-INDEX.md`. Select the skills that are relevant to the instruction.

Rules:
- Pick minimum skills needed — don't over-route
- Order matters: diagnose bugs before writing tests, zoom-out before architecture changes
- Skip `meta/` bucket — never route to yourself

For each selected skill, log to session file:
```
[ROUTE] → <skill-name>
         Why: <one sentence reason>
         Status: QUEUED
```

Then output in chat:
```
Routing to: skill-one, skill-two. Watch terminal for details.
```

## Phase 4 — Execute skills in sequence

For each skill in the route:

1. Log to session file:
```
[START] → <skill-name> | <timestamp>
```

2. Invoke the skill. Let it run fully. Do not interrupt.

3. After skill completes, evaluate output:
   - Did it resolve the problem fully? → log DONE, check if next skill still needed
   - Did it fail or produce unexpected output? → go to Phase 5

4. Log to session file:
```
[DONE] → <skill-name> | <timestamp>
         Result: <one-line summary>
```

5. If another skill is queued → proceed. If not → go to Phase 6.

## Phase 5 — Error recovery

If a skill fails:

1. Re-read the original instruction and conversation context
2. Re-analyze — is there a better skill to use?
3. If confident in new route → silently re-route, log to session file:
```
[RECOVER] <failed-skill> failed → re-routing to <new-skill>
          Reason: <why>
```
4. If still ambiguous after re-analysis → ask user in chat:
```
<skill-name> didn't produce the expected result. Should I try <alternative>?
```

## Phase 6 — Session complete

Log to session file:
```
[COMPLETE] Session ended <timestamp>
Skills used: skill-one, skill-two
```

Output in chat:
```
Done. Skills used: skill-one → skill-two.
```

---

## Rebuilding the index

If user runs `/thinkLikeMp rebuild-index`, delete `SKILLS-INDEX.md` and re-run Phase 1.

# Skill Improvements Implementation Plan

> **For agentic workers:** REQUIRED: Use superpowers:subagent-driven-development (if subagents available) or superpowers:executing-plans to implement this plan. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Apply 10 surgical fixes to superpowers skill files to reduce context overhead, fix CSO rule violations, and clarify skill boundaries.

**Architecture:** Each task edits one file (or deletes files) independently. All tasks are parallel-safe — no task depends on another. Each task follows verify-edit-verify-commit: confirm the "before" text exists, apply the edit, confirm the "after" state, commit.

**Tech Stack:** Markdown file edits, git commits. No code, no tests, no build steps.

**Spec:** `docs/superpowers/specs/2026-04-27-skill-improvements-design.md`

**Important note on Fix 5:** The cached 5.0.4 version of `brainstorming/SKILL.md` (which was reviewed) had a subagent spec-review loop. The actual repo (5.0.7) already replaced this with a simple inline self-review. Fix 5 does not apply to the actual file — skip it.

---

## File Change Map

| Task | File | Fixes Applied |
|---|---|---|
| 1 | `commands/execute-plan.md` + `commands/write-plan.md` + `commands/brainstorm.md` | Delete (Fix 8) |
| 2 | `skills/using-superpowers/SKILL.md` | Fix 1 + Fix 2 |
| 3 | `skills/brainstorming/SKILL.md` | Fix 3 + Fix 7 + size reduction |
| 4 | `skills/finishing-a-development-branch/SKILL.md` | Fix 4 |
| 5 | `skills/dispatching-parallel-agents/SKILL.md` | Fix 6a |
| 6 | `skills/subagent-driven-development/SKILL.md` | Fix 6b |
| 7 | `skills/writing-plans/SKILL.md` | Fix 9 |
| 8 | `skills/test-driven-development/SKILL.md` | Fix 10 |
| 9 | `skills/systematic-debugging/SKILL.md` | Fix 10 |

---

## Task 1: Delete deprecated command files

**Files:**
- Delete: `commands/execute-plan.md`
- Delete: `commands/write-plan.md`
- Delete: `commands/brainstorm.md`

**Why:** These files cause three deprecated skill aliases to appear in every session's skill list, creating confusion about which skills to use. The replacement skills are fully in place.

- [ ] **Step 1: Verify files exist**

```bash
ls /Users/gaobo/repos/GitHub/superpowers/commands/
```
Expected: `brainstorm.md  execute-plan.md  write-plan.md`

- [ ] **Step 2: Delete all three files**

```bash
git -C /Users/gaobo/repos/GitHub/superpowers rm commands/execute-plan.md commands/write-plan.md commands/brainstorm.md
```

- [ ] **Step 3: Verify deletion**

```bash
ls /Users/gaobo/repos/GitHub/superpowers/commands/ 2>/dev/null || echo "directory empty or gone"
```
Expected: empty or "no such file"

- [ ] **Step 4: Commit**

```bash
git -C /Users/gaobo/repos/GitHub/superpowers commit -m "Remove deprecated command aliases (execute-plan, write-plan, brainstorm)"
```

---

## Task 2: Fix `skills/using-superpowers/SKILL.md` (Fix 1 + Fix 2)

**Files:**
- Modify: `skills/using-superpowers/SKILL.md`

**Why Fix 1:** The 1% rule + Red Flags table leave no room to skip the skill check for trivially informational questions. Add an explicit exemptions block.

**Why Fix 2:** This skill loads in every session at ~790 words. Target ~300 words by cutting non-behavioral content.

- [ ] **Step 1: Verify current state**

```bash
grep -n "How to Access Skills\|Platform Adaptation\|Skill Priority\|Skill Types\|User Instructions" \
  /Users/gaobo/repos/GitHub/superpowers/skills/using-superpowers/SKILL.md
```
Expected: lines 28, 38, 97, 107, 115 all present.

- [ ] **Step 2: Apply the full replacement**

Replace the entire file content with the following. This applies both Fix 1 (exemptions block) and Fix 2 (size reduction). The file is short enough to replace in full.

Preserved: `<SUBAGENT-STOP>`, `<EXTREMELY-IMPORTANT>`, priority hierarchy, the 1% rule, the flowchart, the Red Flags table (compressed to 6 entries).

Removed: "How to Access Skills" section (compressed to 1 line), "Platform Adaptation" section (1 line), "Skill Priority" section, "Skill Types" section, "User Instructions" section.

Added: Exemptions block before Red Flags.

New file content:

```markdown
---
name: using-superpowers
description: Use when starting any conversation - establishes how to find and use skills, requiring Skill tool invocation before ANY response including clarifying questions
---

<SUBAGENT-STOP>
If you were dispatched as a subagent to execute a specific task, skip this skill.
</SUBAGENT-STOP>

<EXTREMELY-IMPORTANT>
If you think there is even a 1% chance a skill might apply to what you are doing, you ABSOLUTELY MUST invoke the skill.

IF A SKILL APPLIES TO YOUR TASK, YOU DO NOT HAVE A CHOICE. YOU MUST USE IT.

This is not negotiable. This is not optional. You cannot rationalize your way out of this.
</EXTREMELY-IMPORTANT>

## Instruction Priority

Superpowers skills override default system prompt behavior, but **user instructions always take precedence**:

1. **User's explicit instructions** (CLAUDE.md, GEMINI.md, AGENTS.md, direct requests) — highest priority
2. **Superpowers skills** — override default system behavior where they conflict
3. **Default system prompt** — lowest priority

**In Claude Code:** Use the `Skill` tool to invoke skills. Non-CC platforms: see platform docs for equivalents.

## The Rule

**Invoke relevant or requested skills BEFORE any response or action.** Even a 1% chance a skill might apply means that you should invoke the skill to check.

```dot
digraph skill_flow {
    "User message received" [shape=doublecircle];
    "About to EnterPlanMode?" [shape=doublecircle];
    "Already brainstormed?" [shape=diamond];
    "Invoke brainstorming skill" [shape=box];
    "Might any skill apply?" [shape=diamond];
    "Invoke Skill tool" [shape=box];
    "Announce: 'Using [skill] to [purpose]'" [shape=box];
    "Has checklist?" [shape=diamond];
    "Create TodoWrite todo per item" [shape=box];
    "Follow skill exactly" [shape=box];
    "Respond (including clarifications)" [shape=doublecircle];

    "About to EnterPlanMode?" -> "Already brainstormed?";
    "Already brainstormed?" -> "Invoke brainstorming skill" [label="no"];
    "Already brainstormed?" -> "Might any skill apply?" [label="yes"];
    "Invoke brainstorming skill" -> "Might any skill apply?";

    "User message received" -> "Might any skill apply?";
    "Might any skill apply?" -> "Invoke Skill tool" [label="yes, even 1%"];
    "Might any skill apply?" -> "Respond (including clarifications)" [label="definitely not"];
    "Invoke Skill tool" -> "Announce: 'Using [skill] to [purpose]'";
    "Announce: 'Using [skill] to [purpose]'" -> "Has checklist?";
    "Has checklist?" -> "Create TodoWrite todo per item" [label="yes"];
    "Has checklist?" -> "Follow skill exactly" [label="no"];
    "Create TodoWrite todo per item" -> "Follow skill exactly";
}
```

## Exemptions

Skip the skill check only when ALL of these are true:
- The message asks for factual/definitional information only (no action requested)
- No files will be read, written, or executed
- The answer fits in 1-2 sentences

Examples: "what does X mean?", "what year was Y released?", "which flag does Z use?"

## Red Flags

These thoughts mean STOP—you're rationalizing:

| Thought | Reality |
|---------|---------|
| "This is just a simple question" | Questions are tasks. Check for skills. |
| "I need more context first" | Skill check comes BEFORE clarifying questions. |
| "Let me explore the codebase first" | Skills tell you HOW to explore. Check first. |
| "This doesn't need a formal skill" | If a skill exists, use it. |
| "I remember this skill" | Skills evolve. Read current version. |
| "This doesn't count as a task" | Action = task. Check for skills. |
```

- [ ] **Step 3: Verify word count is under 350**

```bash
wc -w /Users/gaobo/repos/GitHub/superpowers/skills/using-superpowers/SKILL.md
```
Expected: under 350 words.

- [ ] **Step 4: Verify key elements are present**

```bash
grep -c "SUBAGENT-STOP\|EXTREMELY-IMPORTANT\|Instruction Priority\|Exemptions\|Red Flags\|digraph" \
  /Users/gaobo/repos/GitHub/superpowers/skills/using-superpowers/SKILL.md
```
Expected: 6 (one match per pattern).

- [ ] **Step 5: Commit**

```bash
git -C /Users/gaobo/repos/GitHub/superpowers add skills/using-superpowers/SKILL.md
git -C /Users/gaobo/repos/GitHub/superpowers commit -m "using-superpowers: add exemptions block, compress from 790 to ~300 words"
```

---

## Task 3: Fix `skills/brainstorming/SKILL.md` (Fix 3 + Fix 7 + size reduction)

**Files:**
- Modify: `skills/brainstorming/SKILL.md`

**Why Fix 3:** Description uses first-person ("You MUST use this") and appends a workflow summary — both violate the CSO rules.

**Why Fix 7:** Visual companion offer is irrelevant in Claude Code CLI. Add exemption before the offer template.

**Why size reduction:** File is ~1155 words; target ~800 words. Cut: redundant prose in "The Process" and "After the Design" sections.

**Note:** Fix 5 (spec-review complexity gate) does NOT apply — this version already has a simple inline self-review, not a subagent loop.

- [ ] **Step 1: Verify description "before" text**

```bash
grep -n "You MUST use this\|Explores user intent" \
  /Users/gaobo/repos/GitHub/superpowers/skills/brainstorming/SKILL.md
```
Expected: line 3, both phrases present.

- [ ] **Step 2: Fix description (Fix 3)**

Replace:
```
description: "You MUST use this before any creative work - creating features, building components, adding functionality, or modifying behavior. Explores user intent, requirements and design before implementation."
```
With:
```
description: "Use when creating features, building components, adding functionality, or modifying behavior"
```

- [ ] **Step 3: Verify description fix**

```bash
grep "^description:" /Users/gaobo/repos/GitHub/superpowers/skills/brainstorming/SKILL.md
```
Expected: `description: "Use when creating features, building components, adding functionality, or modifying behavior"`

- [ ] **Step 4: Add visual companion CLI exemption (Fix 7)**

Find the line:
```
**Offering the companion:** When you anticipate that upcoming questions will involve visual content (mockups, layouts, diagrams), offer it once for consent:
```

Insert immediately before it:
```
**In Claude Code (CLI): skip the visual companion offer entirely — no browser interface is available.**

```

- [ ] **Step 5: Verify visual companion exemption**

```bash
grep -n "Claude Code (CLI)" /Users/gaobo/repos/GitHub/superpowers/skills/brainstorming/SKILL.md
```
Expected: one match in the Visual Companion section.

- [ ] **Step 6: Apply size reduction**

Target cuts (do not remove any behavior-shaping content):

**Cut "Design for isolation and clarity" block.** Remove this entire block:
```
**Design for isolation and clarity:**

- Break the system into smaller units that each have one clear purpose, communicate through well-defined interfaces, and can be understood and tested independently
- For each unit, you should be able to answer: what does it do, how do you use it, and what does it depend on?
- Can someone understand what a unit does without reading its internals? Can you change the internals without breaking consumers? If not, the boundaries need work.
- Smaller, well-bounded units are also easier for you to work with - you reason better about code you can hold in context at once, and your edits are more reliable when files are focused. When a file grows large, that's often a signal that it's doing too much.
```
This design guidance belongs in writing-plans, not in the brainstorming process skill.

**Cut "Working in existing codebases" block.** Remove this entire block:
```
**Working in existing codebases:**

- Explore the current structure before proposing changes. Follow existing patterns.
- Where existing code has problems that affect the work (e.g., a file that's grown too large, unclear boundaries, tangled responsibilities), include targeted improvements as part of the design - the way a good developer improves code they're working in.
- Don't propose unrelated refactoring. Stay focused on what serves the current goal.
```

**Compress "Exploring approaches" and "Presenting the design" subsections** — reduce each from 4-5 bullets to 2-3 essential bullets only. Remove bullets that restate the checklist.

- [ ] **Step 7: Verify word count**

```bash
wc -w /Users/gaobo/repos/GitHub/superpowers/skills/brainstorming/SKILL.md
```
Expected: 800 or fewer words.

- [ ] **Step 8: Verify key elements preserved**

```bash
grep -c "HARD-GATE\|Checklist\|Process Flow\|Visual Companion\|writing-plans" \
  /Users/gaobo/repos/GitHub/superpowers/skills/brainstorming/SKILL.md
```
Expected: 5.

- [ ] **Step 9: Commit**

```bash
git -C /Users/gaobo/repos/GitHub/superpowers add skills/brainstorming/SKILL.md
git -C /Users/gaobo/repos/GitHub/superpowers commit -m "brainstorming: fix description CSO violations, add CLI exemption for visual companion, reduce size"
```

---

## Task 4: Fix `skills/finishing-a-development-branch/SKILL.md` (Fix 4)

**Files:**
- Modify: `skills/finishing-a-development-branch/SKILL.md`

**Why:** Description appends a workflow summary ("guides completion of development work by presenting structured options for merge, PR, or cleanup") after the triggering condition — a CSO rule violation.

- [ ] **Step 1: Verify "before" text**

```bash
grep "^description:" /Users/gaobo/repos/GitHub/superpowers/skills/finishing-a-development-branch/SKILL.md
```
Expected: description ending with "guides completion of development work by presenting structured options for merge, PR, or cleanup"

- [ ] **Step 2: Apply fix**

Replace the description line:
```
description: Use when implementation is complete, all tests pass, and you need to decide how to integrate the work - guides completion of development work by presenting structured options for merge, PR, or cleanup
```
With:
```
description: Use when implementation is complete, all tests pass, and you need to decide how to integrate the work
```

- [ ] **Step 3: Verify "after" text**

```bash
grep "^description:" /Users/gaobo/repos/GitHub/superpowers/skills/finishing-a-development-branch/SKILL.md
```
Expected: description ending with "how to integrate the work" — no workflow summary.

- [ ] **Step 4: Commit**

```bash
git -C /Users/gaobo/repos/GitHub/superpowers add skills/finishing-a-development-branch/SKILL.md
git -C /Users/gaobo/repos/GitHub/superpowers commit -m "finishing-a-development-branch: remove workflow summary from description"
```

---

## Task 5: Fix `skills/dispatching-parallel-agents/SKILL.md` (Fix 6a)

**Files:**
- Modify: `skills/dispatching-parallel-agents/SKILL.md`

**Why:** No cross-reference to `subagent-driven-development`, causing confusion when tasks form part of a structured implementation plan.

- [ ] **Step 1: Verify "Don't use when" section exists**

```bash
grep -n "Don't use when" /Users/gaobo/repos/GitHub/superpowers/skills/dispatching-parallel-agents/SKILL.md
```
Expected: one match.

- [ ] **Step 2: Find the last bullet in "Don't use when" section**

```bash
grep -n "Failures are related\|shared state\|interfere" \
  /Users/gaobo/repos/GitHub/superpowers/skills/dispatching-parallel-agents/SKILL.md
```
Note the last bullet line number.

- [ ] **Step 3: Add cross-reference after the last "Don't use when" bullet**

After:
```
- Agents would interfere with each other
```
Add:
```
- Tasks form part of a structured implementation plan → use subagent-driven-development instead
```

- [ ] **Step 4: Verify addition**

```bash
grep "subagent-driven-development" /Users/gaobo/repos/GitHub/superpowers/skills/dispatching-parallel-agents/SKILL.md
```
Expected: one match in the "Don't use when" section.

- [ ] **Step 5: Commit**

```bash
git -C /Users/gaobo/repos/GitHub/superpowers add skills/dispatching-parallel-agents/SKILL.md
git -C /Users/gaobo/repos/GitHub/superpowers commit -m "dispatching-parallel-agents: add cross-reference to subagent-driven-development"
```

---

## Task 6: Fix `skills/subagent-driven-development/SKILL.md` (Fix 6b)

**Files:**
- Modify: `skills/subagent-driven-development/SKILL.md`

**Why:** No cross-reference to `dispatching-parallel-agents`. The `**vs. Executing Plans (parallel session):**` block is the right anchor — add a sibling `**vs. Dispatching Parallel Agents:**` block immediately after it.

- [ ] **Step 1: Verify anchor exists**

```bash
grep -n "vs. Executing Plans" /Users/gaobo/repos/GitHub/superpowers/skills/subagent-driven-development/SKILL.md
```
Expected: one match around line 34.

- [ ] **Step 2: Find the end of the "vs. Executing Plans" block**

```bash
sed -n '34,42p' /Users/gaobo/repos/GitHub/superpowers/skills/subagent-driven-development/SKILL.md
```
Identify the blank line after the last bullet of the "vs. Executing Plans" block (before `## The Process`).

- [ ] **Step 3: Insert cross-reference after the "vs. Executing Plans" block**

After:
```
- Faster iteration (no human-in-loop between tasks)
```
Add:
```

**vs. Dispatching Parallel Agents (debugging multiple failures):**
For debugging multiple independent failures across subsystems — not executing a plan — use dispatching-parallel-agents instead.
```

- [ ] **Step 4: Verify addition**

```bash
grep "dispatching-parallel-agents" /Users/gaobo/repos/GitHub/superpowers/skills/subagent-driven-development/SKILL.md
```
Expected: one match.

- [ ] **Step 5: Commit**

```bash
git -C /Users/gaobo/repos/GitHub/superpowers add skills/subagent-driven-development/SKILL.md
git -C /Users/gaobo/repos/GitHub/superpowers commit -m "subagent-driven-development: add cross-reference to dispatching-parallel-agents"
```

---

## Task 7: Fix `skills/writing-plans/SKILL.md` (Fix 9)

**Files:**
- Modify: `skills/writing-plans/SKILL.md`

**Why:** Two occurrences of `docs/superpowers/plans/` hardcode a superpowers-branded path that won't exist in most projects.

- [ ] **Step 1: Verify both occurrences**

```bash
grep -n "superpowers/plans" /Users/gaobo/repos/GitHub/superpowers/skills/writing-plans/SKILL.md
```
Expected: 2 matches — one at ~line 18 (with `YYYY-MM-DD-<feature-name>`) and one at ~line 138 (with `<filename>`).

- [ ] **Step 2: Fix line 18**

Replace:
```
**Save plans to:** `docs/superpowers/plans/YYYY-MM-DD-<feature-name>.md`
```
With:
```
**Save plans to:** `docs/plans/YYYY-MM-DD-<feature-name>.md`
- (Follow project conventions if a plans directory already exists elsewhere)
```

- [ ] **Step 3: Fix line 138**

Replace:
```
**"Plan complete and saved to `docs/superpowers/plans/<filename>.md`. Two execution options:**
```
With:
```
**"Plan complete and saved to `docs/plans/<filename>.md`. Two execution options:**
```

- [ ] **Step 4: Verify no remaining occurrences**

```bash
grep "superpowers/plans" /Users/gaobo/repos/GitHub/superpowers/skills/writing-plans/SKILL.md
```
Expected: no output (zero matches).

- [ ] **Step 5: Commit**

```bash
git -C /Users/gaobo/repos/GitHub/superpowers add skills/writing-plans/SKILL.md
git -C /Users/gaobo/repos/GitHub/superpowers commit -m "writing-plans: replace hard-coded superpowers/plans/ path with project-neutral docs/plans/"
```

---

## Task 8: Fix `skills/test-driven-development/SKILL.md` (Fix 10)

**Files:**
- Modify: `skills/test-driven-development/SKILL.md`

**Why:** At ~1496 words, this skill is too large for something invoked on every feature or bugfix. The "Why Order Matters" section (lines ~206-254) is 48 lines of prose that restates content already covered by the Common Rationalizations table and Red Flags list.

- [ ] **Step 1: Verify "Why Order Matters" section exists**

```bash
grep -n "Why Order Matters" /Users/gaobo/repos/GitHub/superpowers/skills/test-driven-development/SKILL.md
```
Expected: one match around line 206.

- [ ] **Step 2: Verify Common Rationalizations table exists (must be preserved)**

```bash
grep -n "Common Rationalizations\|Red Flags - STOP" \
  /Users/gaobo/repos/GitHub/superpowers/skills/test-driven-development/SKILL.md
```
Expected: both present.

- [ ] **Step 3: Remove the "Why Order Matters" section**

Delete the entire block from `## Why Order Matters` through the end of the last subsection ("30 minutes of tests after ≠ TDD. You get coverage, lose proof tests work."), stopping before `## Common Rationalizations`.

The block to remove spans approximately lines 206-254. Verify boundaries with:
```bash
sed -n '200,260p' /Users/gaobo/repos/GitHub/superpowers/skills/test-driven-development/SKILL.md
```

- [ ] **Step 4: Verify word count reduction**

```bash
wc -w /Users/gaobo/repos/GitHub/superpowers/skills/test-driven-development/SKILL.md
```
Expected: under 1150 words (down from ~1496).

- [ ] **Step 5: Verify Iron Law and Red Flags are still present**

```bash
grep -c "Iron Law\|NO PRODUCTION CODE\|Red Flags - STOP\|Common Rationalizations" \
  /Users/gaobo/repos/GitHub/superpowers/skills/test-driven-development/SKILL.md
```
Expected: 4.

- [ ] **Step 6: Commit**

```bash
git -C /Users/gaobo/repos/GitHub/superpowers add skills/test-driven-development/SKILL.md
git -C /Users/gaobo/repos/GitHub/superpowers commit -m "test-driven-development: remove redundant 'Why Order Matters' section (~350 words)"
```

---

## Task 9: Fix `skills/systematic-debugging/SKILL.md` (Fix 10)

**Files:**
- Modify: `skills/systematic-debugging/SKILL.md`

**Why:** At ~1504 words, this skill is too large. Two sections can be removed without losing behavioral guidance:

1. **"your human partner's Signals You're Doing It Wrong"** (lines ~234-243) — 10 lines of signal-watching advice. This describes meta-signals that belong in a coaching context, not in a process skill loaded on every bug.

2. **"Real-World Impact"** (lines ~290-296) — 7 lines of statistics ("15-30 minutes", "95% first-time fix rate"). Aspirational filler with no behavioral value.

- [ ] **Step 1: Verify both sections exist**

```bash
grep -n "your human partner's Signals\|Real-World Impact" \
  /Users/gaobo/repos/GitHub/superpowers/skills/systematic-debugging/SKILL.md
```
Expected: both present.

- [ ] **Step 2: Remove "your human partner's Signals You're Doing It Wrong" section**

Delete from `## your human partner's Signals You're Doing It Wrong` through `**When you see these:** STOP. Return to Phase 1.`

Verify boundary:
```bash
sed -n '230,248p' /Users/gaobo/repos/GitHub/superpowers/skills/systematic-debugging/SKILL.md
```

- [ ] **Step 3: Remove "Real-World Impact" section**

Delete from `## Real-World Impact` to end of file (it's the last section).

Verify boundary:
```bash
tail -15 /Users/gaobo/repos/GitHub/superpowers/skills/systematic-debugging/SKILL.md
```

- [ ] **Step 4: Verify word count reduction**

```bash
wc -w /Users/gaobo/repos/GitHub/superpowers/skills/systematic-debugging/SKILL.md
```
Expected: under 1350 words (down from ~1504).

- [ ] **Step 5: Verify Iron Law and Red Flags are still present**

```bash
grep -c "Iron Law\|NO FIXES WITHOUT\|Red Flags - STOP\|Common Rationalizations" \
  /Users/gaobo/repos/GitHub/superpowers/skills/systematic-debugging/SKILL.md
```
Expected: 4.

- [ ] **Step 6: Commit**

```bash
git -C /Users/gaobo/repos/GitHub/superpowers add skills/systematic-debugging/SKILL.md
git -C /Users/gaobo/repos/GitHub/superpowers commit -m "systematic-debugging: remove 'Signals' and 'Real-World Impact' sections (~150 words)"
```

---

## Final Verification

After all tasks complete:

- [ ] **Confirm deprecated commands are gone**

```bash
ls /Users/gaobo/repos/GitHub/superpowers/commands/ 2>/dev/null && echo "FAIL: files remain" || echo "PASS: directory empty"
```

- [ ] **Confirm no remaining CSO violations in descriptions**

```bash
grep -n "Explores user intent\|guides completion\|You MUST use this" \
  /Users/gaobo/repos/GitHub/superpowers/skills/*/SKILL.md
```
Expected: no output.

- [ ] **Confirm cross-references are in place**

```bash
grep -l "subagent-driven-development" /Users/gaobo/repos/GitHub/superpowers/skills/dispatching-parallel-agents/SKILL.md
grep -l "dispatching-parallel-agents" /Users/gaobo/repos/GitHub/superpowers/skills/subagent-driven-development/SKILL.md
```
Expected: both files listed.

- [ ] **Confirm hard-coded path is gone**

```bash
grep -r "superpowers/plans" /Users/gaobo/repos/GitHub/superpowers/skills/
```
Expected: no output.

- [ ] **Confirm using-superpowers has Exemptions section**

```bash
grep "Exemptions" /Users/gaobo/repos/GitHub/superpowers/skills/using-superpowers/SKILL.md
```
Expected: one match.

- [ ] **Confirm brainstorming has CLI exemption**

```bash
grep "Claude Code (CLI)" /Users/gaobo/repos/GitHub/superpowers/skills/brainstorming/SKILL.md
```
Expected: one match.

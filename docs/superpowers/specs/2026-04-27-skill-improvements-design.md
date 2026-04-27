# Skill Improvements Design — Personal Fork

**Date:** 2026-04-27
**Scope:** Personal fork only (not upstream contribution)
**Strategy:** Surgical patches — one targeted fix per issue, no restructuring

---

## Background

A Claude Code perspective review of the superpowers skills identified 10 issues across five categories. All fixes are additive or minimal edits; no behavior-shaping content (Iron Law, Red Flags tables, rationalization counters) is removed.

---

## Fix 1 — `using-superpowers`: Add trigger exemptions

**Problem:** The 1% rule + Red Flags table cover essentially every mental state Claude can reach, making it impossible to skip the skill check even for trivially informational questions with no action.

**Fix:** Insert an "Exemptions" block before the Red Flags table. The 1% rule and table stay intact.

**Exemption criteria — ALL must be true:**
- The message asks for factual/definitional information only (no action requested)
- No files will be read, written, or executed
- The answer fits in 1-2 sentences

**Examples of exempt messages:** "what does X mean?", "what year was Y released?", "which flag does Z use?"

---

## Fix 2 — `using-superpowers`: Compress from ~790 → ~300 words

**Problem:** This skill loads in every session. At ~790 words it inflates context significantly; the writing-skills guide targets <150 words for always-loaded workflows.

**Fix:** Surgical prose cuts — nothing behavioral removed:
- Compress Red Flags table from 12 entries to 6 (consolidate variations of the same rationalization pattern)
- Reduce platform adaptation section from ~100 words to 2 sentences
- Remove "Skill Priority" section (already encoded in the flowchart)
- Trim verbose flowchart surrounding prose

**Preserved:** `<SUBAGENT-STOP>`, priority hierarchy, 1% rule, Red Flags table, exemptions from Fix 1.

---

## Fix 3 — `brainstorming`: Fix description

**Problem:** Description (a) uses first-person ("You MUST use this"), and (b) appends a workflow summary ("Explores user intent, requirements and design before implementation") — both violations of the CSO rules in writing-skills, which require third-person triggering conditions only.

**Before:**
```
"You MUST use this before any creative work - creating features, building
components, adding functionality, or modifying behavior. Explores user intent,
requirements and design before implementation."
```

**After:**
```
"Use when creating features, building components, adding functionality,
or modifying behavior"
```

---

## Fix 4 — `finishing-a-development-branch`: Fix description

**Problem:** Description appends a workflow summary after the triggering condition.

**Before:**
```
"Use when implementation is complete, all tests pass, and you need to decide
how to integrate the work - guides completion of development work by presenting
structured options for merge, PR, or cleanup"
```

**After:**
```
"Use when implementation is complete, all tests pass, and you need to decide
how to integrate the work"
```

---

## Fix 5 — `brainstorming`: Gate spec-review loop on complexity

**Problem:** The spec-review subagent loop (step 7) fires unconditionally — even for a simple CLI flag addition or single-file change.

**Fix:** Add a complexity gate before step 7. Skip the spec-review loop when ALL are true:
- ≤3 files will be modified
- Single clear responsibility (no architectural decisions)
- No new interfaces or data models introduced

For all other changes, the full spec-review loop runs unchanged.

---

## Fix 6 — Clarify overlap between `dispatching-parallel-agents` and `subagent-driven-development`

**Problem:** Both skills handle independent tasks via subagents. The distinction (debugging multiple failures vs. executing implementation plans) is not obvious from descriptions alone.

**Fix — `dispatching-parallel-agents`:** Add to "Don't use when" list:
```
- Tasks form part of a structured implementation plan → use subagent-driven-development instead
```

**Fix — `subagent-driven-development`:** Add a prose note at the end of the "When to Use" section, after the `**vs. Executing Plans (parallel session):**` block (which ends around line 38):
```
**vs. Dispatching Parallel Agents (debugging multiple failures):**
For debugging multiple independent failures across subsystems — not executing a plan — use dispatching-parallel-agents instead.
```

---

## Fix 7 — `brainstorming`: Add Claude Code visual companion exemption

**Problem:** The visual companion offer (browser-based mockups/diagrams) is irrelevant in Claude Code's CLI environment and produces a confusing or dead offer.

**Fix:** Add one line to the Visual Companion section, before the offer template:
```
In Claude Code (CLI): skip the visual companion offer entirely —
no browser interface is available.
```

---

## Fix 8 — Delete deprecated command files

**Problem:** `commands/execute-plan.md`, `commands/write-plan.md`, `commands/brainstorm.md` cause the deprecated skill aliases to appear in every session's skill list, creating confusion about which skills to use.

**Fix:** Delete all three files. The replacement skills (`executing-plans`, `writing-plans`, `brainstorming`) are fully in place.

---

## Fix 9 — `writing-plans`: Remove hard-coded path

**Problem:** `docs/superpowers/plans/` is a superpowers-branded path that won't exist in most projects.

Two occurrences of `docs/superpowers/plans/` in the file. Both must be updated.

**Line 18 — before:**
```
Save plans to: `docs/superpowers/plans/YYYY-MM-DD-<feature-name>.md`
```
**Line 18 — after:**
```
Save plans to: `docs/plans/YYYY-MM-DD-<feature-name>.md`
(Follow project conventions if a plans directory already exists elsewhere)
```

**Line 138 — before** (note: this occurrence omits the date prefix):
```
docs/superpowers/plans/<filename>.md
```
**Line 138 — after:**
```
docs/plans/<filename>.md
```

---

## Fix 10 — Size reduction: `test-driven-development`, `systematic-debugging`, `brainstorming`

**Problem:** All three are ~1500 words against the writing-skills target of <500 words. Each is invoked on every feature implementation or bug fix.

**Fix:** Target ~900 words each (40% reduction). Cuts are limited to:
- Repeated restatements of the Iron Law (stated once, then restated in 2-3 other sections)
- Examples illustrating the same pattern already shown by another example
- Connecting prose between sections that adds no information

**Not touched:** Iron Law statements, Red Flags tables, rationalization counters, all discipline-enforcing content.

---

## File change summary

| File | Change type |
|---|---|
| `skills/using-superpowers/SKILL.md` | Edit — exemptions block + size reduction |
| `skills/brainstorming/SKILL.md` | Edit — description fix + complexity gate + visual companion exemption + size reduction |
| `skills/finishing-a-development-branch/SKILL.md` | Edit — description fix |
| `skills/dispatching-parallel-agents/SKILL.md` | Edit — add cross-reference to subagent-driven-development |
| `skills/subagent-driven-development/SKILL.md` | Edit — add cross-reference to dispatching-parallel-agents |
| `skills/test-driven-development/SKILL.md` | Edit — size reduction |
| `skills/systematic-debugging/SKILL.md` | Edit — size reduction |
| `skills/writing-plans/SKILL.md` | Edit — remove hard-coded path |
| `commands/execute-plan.md` | Delete |
| `commands/write-plan.md` | Delete |
| `commands/brainstorm.md` | Delete |

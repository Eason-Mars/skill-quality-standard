---
name: your-skill-name
description: >
  Use when [trigger scenarios — describe the primary use case in plain language].
  Trigger phrases: "[phrase1]", "[phrase2]", "[phrase3]", "[phrase4]", "[phrase5]".
  Use when: [scenario A] / [scenario B] / [scenario C].
  NOT for: [exclusion 1] / [exclusion 2].
---

# [Skill Name]

> **Why this exists**: [One sentence explaining the core problem this solves — what would go wrong without this Skill?]

---

## Quick Reference

| Situation | Action | Notes |
|-----------|--------|-------|
| [Most common scenario] | [Concrete action] | [Any caveats] |
| [Second scenario] | [Concrete action] | [Any caveats] |
| [Edge case] | [How to handle it] | [Warning if any] |

---

## Workflow

### Step 1: [First Action]

[Instructions for step 1. Be specific about what the LLM should do, what to check, what tool to call.]

### Step 2: [Second Action]

[Instructions for step 2.]

### Step 3: [Third Action]

[Instructions for step 3. If this produces output, specify the exact path.]

Output path: `[path template, e.g., output/{date}-{skill-name}-{id}.md]`

---

## Output Format

[Describe the expected output. What does a good result look like? What format/structure?]

---

## Error Handling

| Error | Likely Cause | Resolution |
|-------|-------------|------------|
| [Error type] | [Why it happens] | [How to fix] |
| [Error type] | [Why it happens] | [How to fix] |

---

## Reference Documents

- [Topic A] → `references/[file-a].md`
- [Topic B] → `references/[file-b].md`
- Output template → `assets/[template-name]/`

---

## Commissioning Checklist

Before registering this Skill as active:

- [ ] Step 1: `python3 scripts/[main-script].py --help` runs without error
- [ ] Step 2: All paths in this SKILL.md are absolute (no `~` shortcuts)
- [ ] Step 2: No stale references to old file/skill names
- [ ] Step 3: Ran with real data, output file exists at expected path
- [ ] Step 3: Output content verified correct (not empty, not error)
- [ ] Trigger eval: ≥ 80% pass rate on `evals/trigger-evals.json`

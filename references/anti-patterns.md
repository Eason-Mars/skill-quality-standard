# Anti-Patterns: Common OpenClaw Skill Mistakes

These are mistakes observed in real OpenClaw skill development. Each one has a documented root cause and a concrete fix. Avoid them.

---

## Anti-Pattern 1: HTML Templates in references/

### What It Looks Like

```
my-skill/
├── SKILL.md
└── references/
    ├── style-guide.md
    └── article-template.html   ← WRONG
```

### Why It Happens

The builder puts everything "the LLM might need" in references/. HTML templates seem like reference material, so they go there.

### Why It's Wrong

`references/` is loaded into the LLM's context window on every invocation. An HTML template with CSS, layout structure, and placeholder markup can easily be 3,000–8,000 tokens. Loading it every time burns context window whether or not it's needed.

### The Fix

```
my-skill/
├── SKILL.md
└── references/
│   └── style-guide.md      ← LLM reads this
└── assets/
    └── article-template.html   ← LLM only gets the PATH
```

In SKILL.md: `Template path: assets/article-template.html`

The LLM knows where the template is. It passes the path to the script. The script reads the template and fills it in. The LLM never loads the template content directly.

**Rule**: If a file is bigger than 500 tokens and the LLM doesn't need to read it word-by-word, it belongs in `assets/`, not `references/`.

---

## Anti-Pattern 2: SKILL.md Over 500 Lines

### What It Looks Like

A SKILL.md with 800+ lines covering every edge case, all the reference information, the full style guide, and detailed instructions for every scenario.

### Why It Happens

"I want the LLM to have everything it needs." Adding more feels safer than pruning.

### Why It's Wrong

SKILL.md is always loaded in full. At 800 lines, it consumes 4,000–6,000 tokens on every invocation — whether the user is doing a simple task or a complex one. Most of that content is not needed for most invocations.

Additionally, long SKILL.md files have low signal density. The LLM has to wade through more text to find what it needs, increasing the chance it misses or misapplies instructions.

### The Fix

Keep SKILL.md under 300 lines (ideal) or 500 lines (absolute maximum). Move detail into references/:

```
SKILL.md (< 300 lines):
- Module summaries with key rules
- Links to reference docs for detail
- Quick checklist

references/design-principles.md:
- Full Q1-Q5 explanations with examples

references/description-as-trigger.md:
- All the detail on trigger phrases
```

In SKILL.md: `→ Full detail: references/design-principles.md`

The LLM loads the full detail only when it needs it. For quick invocations, SKILL.md alone is sufficient.

---

## Anti-Pattern 3: Description Without Trigger Phrases

### What It Looks Like

```yaml
description: >
  A comprehensive tool for building and maintaining OpenClaw skills.
  Provides guidance on skill development best practices and ensures
  high-quality outputs through systematic evaluation.
```

### Why It Happens

The builder writes the description like documentation — explaining what the Skill does. This is natural but wrong.

### Why It's Wrong

The description is a routing signal, not documentation. The routing system needs to match user messages against it. A description full of nouns ("comprehensive tool", "systematic evaluation") will never match the verb phrases users actually type ("build a skill", "why isn't my skill working").

Trigger rate from a description like the example above: typically 20–40%.

### The Fix

```yaml
description: >
  Use when building a new OpenClaw skill from scratch, debugging why a skill
  isn't triggering, or evaluating skill quality.
  Trigger phrases: "build a new skill", "skill not triggering", "skill quality",
  "how to write description", "commissioning", "three-layer architecture".
  NOT for: using an existing skill to solve a business problem.
```

**Rule**: Every description must contain at least 5 quoted trigger phrases — words users literally type. If you haven't written "Trigger phrases:" in your description, you haven't finished it.

---

## Anti-Pattern 4: Commissioning Without All Three Steps

### What It Looks Like

```markdown
# CAPABILITIES.md

## Skills
- my-new-skill: Analyzes X and produces Y ✅ Activated 2025-01-15
```

The skill is listed as activated, but only Step 1 (directory structure) was completed. Steps 2 and 3 were skipped because "it should work."

### Why It Happens

Step 1 (structure) feels like done. The files exist, the SKILL.md is written. Adding it to CAPABILITIES.md feels like the natural conclusion. Steps 2 and 3 feel like extra work.

### Why It's Wrong

**Step 2 (reference integration)** catches path issues. Using `~/` shortcuts instead of absolute paths means the Skill breaks when run from a different context. Using old skill names in cross-references creates silent failures.

**Step 3 (real-world verification)** catches everything else. Until you run the Skill with real data and see actual output, you don't know if it works. "Should work" has a 30–50% failure rate in practice.

### The Fix

Don't add anything to CAPABILITIES.md until all three steps are verified:

```
Commissioning Checklist:
[ ] Step 1: python3 scripts/main.py --help runs without error
[ ] Step 2: All paths in SKILL.md are absolute or workspace-relative (no ~/)
[ ] Step 2: No references to old/renamed skill files
[ ] Step 3: Ran with real data, verified output file exists at expected path
[ ] Step 3: Output content is correct (not empty, not error message)
Only after all boxes checked: add to CAPABILITIES.md
```

**Rule**: "Should work" is not a commissioning step result. You need evidence: an output file, a screenshot, a log showing the right values.

---

## Anti-Pattern 5: Using `~` Paths in SKILL.md

### What It Looks Like

```markdown
## Running the script

```bash
python3 ~/workspace/scripts/run_analysis.py --input ~/workspace/data/input.csv
```
```

### Why It Happens

`~` works on the developer's machine when they write the SKILL.md. It's convenient shorthand.

### Why It's Wrong

When an LLM constructs a command from SKILL.md instructions, it uses the path literally. `~` expands differently depending on:
- Which user account the agent runs under
- Whether the shell is a login shell or non-login
- The execution context (some scripts run in contexts where `~` doesn't expand)

More importantly, when the LLM generates code referencing `~/workspace/...`, it may not know whose home directory that refers to in the current context.

### The Fix

Always use workspace-relative paths anchored to the known workspace root:

```markdown
## Running the script

```bash
python3 /Users/username/.openclaw/workspace/scripts/run_analysis.py \
  --input /Users/username/.openclaw/workspace/data/input.csv
```

Or reference the workspace variable:
```bash
WORKSPACE=/Users/username/.openclaw/workspace
python3 $WORKSPACE/scripts/run_analysis.py --input $WORKSPACE/data/input.csv
```

**Rule**: No `~` in any SKILL.md path. Always use the full absolute path from root.

---

## Anti-Pattern 6: Declaration ≠ Integration

### What It Looks Like

```markdown
# CAPABILITIES.md

## Data Sources
- Tushare Pro: Real-time A-share data ✅ Integrated 2025-03-08
```

Meanwhile, every script still uses the old Yahoo Finance API. Tushare Pro was "integrated" by updating the CAPABILITIES.md entry. No script was changed.

### Why It Happens

This is the most insidious anti-pattern because it's done in good faith. The developer intends to integrate. They document the plan. Then circumstances intervene, and the documentation becomes permanent wishful thinking.

### Why It's Wrong

Other agents and team members read CAPABILITIES.md and make decisions based on it. If it says Tushare Pro is integrated but it isn't, they'll build on a false assumption. Errors propagate silently for months.

In one documented case: 24 scripts were listed as integrated with a new data provider in CAPABILITIES.md. An audit 12 days later found zero scripts had actually been modified.

### The Fix

**Three-step commissioning is the only way to register a capability:**

```
Integration Verification:
Step 1: grep -rn "tushare\|new_provider" scripts/*.py | grep -v ".bak"
        → Must find actual imports/calls in the scripts listed
Step 2: python3 -c "from new_provider import DataProvider; dp = DataProvider(); print('OK')"
        → Must run without error
Step 3: python3 scripts/actual_script.py --mode live --ticker 600519
        → Must return real data, not mock/error

Only after Step 3 passes: update CAPABILITIES.md
```

**Rule**: CAPABILITIES.md is a record of verified capabilities, not intentions. Every entry must have passed all three commissioning steps. If a step is skipped, the entry is wrong.

---

## Summary Table

| Anti-Pattern | Root Cause | Cost | Fix |
|--------------|------------|------|-----|
| HTML in references/ | Confusion about folder purpose | +3,000–8,000 tokens per invocation | Move to assets/ |
| SKILL.md > 500 lines | "More is safer" mindset | High context cost, low signal density | Extract to references/, keep SKILL.md < 300 lines |
| Description without trigger phrases | Writing docs, not routing signals | Trigger rate 20–40% | Add "Trigger phrases:" section with 5+ quoted phrases |
| Commissioning skip | Step 1 feels like done | Silent failures, "should work" bugs | All 3 steps required, evidence required |
| `~` paths | Convenience on developer's machine | Path resolution failures in agent context | Use absolute paths from root |
| Declaration ≠ Integration | Good intentions, no follow-through | False capabilities, silent errors | Three-step verification required before CAPABILITIES.md entry |

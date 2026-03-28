---
name: skill-quality-standard
description: >
  Use when building a new OpenClaw skill from scratch, debugging why a skill isn't triggering,
  writing or improving a skill description, evaluating skill quality, or following best practices
  for skill development. This is the OpenClaw-native practice guide for skill-creator — battle-tested
  standards derived from real skill development (lesson-keeper 20/20 trigger test, trigger-bench).
  Use when: "build a new skill", "skill not triggering", "how to write description",
  "skill quality", "commissioning", "eval-driven development", "prove my skill works",
  "skill design principles", "three-layer architecture".
  NOT for: using an existing skill to solve a business problem.
---

# Skill Quality Standard

## Core Principle

A good Skill = Accurate Triggering × Effective Execution × Measurable Quality

A Skill without evaluation is not finished — it's waiting to fail.

---

## Module 1: Five Design Questions (Answer Before Building)

Answer all five before writing any code or documentation.

### Q1: Can a single LLM call handle this?
- **Yes** → Don't build a Skill; a good prompt is enough
- **No** (requires tools, scripts, multi-step processes, or specific standards) → Worth building

**Decision rule**: If the LLM can complete this task using its own knowledge alone, no Skill is needed.

### Q2: What is the single output artifact?
Define all three:
- **File type**: `.md` / `.html` / `.json` / `.png`
- **Path template**: `output/{date}-{skill-name}-{project}.md`
- **Consumer**: Human reader? Another script? A downstream Skill?

**Vague output is the most common reason Skills go unused.**

### Q3: Is there a loop? How is it designed?
- Loops with humans: soft exit ("Say 'continue' to go deeper")
- Loops with AI/scripts: hard ceiling (`MAX_ITERATIONS = 5`)
- No loop: explicitly state it's single-execution

### Q4: If the LLM hallucinates at each stage, what's the worst outcome?
Assess risk per stage:
- Low risk (drafting text) → Allow LLM autonomy, human reviews
- High risk (writing files, calling APIs) → Require preview and confirmation

### Q5: Where does output live? How does downstream discover it?
- Define exact output path (not just "save locally")
- If a downstream Skill consumes it, define the discovery mechanism (path convention / filename template)

→ Full detail: `references/design-principles.md`

---

## Module 2: Three-Layer Resource Architecture

```
your-skill/
├── SKILL.md          ← LLM reads: trigger conditions + workflow + spec refs
├── scripts/          ← Executable code: deterministic operations
├── references/       ← LLM loads into context: specs, style guides, frameworks
├── assets/           ← LLM only references path: templates, fonts, config files
└── evals/            ← Evaluation cases: trigger-evals.json / quality-cases.json
```

### Division of Responsibility

| Location | What Goes Here | Why |
|----------|---------------|-----|
| `scripts/` | Python/Shell scripts | Deterministic, not LLM-guessed |
| `references/` | Spec docs, style guides | LLM reads and understands |
| `assets/` | HTML templates, fonts, configs | LLM references path only, no context cost |
| `evals/` | Test case JSON files | Separate from main flow, for evaluation only |

**Most common mistake**: Putting HTML templates in `references/`, consuming large amounts of context window unnecessarily.

**Rule: Output templates always go in `assets/`, never in `references/`.**

---

## Module 3: Description = Trigger Hook (Key Insight)

Description is not just a summary — it's the recall mechanism that determines whether the Skill gets selected.

### Four Required Elements

```
1. What it does (one-line core function)
2. Trigger phrases (actual words users say, comma-separated)
3. Use when: explicit trigger scenarios
4. NOT for: explicit exclusions (prevents mis-triggering)
```

### Good Description Example

```
When the user needs to analyze financial statements, extract key metrics,
or summarize quarterly results. Use when: financial analysis / earnings summary /
balance sheet review / revenue breakdown / Q4 results. NOT for: real-time stock
prices (use market-data skill) / tax filing advice.
```

### Trigger Rate Testing

After writing description, test immediately:

```bash
# Use trigger-bench's OpenClaw-compatible version
python3 ~/.openclaw/workspace/.agents/skills/trigger-bench/scripts/run_eval_openclaw.py \
  --eval-set evals/trigger-evals.json \
  --skill-path ~/.openclaw/workspace/.agents/skills/your-skill \
  --model claude-sonnet-4-5 \
  --runs-per-query 3 \
  --verbose
```

- Target: trigger rate ≥ 80% (should-trigger scenarios)
- < 60%: Must fix description before going live
- Fix strategy: Are trigger phrases too technical? Is the scope too narrow?

→ Full detail: `references/description-as-trigger.md`

---

## Module 4: Three-Step Commissioning Verification

**Incomplete commissioning = not activated, not in capabilities list.**

### Step 1: Basic Integration
- Directory structure is complete
- All scripts run: `python3 scripts/main.py --help` works
- Dependencies installed successfully

### Step 2: Reference Integration
- Capability registry updated with correct Skill path
- Paths use workspace-relative format, not `~` shortcuts
- No stale references to old skill names remain

### Step 3: Real-World Verification
- Run a complete workflow with **real data** (not test/mock data)
- Manually inspect the output, confirm it meets expectations
- **"Should work" is not a verification result** — you need an output file or screenshot as evidence

---

## Module 5: Evaluation-Driven Development

### Phase 1 — Trigger Rate (Required for All Skills)

Write test cases in `evals/trigger-evals.json`:
- `should_trigger`: 5+ messages that should trigger this Skill
- `should_not_trigger`: 5+ messages that should not trigger

Run:
```bash
# Use trigger-bench's OpenClaw-compatible version
python3 ~/.openclaw/workspace/.agents/skills/trigger-bench/scripts/run_eval_openclaw.py \
  --eval-set evals/trigger-evals.json \
  --skill-path ~/.openclaw/workspace/.agents/skills/your-skill \
  --model claude-sonnet-4-5 \
  --runs-per-query 3 \
  --verbose
```

Threshold: ≥ 80% to go live, < 60% requires mandatory fixes.

### Phase 2 — Quality Evaluation (Required for Content Skills)

Define in `evals/quality-cases.json`:
- 3-5 real usage scenarios with explicit expectations
- Grader scores each case: PASS/FAIL + improvement suggestions
- Target: average ≥ 3.5/5.0, otherwise revise

### Phase 3 — Anti-Overfitting

**Core rule: Don't use exam questions as practice questions.**

- Split cases into train/test (70:30)
- Only use train set to tune description
- Test set is for final validation only, never for optimization
- New cases enter train set first; move to test set after one full iteration

→ Full detail: `references/eval-driven-development.md`

---

## Quick Checklist

Before building:
- [ ] All five design questions answered
- [ ] Three-layer directory structure planned
- [ ] Description contains all four required elements

Before going live:
- [ ] `evals/trigger-evals.json` has ≥ 10 cases
- [ ] Trigger rate ≥ 80% verified
- [ ] All three commissioning steps completed
- [ ] Capability registry updated

---

## Reference Documents

- Design principles → `references/design-principles.md`
- Description as trigger hook → `references/description-as-trigger.md`
- Evaluation-driven development → `references/eval-driven-development.md`
- Anti-patterns → `references/anti-patterns.md`
- Skill template → `assets/skill-template/SKILL.md`

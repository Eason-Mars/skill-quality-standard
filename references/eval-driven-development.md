# Evaluation-Driven Development

## The Principle

A Skill without a passing eval is not done — it's a hypothesis. The eval converts "I think this works" into "I have evidence this works."

Evaluation-driven development means: **write the test cases before or alongside the Skill, not after.**

---

## Three Evaluation Phases

### Phase 1 — Trigger Rate (Required for All Skills)

**Question**: Does the Skill get selected when it should?

**Test file**: `evals/trigger-evals.json`

**What to measure**:
- `should_trigger` test cases: what % actually triggered the Skill?
- `should_not_trigger` test cases: what % correctly did NOT trigger?

**Thresholds**:
| Score | Action |
|-------|--------|
| ≥ 80% | Safe to go live |
| 60–79% | Proceed with caution, monitor |
| < 60% | **Do not go live. Fix description first.** |

---

### Phase 2 — Quality Evaluation (Required for Content Skills)

**Question**: When the Skill does trigger, does it produce good output?

**Test file**: `evals/quality-cases.json`

**What to measure**:
- Does the output meet the defined expectations?
- Is the format correct?
- Does the content hit the key requirements?

**Scoring**: Each case is graded PASS/FAIL or 1–5. Target average ≥ 3.5/5.0.

---

### Phase 3 — Anti-Overfitting

**Question**: Does the Skill work on NEW inputs it has never seen?

**Key protocol**: Train/test split. See section below.

---

## trigger-evals.json Format

### Schema

```json
{
  "skill_name": "your-skill-name",
  "description": "Brief explanation of what this eval tests",
  "cases": [
    {
      "id": "trigger-001",
      "query": "I want to build a new skill for weekly reports",
      "expected": "should_trigger",
      "rationale": "Direct request to build a skill"
    },
    {
      "id": "no-trigger-001",
      "query": "translate this paragraph to Spanish",
      "expected": "should_not_trigger",
      "rationale": "Translation task, not skill development"
    }
  ]
}
```

### Field Definitions

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `skill_name` | string | Yes | Must match the `name` in SKILL.md frontmatter |
| `description` | string | Yes | Human-readable purpose of this eval file |
| `cases[].id` | string | Yes | Unique identifier, format: `trigger-NNN` or `no-trigger-NNN` |
| `cases[].query` | string | Yes | The exact message to test — write it as a user would say it |
| `cases[].expected` | enum | Yes | `"should_trigger"` or `"should_not_trigger"` |
| `cases[].rationale` | string | Yes | Why this case should/shouldn't trigger — helps debug failures |

### Writing Good Test Cases

**For should_trigger cases**:
- Use the language a real user would type, not technical documentation language
- Include both explicit requests ("build me a skill for X") and implicit ones ("how do I make my skill work better")
- Include variations in phrasing — the description needs to handle real linguistic diversity

**For should_not_trigger cases**:
- Include tasks that are superficially related but belong to other Skills
- Include tasks from completely different domains
- Aim for ~50/50 split between related-but-wrong and unrelated

---

## quality-cases.json Format

### Schema

```json
{
  "skill_name": "your-skill-name",
  "grader_instructions": "Score each case 1-5. 5=fully meets expectations, 1=completely misses.",
  "cases": [
    {
      "id": "quality-001",
      "input": {
        "query": "Help me build a skill that generates weekly summary reports",
        "context": "User is new to skill development"
      },
      "expected_output": {
        "contains": ["five design questions", "output artifact", "trigger phrases"],
        "format": "Structured workflow with numbered steps",
        "min_length": 500
      },
      "grader_notes": "Must cover Q1-Q5. Must mention description requirements."
    }
  ]
}
```

### When to Use quality-cases.json

Not every Skill needs quality evaluation. Use it when:
- The Skill produces content that will be published or shared
- Output quality is subjective and hard to auto-verify
- The Skill has multiple modes with different quality expectations

Skip it when:
- The Skill produces deterministic output (scripts, JSON, structured data)
- The output can be validated programmatically

---

## Train/Test Split Protocol

### The Problem

If you look at your test cases to fix your description, you're overfitting. Your Skill will score 100% on cases it has "seen" and fail on new inputs.

### The Protocol

**Step 1: Create 20 cases minimum**
- 10 should_trigger
- 10 should_not_trigger

**Step 2: Split 70:30**
- 14 cases → `train` set (for tuning)
- 6 cases → `test` set (for final validation only)

Mark them in the JSON:
```json
{
  "id": "trigger-001",
  "query": "...",
  "expected": "should_trigger",
  "split": "train"
}
```

**Step 3: Use ONLY train set during development**
- Run trigger-bench with `--split train`
- Fix description based on train failures
- Never look at test cases while iterating

**Step 4: Final validation with test set**
- Only run once you're satisfied with train performance
- If test performance is significantly lower than train, you've overfit
- Do not go back and fix for test cases — that defeats the purpose

**Step 5: New cases always enter train first**
- Add new failure cases to train set
- Move a case to test only after one full iteration has passed without it being used for tuning

---

## Real-World Case: lesson-keeper

### Background

`lesson-keeper` is an OpenClaw skill for capturing lessons and insights from development work. It was the first skill developed using trigger-bench as a validation tool.

### Development Timeline

**Iteration 1 (first draft)**
- Description: Generic, no trigger phrases, no exclusions
- Trigger rate: ~45% on should-trigger cases
- Action: Complete rewrite of description

**Iteration 2 (added trigger phrases)**
- Added 8 verbatim trigger phrases to description
- Added "Use when" scenarios
- Trigger rate: 75% on should-trigger cases
- False positive rate: 30% (too many wrong triggers)

**Iteration 3 (added exclusions, refined scope)**
- Added NOT-for section
- Narrowed scope of should-trigger phrases
- Trigger rate: 95% on should-trigger cases
- False positive rate: 5%

### Final Results

- **17/17 assertions passed** (quality evaluation)
- **20/20 trigger test cases passed** (trigger-bench)
- 3 iterations from first draft to shipping

### Key Lessons

1. **Generic descriptions fail**. The first draft had zero specificity and hit 45%.
2. **False positives are as bad as false negatives**. After fixing recall, had to fix precision too.
3. **The NOT-for section is not optional**. It cut false positives from 30% to 5%.
4. **3 iterations is typical**. Budget for at least 2–3 rounds of description refinement.

---

## Running the Full Evaluation Pipeline

```bash
# Step 1: Run trigger eval on train set
python3 ~/.openclaw/workspace/.agents/skills/trigger-bench/scripts/run_eval_openclaw.py \
  --eval-set evals/trigger-evals.json \
  --skill-path ~/.openclaw/workspace/.agents/skills/your-skill \
  --split train \
  --model claude-sonnet-4-5 \
  --runs-per-query 3 \
  --verbose

# Step 2: Review failures, update description, re-run
# Repeat until train trigger rate ≥ 80%

# Step 3: Final validation on test set (once only)
python3 ~/.openclaw/workspace/.agents/skills/trigger-bench/scripts/run_eval_openclaw.py \
  --eval-set evals/trigger-evals.json \
  --skill-path ~/.openclaw/workspace/.agents/skills/your-skill \
  --split test \
  --model claude-sonnet-4-5 \
  --runs-per-query 3

# Step 4: If test ≥ 80%, proceed to commissioning
# If test < 70%, investigate overfitting
```

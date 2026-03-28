# skill-quality-standard

> The OpenClaw-native practice guide for skill development.
> OpenClaw Skill 开发实践手册——经过实战验证的标准，不是理论框架。

---

## What this is / 这是什么

**English**: A battle-tested practice guide for building high-quality OpenClaw skills. This covers everything from the five design questions you must answer before building, to trigger rate evaluation with trigger-bench, to the three-step commissioning verification that proves a skill actually works.

**中文**: 专为 OpenClaw 环境设计的 Skill 开发实践手册。涵盖动手前必须回答的五设计问题、用 trigger-bench 验证触发率、以及证明 Skill 真正可用的 Commissioning 三步验证。

---

## Why not just skill-creator / 为什么不用 skill-creator

| | skill-creator | skill-quality-standard |
|--|---------------|------------------------|
| **Scope** | General framework for any skill system | OpenClaw-native practice guide |
| **Commissioning** | Generic "test your skill" guidance | Three-step iron-clad verification with evidence requirements |
| **Paths** | Generic path examples | Workspace-relative path rules (`~` is forbidden) |
| **Evaluation** | Conceptual evaluation guidance | Integrated with trigger-bench, with train/test split protocol |
| **Validation** | Theoretical | Validated in production: lesson-keeper 20/20 trigger tests |

**The core difference**: skill-creator tells you what to think about. skill-quality-standard tells you exactly what to do in OpenClaw, based on what has actually worked and failed in real development.

Specific OpenClaw rules not in skill-creator:
- The three-step commissioning verification (Step 3 requires evidence, not just "should work")
- Workspace-relative paths (absolute, no `~` shortcuts)
- trigger-bench integration with `run_eval_openclaw.py`
- The declaration ≠ integration lesson (24 scripts listed as integrated, 0 actually modified)

---

## The Five Design Questions / 五设计问题

Before writing a single line of code or documentation, answer all five:

| # | Question | If you can't answer | What it catches |
|---|----------|---------------------|-----------------|
| Q1 | Can a single LLM call handle this? | Don't build yet | Unnecessary skills |
| Q2 | What is the single output artifact? | Define output first | Vague, unused skills |
| Q3 | Is there a loop? How is it designed? | Design the loop explicitly | Runaway loops |
| Q4 | What's the worst-case hallucination? | Add confirmation steps | Irreversible errors |
| Q5 | Where does output live? How does downstream find it? | Define the path convention | Skills that produce orphaned output |

→ Full detail with examples: `references/design-principles.md`

---

## Installation / 安装

```bash
# Via clawhub (recommended)
npx clawhub@latest install skill-quality-standard

# Via git
git clone https://github.com/Eason-Mars/skill-quality-standard \
  ~/.openclaw/workspace/.agents/skills/skill-quality-standard
```

---

## Quick Start / 快速上手

**Building a new skill:**
1. Read `SKILL.md` — answer Q1–Q5 before touching any files
2. Create directory structure: `scripts/` `references/` `assets/` `evals/`
3. Write description with trigger phrases (see Module 3)
4. Add `evals/trigger-evals.json` with ≥ 10 test cases
5. Run trigger-bench, achieve ≥ 80% trigger rate
6. Complete all three commissioning steps
7. Register in capabilities

**Debugging a skill that isn't triggering:**
1. Check description — does it contain quoted trigger phrases?
2. Run trigger-bench to measure current trigger rate
3. Follow the fix strategy in `references/description-as-trigger.md`

**Template:** Copy `assets/skill-template/SKILL.md` as your starting point.

---

## Real-World Validation / 实战验证

This guide is derived from building `lesson-keeper`, the first skill developed with trigger-bench as a validation tool.

**lesson-keeper development timeline:**
- Iteration 1: Generic description → 45% trigger rate
- Iteration 2: Added trigger phrases → 75% trigger rate, but 30% false positives
- Iteration 3: Added NOT-for exclusions → **95% trigger rate, 5% false positives**

**Final results:**
- 17/17 quality assertions passed ✅
- 20/20 trigger test cases passed ✅
- 3 iterations from first draft to production

**trigger-bench first use case**: lesson-keeper was the first skill validated end-to-end with trigger-bench's OpenClaw-compatible runner.

---

## Structure / 目录结构

```
skill-quality-standard/
├── SKILL.md                              ← Load this when building skills
├── README.md                             ← This file
├── references/
│   ├── design-principles.md              ← Five design questions, detailed
│   ├── description-as-trigger.md         ← Trigger phrase writing guide
│   ├── eval-driven-development.md        ← trigger-bench integration
│   └── anti-patterns.md                  ← Six common mistakes to avoid
├── assets/
│   └── skill-template/
│       └── SKILL.md                      ← Copy this to start a new skill
└── evals/
    └── trigger-evals.json                ← 20 test cases for this skill itself
```

---

## Credits / 致谢

Inspired by [skill-creator](https://github.com/steipete/skill-creator) by [@steipete](https://github.com/steipete).

skill-quality-standard takes skill-creator's framework and adds OpenClaw-specific battle-tested standards from real skill development in production.

---

## License

MIT

# Design Principles: Five Questions Before You Build

## Why Five Questions?

Most failed Skills share one root cause: they were built before the builder knew what "done" looked like. The five design questions force clarity before a single file is created. Spending 10 minutes on these saves hours of rework.

---

## Q1: Can a Single LLM Call Handle This?

**The test**: Give an expert user your exact task description and tell them they have only one LLM prompt. Can they get a good result?

If **yes** → Don't build a Skill. Write a prompt template instead. Examples of tasks that don't need a Skill:
- "Summarize this article in 3 bullet points" — any good model does this natively
- "Translate this paragraph to French" — no special knowledge required
- "Write a subject line for this email" — LLM has sufficient context

If **no** → A Skill is justified. You need a Skill when:
- The task requires running scripts or external tools
- The task requires loading specific reference documents (style guides, templates, data schemas)
- The task has a multi-step workflow where order matters
- The task requires deterministic operations (calculations, file I/O, API calls)
- The task needs specific output formats the LLM wouldn't produce naturally

**Common mistake**: Building a Skill because it "feels complex" rather than because it genuinely requires capabilities beyond a single LLM call.

---

## Q2: What Is the Single Output Artifact?

Every Skill must produce exactly one primary output artifact. If you can't define it, the Skill has no clear purpose.

### The Three-Part Definition

| Dimension | Question | Example |
|-----------|----------|---------|
| File type | What format? | `.html` report |
| Path template | Where exactly? | `output/{date}-research-{ticker}.html` |
| Consumer | Who/what reads it? | Human reviewer in browser |

### Why Path Templates Matter

Vague: "Save the report somewhere"
Specific: `~/.openclaw/workspace/output/{YYYY-MM-DD}-{skill-name}-{project}.md`

A path template answers three questions at once:
1. Where will the file be? (Never "wherever")
2. How will downstream workflows find it? (By convention, not search)
3. Will files from different runs collide? (Date prefix prevents this)

### When Output Is Ambiguous

If you're tempted to say "it depends what the user wants," that's a sign to either:
- Narrow the Skill scope (one Skill, one output type)
- Define variants explicitly (mode A → `.md`, mode B → `.html`)

**Never leave output type as a runtime decision the LLM makes spontaneously.**

---

## Q3: Is There a Loop? How Is It Designed?

### Three Patterns

**Pattern 1: No loop (single-execution)**
Most Skills. The Skill runs once, produces output, ends.
- Explicitly state: "This Skill runs once and exits."
- Example: `baoyu-translate` — translate → save → done

**Pattern 2: Human-in-the-loop**
The Skill produces a draft, waits for human feedback, iterates.
- Use soft exit: "Review the output above. Say 'refine: [feedback]' to iterate, or 'done' to finalize."
- Set max iterations: never loop more than 5 times without explicit human re-trigger
- Example: `canvas-design` — generate → present → user says "make it darker" → iterate

**Pattern 3: Automated loop**
The Skill calls itself or calls another Skill in a pipeline.
- Use hard ceiling: `MAX_ITERATIONS = 5` (hardcoded, not configurable)
- Define exit condition explicitly: "Stop when quality score ≥ 4.0 or iterations = 5"
- Log each iteration result for debugging
- Example: `trigger-bench` — test → analyze failures → rewrite description → re-test (max 3 rounds)

### Runaway Loop Protection

Any automated loop without a hard ceiling is a liability. A looping Skill that hits an unexpected state will keep running until:
- The context window fills up
- API costs explode
- The user force-quits

Always add: `if iterations >= MAX_ITERATIONS: break`

---

## Q4: What's the Worst-Case Hallucination?

This is a risk assessment question. For each stage of your Skill's workflow, ask: "If the LLM gets this completely wrong, what happens?"

### Risk Matrix

| Stage Type | Hallucination Risk | Mitigation |
|------------|-------------------|------------|
| Drafting text | Low — human reviews before use | Allow LLM autonomy |
| Summarizing / extracting | Medium — errors persist silently | Show LLM's work, make it citeable |
| Writing to files | High — wrong content becomes canonical | Preview before write, confirm step |
| Calling APIs / making requests | High — irreversible side effects | Always preview, require explicit "confirm" |
| Generating code that runs | Very high — execution is irreversible | Code review step before exec |

### Design Implications

**For low-risk stages**: Let the LLM run freely. Human will catch errors.

**For high-risk stages**, always add:
```
Before writing the file, show me the content and ask: 
"Ready to write to [path]? (yes/no)"
Only proceed if user confirms.
```

**The key question**: "If this stage produces garbage, can the user recover?"
- Yes, easily → Low risk
- Yes, but annoying → Medium risk
- No, or very painful → High risk

---

## Q5: Where Does Output Live? How Does Downstream Find It?

### The Discovery Problem

A Skill that produces output only it knows how to find is a dead end. Define discovery upfront.

### Three Discovery Mechanisms

**1. Path Convention (Recommended)**
Files always land in a predictable location with a predictable name.
```
output/YYYY-MM-DD-{skill}-{id}.md
```
Any downstream skill can `ls output/` and find what it needs.

**2. Manifest File**
The Skill writes a small JSON index after each run:
```json
{ "latest": "output/2025-01-15-research-AAPL.html", "created_at": "2025-01-15T09:30:00" }
```
Downstream reads the manifest to find the latest output.

**3. Explicit Handoff**
The Skill explicitly tells the user (or calling Skill) the exact path of what it produced.
```
✅ Report saved to: output/2025-01-15-analysis.html
```

### When Two Skills Chain Together

If Skill A feeds Skill B:
- Define the handoff format (not just "Skill A produces something Skill B reads")
- Agree on the path convention before building either Skill
- Test the handoff explicitly in commissioning Step 3

---

## When NOT to Build a Skill

Save time — don't build if:

1. **The task is rare** — If you'll use it fewer than 5 times, write a one-time script instead
2. **The task has no standard output** — If every run produces something structurally different, a Skill can't enforce quality
3. **The LLM already does it well** — Test with a plain prompt first. If 90% of the time it's good enough, no Skill needed
4. **You can't write the trigger-evals.json** — If you can't write 5 clear "should trigger" messages, the Skill's scope is undefined
5. **You're just wrapping a single API call** — Use a script, not a Skill

---

## Checklist for Q1–Q5

```
Q1: [ ] Confirmed: plain LLM call is not sufficient for this task
Q2: [ ] Output file type: _______
    [ ] Output path template: _______
    [ ] Output consumer: _______
Q3: [ ] Loop type: none / human-in-loop / automated
    [ ] If loop: exit condition and hard ceiling defined
Q4: [ ] High-risk stages identified: _______
    [ ] Confirmation step added for each high-risk stage
Q5: [ ] Discovery mechanism: path convention / manifest / explicit handoff
    [ ] Downstream Skill handoff format defined (if applicable)
```

# Description as Trigger Hook

## The Core Insight

The description field is not documentation — it's the recall mechanism. When a user sends a message, the system scans all available skill descriptions to decide which one to load. A description that doesn't contain the right words will never trigger, no matter how good the Skill's content is.

**Recall = 0% if your description doesn't contain the words users actually say.**

---

## Four Required Elements

Every skill description must contain all four:

### Element 1: What It Does (One-Line Core Function)
The clearest possible statement of the primary action. Not a mission statement — a verb phrase.

✅ `When the user wants to build a new OpenClaw skill from scratch`
❌ `A comprehensive framework for skill development best practices`

### Element 2: Trigger Phrases (Actual Words Users Say)
Quote the exact phrases users type. These are not categories — they are verbatim fragments.

✅ `Use when: "build a new skill", "skill not triggering", "how to write description"`
❌ `Use when skill development is needed`

### Element 3: Explicit Use-When Scenarios
Enumerate the concrete situations that should trigger the Skill. Make it easy for the routing system to match.

✅ `Use when the user asks about commissioning, three-layer architecture, or trigger-bench integration`
❌ `Use when appropriate`

### Element 4: NOT-For Exclusions
Explicitly list what this Skill does NOT cover. This prevents mis-triggering when a related but different Skill should be used instead.

✅ `NOT for: using an existing skill to solve a business problem`
❌ (no exclusion section — system may trigger this Skill for unrelated tasks)

---

## 10 Good vs. Bad Description Examples

### Example 1 — Blog Publishing

**BAD:**
```
Handles the process of publishing content to Ghost blog platform.
```
Problems: No trigger phrases. "Handles" is vague. No exclusions.

**GOOD:**
```
Use when publishing articles to Ghost blog, updating existing posts, or managing drafts.
Trigger phrases: "publish to Ghost", "blog post draft", "update Ghost article",
"ghost-blog skill", "post to my blog". NOT for: creating blog content (see copywriting skill).
```

---

### Example 2 — Stock Research

**BAD:**
```
Provides financial analysis and market research capabilities for investment decisions.
```
Problems: "Provides" is passive. No specific ticker/market phrases. Too broad.

**GOOD:**
```
When the user wants A-share stock analysis, research reports, premarket scans, or
portfolio tracking. Use when: "analyze [ticker]", "premarket report", "research report",
"stock screener", "morning scan". NOT for: US stocks, crypto, or general financial advice.
```

---

### Example 3 — Translation

**BAD:**
```
Translation skill for converting text between languages.
```
Problems: No language pairs specified. No trigger phrases. No context about what gets translated.

**GOOD:**
```
Use when translating articles, documents, or long-form content between languages,
especially Chinese-English. Trigger phrases: "translate this", "翻译", "translate to Chinese",
"translate to English", "localize". NOT for: single-word lookups or grammar correction.
```

---

### Example 4 — Image Generation

**BAD:**
```
Generates images based on user prompts using AI models.
```
Problems: Too generic. Doesn't distinguish from other image tools. No trigger phrases.

**GOOD:**
```
Use when generating images from text descriptions, creating cover images, article illustrations,
or batch image production. Use when: "generate image", "create an image", "draw",
"image for my article", "cover image". Supports multiple backends (OpenAI/Gemini/Replicate).
NOT for: editing existing images or image compression.
```

---

### Example 5 — Meeting Notes

**BAD:**
```
Processes meeting-related content and produces structured outputs.
```
Problems: "Processes" and "structured outputs" say nothing. Would never match user intent.

**GOOD:**
```
Use when creating meeting notes, summarizing a call, extracting action items from a transcript,
or formatting discussion outcomes. Trigger phrases: "meeting notes", "call summary",
"action items from", "summarize our discussion", "recap the meeting".
NOT for: scheduling meetings or sending invites.
```

---

### Example 6 — SEO Audit

**BAD:**
```
SEO optimization tool for improving website content and search rankings.
```
Problems: Too broad, overlaps with 20 other skills. No specifics.

**GOOD:**
```
When the user wants to audit on-page SEO, fix technical SEO issues, or improve page rankings.
Use when: "SEO audit", "why isn't my page ranking", "fix my SEO", "on-page optimization",
"meta description", "title tag". NOT for: content writing (see seo-content-writer),
backlink analysis (see backlink-analyzer).
```

---

### Example 7 — Email Sequence

**BAD:**
```
Creates email sequences for marketing and communication purposes.
```
Problems: "Marketing and communication" is too broad. No user language.

**GOOD:**
```
When the user wants to write a drip campaign, onboarding sequence, welcome series,
or automated email flow. Use when: "email sequence", "drip campaign", "welcome emails",
"onboarding emails", "what emails should I send". NOT for: cold outreach (see cold-email).
```

---

### Example 8 — PDF Generation

**BAD:**
```
Converts content to PDF format.
```
Problems: A single sentence. Would trigger for every PDF-related task indiscriminately.

**GOOD:**
```
Use when exporting a final report, document, or article to PDF with proper formatting.
Trigger phrases: "export to PDF", "save as PDF", "generate PDF", "PDF version",
"print-ready". NOT for: reading or extracting content from existing PDFs.
```

---

### Example 9 — Competitor Analysis

**BAD:**
```
Analyzes competitors and provides competitive intelligence.
```
Problems: Says nothing specific. Competitor analysis of what? For what purpose?

**GOOD:**
```
When the user wants to research a competitor's SEO strategy, content approach, or keyword
rankings. Use when: "competitor analysis", "what is [competitor] doing", "why do they
rank higher", "spy on competitor SEO", "competitive intelligence". NOT for: pricing comparison
or feature comparison (see competitor-alternatives).
```

---

### Example 10 — Skill Builder (This Skill)

**BAD:**
```
Helps with building and developing OpenClaw skills.
```
Problems: Too vague. "Helps with" is weak. No specific trigger scenarios.

**GOOD:**
```
Use when building a new OpenClaw skill from scratch, debugging why a skill isn't triggering,
writing or improving a skill description, evaluating skill quality, or following best practices
for skill development. Use when: "build a new skill", "skill not triggering",
"how to write description", "skill quality", "commissioning". NOT for: using an existing skill.
```

---

## Trigger Rate Below 60%: Fix Strategy

If your trigger rate (should-trigger test cases that actually triggered) is below 60%, follow this diagnostic:

### Step 1: Collect Failure Cases
Run trigger-bench and collect the specific messages that failed to trigger. Don't guess — look at the actual failures.

```bash
python3 ~/.openclaw/workspace/.agents/skills/trigger-bench/scripts/run_eval_openclaw.py \
  --eval-set evals/trigger-evals.json \
  --skill-path ~/.openclaw/workspace/.agents/skills/your-skill \
  --model claude-sonnet-4-5 \
  --runs-per-query 3 \
  --verbose
```

### Step 2: Diagnose the Pattern

| Failure Pattern | Root Cause | Fix |
|-----------------|------------|-----|
| Technical phrasing triggers, casual doesn't | Description too jargon-heavy | Add casual user language |
| Direct requests trigger, implicit ones don't | No "Use when" scenarios | Add concrete scenario descriptions |
| Scope too narrow (misses related tasks) | Description is too specific | Broaden to include adjacent triggers |
| Wrong Skill triggering instead | No exclusions, overlap with sibling Skill | Add NOT-for section |
| Nothing triggers at all | Description too abstract, no phrases | Rewrite with verbatim user quotes |

### Step 3: Rewrite Strategy

**Add verbatim phrases**: Look at the failing test messages. Copy key phrases directly into the description's trigger section.

**Add "Use when" scenarios**: Write out 3–5 concrete situations in plain language. The routing system matches on these scenarios.

**Add exclusions**: If a sibling Skill keeps triggering instead, explicitly carve out the boundary in both descriptions.

**Make it longer**: Short descriptions (< 50 words) almost always have low recall. The description is not taking up context window — it's a routing signal.

### Step 4: Re-test Immediately

After every description change, re-run trigger-bench before doing anything else. Don't stack multiple changes without measuring.

Target thresholds:
- ≥ 80% should-trigger rate → safe to go live
- 60%–79% → proceed with caution, monitor closely
- < 60% → do not go live, mandatory fix

---

## Integration with trigger-bench

### Setup

```bash
# trigger-bench must be installed
ls ~/.openclaw/workspace/.agents/skills/trigger-bench/

# Your eval file
cat evals/trigger-evals.json
```

### Running a Full Evaluation

```bash
python3 ~/.openclaw/workspace/.agents/skills/trigger-bench/scripts/run_eval_openclaw.py \
  --eval-set evals/trigger-evals.json \
  --skill-path ~/.openclaw/workspace/.agents/skills/your-skill \
  --model claude-sonnet-4-5 \
  --runs-per-query 3 \
  --verbose
```

### Interpreting Results

The output shows:
- Per-case pass/fail
- Aggregate trigger rate for should-trigger cases
- False positive rate for should-not-trigger cases
- Which specific cases failed (use these to fix description)

### Golden Rule

**Only use test cases to validate, never to optimize.** If you look at a failing test case and add its exact wording to your description, you're overfitting. Add the *general pattern* to your description, not the literal test phrase.

See `references/eval-driven-development.md` for the train/test split protocol.

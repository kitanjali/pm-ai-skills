---
name: write-evals
description: Turn any LLM-feature description into a first-draft eval suite — test cases, edge cases, refusal cases, a grading method, and the gaps only a domain expert can fill. Domain-agnostic.
---

# Write Evals

Take a description of any AI/LLM feature and produce a structured eval suite: the dataset, the expected behavior, the grading method, and an honest list of what can't be verified without a human.

An eval is a test for non-deterministic software. The same input can produce different outputs, so `assert equals` doesn't work. An eval answers two questions: "Is this feature good enough to ship?" and "Did my last change make it better or worse?"

This is a meta-tool. It is NOT domain-specific. The domain knowledge comes from the input. The same skill works for a document classifier, a support-reply bot, a resume parser, or a medical-summary tool.

## When to use

- Before shipping any LLM feature past a prototype
- Before or after changing a prompt, model, or retrieval setup (regression check)
- When comparing two prompts or two models objectively
- When a stakeholder asks "how do you know it's good enough?"

## Input

Ask the user for:
1. **The feature** — what it does, in one or two sentences
2. **Input shape** — what gets fed in (a product description, a support ticket, a document)
3. **Output shape** — what a correct response looks like (a label, a JSON object, a paragraph, a refusal)
4. **Stakes** — what a wrong answer costs. This drives which edge cases matter.
5. **Source of truth** (if any) — docs, rules, or a spec the answer must conform to
6. **Any real examples** the user already has (input + known-correct output)

If any of these are missing, ask before guessing. Never invent expected answers for a specialized domain.

## Process

1. Restate the feature in one line and confirm understanding before generating.
2. Define **success criteria** — what "good" means for this feature, in plain terms, before writing any cases.
3. Generate the eval dataset across these categories (aim for coverage, not volume):
   - **Happy path** — typical, unambiguous inputs
   - **Edge cases** — boundary conditions, ties, ambiguity, rare-but-valid inputs
   - **Adversarial** — malformed, misleading, prompt-injection, out-of-distribution
   - **Refusal / escalation** — inputs where the correct behavior is "not enough information" or "escalate to a human," NOT a confident guess
   - **Regression anchors** — cases tied to past bugs, so they can't silently return
4. For each case, specify **expected behavior**: an exact answer where one exists, or a rubric ("must cite a source," "must refuse if data is missing," "must not fabricate a value").
5. Recommend a **grading method** per case:
   - **Exact / programmatic** — deterministic checks (label match, JSON schema valid, regex)
   - **Rule-based** — checks a property ("contains a citation," "confidence < 0.5 → escalates")
   - **LLM-as-judge** — a model grades against the rubric; specify the judge prompt and its own failure risk
   - Prefer the cheapest reliable method. Reserve LLM-judge for genuinely open-ended output.
6. Define **pass/fail and the ship bar** as numbers (e.g., "95% on happy path, 100% on refusal cases, no regression"). Note which failures are acceptable and why.
7. **Flag the gaps honestly.** For anything requiring domain truth that can't be verified here, output `⚠️ NEEDS EXPERT: <what to confirm>` instead of a fabricated answer. An eval with confidently wrong expected answers is worse than no eval.

## Output

Present the suite in this format (offer to save it to a file):

```markdown
# Eval Suite — [Feature Name]

**Feature:** [one line]
**Stakes of a wrong answer:** [what it costs]
**Ship bar:** [e.g., 95% happy path, 100% refusal, no regression]

## Success criteria
[What "good" means, in plain terms]

## Test cases

| # | Category | Input | Expected behavior | Grading method |
|---|----------|-------|-------------------|----------------|
| 1 | Happy path | ... | ... | Exact |
| 2 | Edge | ... | ... | Rule |
| 3 | Adversarial | ... | ... | Rule |
| 4 | Refusal | ... | "Insufficient info, escalate" | Rule |
| 5 | Regression | ... | ... | Exact |

## Grading notes
- [Any LLM-judge prompt + its failure risk]
- [Any programmatic check logic]

## Gaps (needs a human / domain expert)
- ⚠️ NEEDS EXPERT: [what to confirm and why]

## How to run this
[Plain steps: run each input, grade the output, compute the score against the ship bar]
```

## Worked example (product classification)

- **Feature:** classify a product description into a category code with a confidence score.
- **Happy path:** "men's cotton knit t-shirt, 100% cotton, short sleeve" → the apparel code, high confidence.
- **Edge:** "60% cotton, 40% polyester knit shirt" → tests the material-dominance tie-break rule instead of guessing.
- **Adversarial:** a description with a misleading brand name that implies the wrong category.
- **Refusal:** "blue thing, medium" → correct behavior is "not enough information, escalate to a human," not a confident wrong code.
- **Grading:** exact-match on the code for the happy path; a rule-based check that low confidence triggers escalation on the refusal case.
- **Gap:** `⚠️ NEEDS EXPERT: confirm the correct code for the mixed-material case against the current classification schedule.`

## Rules

- Never fabricate an expected answer in a specialized domain. Flag it.
- Prefer the cheapest reliable grader. Don't reach for LLM-judge when a regex works.
- Refusal cases are not optional. Every high-stakes feature needs them.
- State the ship bar as a number, not "good enough."

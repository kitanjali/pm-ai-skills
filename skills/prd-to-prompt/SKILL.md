---
name: prd-to-prompt
description: Turn a PRD or feature spec into a structured system-prompt spec plus the eval that checks it. Bridges product intent to something testable.
---

# PRD to Prompt

Take a PRD or feature spec for an AI feature and produce two things: a structured system-prompt specification the engineering team can build against, and the eval that proves it does what the PRD intended. The point is to close the gap between "here's the doc" and "here's something we can test."

A PRD says what the feature should do. A prompt says how the model should behave. They are not the same document, and translating between them is product work, not a hand-off. This skill does the translation and keeps them honest against each other.

## When to use

- After a PRD is drafted for an LLM-backed feature, before engineering starts
- When a feature "works in the demo" but has no written spec for the model's behavior
- When the prompt and the PRD have drifted and nobody's sure which is right

## Input

Ask the user for:
1. **The PRD or spec** — paste it, or the relevant section
2. **The model's job** — the single task the model performs in this feature
3. **Inputs available at runtime** — what the model will actually receive (user text, retrieved context, tool outputs)
4. **Hard constraints** — anything the model must always or never do (never fabricate a price, always cite a source, refuse out-of-scope requests)
5. **Output contract** — the exact shape the rest of the system expects (freeform, JSON schema, a fixed set of labels)

If the PRD doesn't state a constraint or an output contract, flag it as a gap rather than inventing one.

## Process

1. Extract from the PRD, in the PRD's own priority order:
   - The **one job** the model does
   - The **success criteria** (what a good output looks like)
   - The **explicit constraints** (musts and must-nots)
   - The **implicit constraints** (things the PRD assumes but doesn't say, e.g. "obviously it shouldn't leak other users' data")
2. Draft the **system-prompt spec** with these sections:
   - **Role** — what the model is, in one line
   - **Task** — the job, stated imperatively
   - **Inputs** — what it receives and how each is delimited
   - **Rules** — the musts and must-nots, most important first
   - **Output format** — the exact contract, with a concrete example
   - **Refusal / fallback** — what to do when it can't comply (missing data, out of scope, low confidence)
3. Identify the **failure modes** this prompt is exposed to (ambiguous input, injection via user content, over-confident guessing) and note how the spec handles each.
4. Generate a **starter eval** tied directly to the PRD's success criteria and constraints. Each hard constraint becomes at least one test case, including a refusal case for the fallback path. (If a dedicated eval skill is available, hand off to it; otherwise produce the case table inline.)
5. Report **drift**: anything in the PRD the prompt can't satisfy, and anything the prompt does that the PRD never asked for.

## Output

Present three sections (offer to save):

```markdown
# Prompt Spec — [Feature Name]

## System-prompt spec
Role: ...
Task: ...
Inputs: ...
Rules (most important first):
1. ...
Output format: ... (with example)
Refusal / fallback: ...

## Failure modes covered
- [failure mode] → [how the spec handles it]

## Starter eval (maps to PRD)
| # | PRD requirement | Test input | Expected behavior | Grading |
|---|-----------------|-----------|-------------------|---------|
| 1 | ... | ... | ... | ... |

## Drift / open questions
- PRD asks for X but the prompt can't guarantee it because ...
- Prompt does Y, which the PRD never specified. Confirm intended?
```

## Worked example (support-reply drafting)

- **PRD line:** "Draft a reply to an inbound support ticket using only the customer's order data and our help-center articles."
- **Rule extracted:** must not invent policy not present in the provided articles → becomes a prompt rule AND an eval case (feed a question whose answer isn't in the articles; correct behavior is to escalate, not to guess).
- **Output contract:** a draft reply plus a `needs_human` boolean.
- **Drift flagged:** the PRD never says what to do when order data is missing. Open question surfaced instead of a silent default.

## Rules

- The prompt spec serves the PRD, not the other way around. If they conflict, flag it, don't silently pick one.
- Every hard constraint in the PRD must show up as a rule in the prompt AND a case in the eval. No orphans.
- A fallback/refusal path is mandatory. A feature with no defined "I can't do this" behavior is not spec-complete.
- Don't invent constraints the PRD doesn't state. Surface the gap.

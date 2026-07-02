# pm-ai-skills

A small, focused set of [Claude Code](https://claude.com/claude-code) skills for the product work of building and shipping AI features. Not a generic prompt pack. These cover the parts most PMs don't have a repeatable process for yet: writing evals, turning a spec into a testable prompt, and red-teaming a feature before it ships.

Each skill is self-contained, domain-agnostic, and ships with a worked example. You bring the feature, the skill brings the structure.

## The skills

| Skill | What it does |
|-------|--------------|
| **`/write-evals`** | Turns any LLM-feature description into a first-draft eval suite: test cases, edge cases, refusal cases, a grading method, and an honest list of the gaps only a domain expert can fill. |
| **`/prd-to-prompt`** | Turns a PRD or feature spec into a structured system-prompt spec plus the eval that checks it. The AI product lifecycle in one pass. |
| **`/red-team-feature`** | Adversarially attacks a feature or prompt for hallucination, jailbreak, data leakage, and quiet failure modes before launch, and proposes mitigations. |

## Why these three

Raw accuracy demos are easy. Knowing whether an AI feature is safe to ship is the hard part, and it's mostly PM work: defining what "good" means, deciding when the model is allowed to act on its own, and proving it fails in the safe direction. These skills make that repeatable.

- `write-evals` answers "how do we know it's good enough?"
- `prd-to-prompt` answers "how do we get from a doc to something testable?"
- `red-team-feature` answers "how does this break, and who gets hurt when it does?"

## Used in the wild

These aren't theoretical. The eval approach in `write-evals` and the trust-boundary thinking in `red-team-feature` came out of building [**trade-classify**](https://github.com/kitanjali/trade-classify), an LLM HS-code classifier for cross-border commerce that grounds every code in real tariff data, routes high-stakes goods to human review, and reports automation precision instead of raw accuracy. If you want to see these ideas applied to a real feature, start there.

## Install

A Claude Code skill is just a folder with a `SKILL.md` inside it, placed where Claude Code looks for skills.

**Per project** (available in one repo):

```bash
git clone https://github.com/kitanjali/pm-ai-skills.git
cp -r pm-ai-skills/skills/* /path/to/your-project/.claude/skills/
```

**Global** (available everywhere on your machine):

```bash
cp -r pm-ai-skills/skills/* ~/.claude/skills/
```

Then, in Claude Code, invoke by name:

```
/write-evals
/prd-to-prompt
/red-team-feature
```

Each will ask you for the few inputs it needs.

## Design principles

Every skill in here follows the same rules:

- **Self-contained.** No reference to any private files or house context. The input comes from you.
- **Honest about limits.** In a specialized domain, a skill flags what it can't verify (`NEEDS EXPERT`) instead of inventing an answer. A confidently wrong expected-answer is worse than none.
- **Cheapest reliable method wins.** Don't reach for an LLM-as-judge when a regex or exact-match will do.
- **Refusal is a first-class outcome.** For any high-stakes feature, "I don't have enough information, escalate to a human" is a correct answer the eval must test for.

## Contributing

Issues and PRs welcome. A good new skill is: domain-agnostic, self-contained, ships a worked example, and covers a real gap in AI product work (not a generic PM task that a dozen packs already cover).

## License

MIT. See [LICENSE](LICENSE).

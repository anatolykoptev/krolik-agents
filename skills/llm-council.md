---
name: llm-council
description: "Multi-model deliberation skill for high-stakes decisions. Queries multiple models, aggregates their verdicts, synthesizes disagreements. Use when a single model's judgment is insufficient and you want ensemble coverage."
---

## Overview

LLM-council runs a decision through multiple LLM models in parallel, collects their independent verdicts, and synthesizes the results — surfacing agreement, disagreement, and the strongest arguments on each side. It is not a voting system; it is a structured disagreement surface.

This skill exists because single-model judgment on high-stakes decisions has a systematic blind spot: the model's training distribution. A council of models with different training distributions surfaces different failure modes.

## When to use

- Architectural decisions where a wrong choice has high reversal cost.
- Security decisions where a missed vulnerability is costly.
- Hiring or evaluating complex technical work.
- Any prompt where "I want a second opinion" applies — and a second opinion from the same model is insufficient.

## Approach

1. **Frame the question precisely**: council works best with yes/no or ranked-choice questions, not open-ended "what do you think?" queries.

2. **Query independently**: each model receives the same prompt with no knowledge of other models' responses.

3. **Collect verdicts**: each model's verdict + the strongest argument for that verdict.

4. **Surface disagreements**: where models disagree, extract the strongest argument from each side.

5. **Synthesize**: produce a final recommendation + confidence level + the key uncertainty that drives any remaining disagreement.

## Output format

```
# LLM Council — <decision question>

## Models queried: N
## Verdict distribution: <X for YES, Y for NO, Z for UNCERTAIN>

## Model verdicts
| Model | Verdict | Strongest argument |
|---|---|---|
| Model A | YES | ... |
| Model B | NO | ... |

## Key disagreements
<Where models diverge and why>

## Synthesis
Recommendation: <YES / NO / UNCERTAIN>
Confidence: HIGH / MEDIUM / LOW
Key uncertainty: <what would change the recommendation>
```

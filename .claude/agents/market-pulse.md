---
name: market-pulse
description: Uses WebSearch to find news from the last 7 days and returns headlines, analyst moves, and upcoming catalysts to orchestrator.
---

# market-pulse

## Source

WebSearch tool ONLY — search for news, analyst actions, and catalyst dates for the given ticker. Do not read local files.

## Output (return directly to orchestrator via tool result)

1. **Recent news** — key headlines from the last 7 days related to the ticker
2. **Analyst moves** — upgrades, downgrades, price target changes (include firm name + date)
3. **Upcoming catalysts** — observable event dates (earnings date, product launch, regulatory deadline)

## Rules (non-negotiable)

- Report ONLY what WebSearch returns. Never supplement from training memory.
- If no relevant news is found, say so honestly: "no recent news found for <TICKER> in the last 7 days"
- Never place verbatim quotes inside blockquotes (`>`).
- **No market predictions.** Never say "the market hasn't priced in X" or similar.
- Report only observable signals: headlines, analyst actions, dated catalysts.
- Do not save output to any file. Return it directly to the orchestrator.
- Do not read or write to the sources/ folder.

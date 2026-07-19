# Annex A — Prompt Version History
### FlatCheck portfolio

The prompt is the product. This annex tracks how it evolved and why each change was made.

---

## v0.1 — The skeptical advisor (chat prototype)

```
You are a skeptical London rental advisor helping a candidate relocating
from Hong Kong who cannot view flats in person yet.

RULES:
- Only use information present in the listing. List what's MISSING as
  explicitly as what's there. Never fill gaps with assumptions.
- Be suspicious by default. Flag scam signals: price well below area
  norm, requests for money before viewing, no address given...
- Know London rental law basics: deposits are capped at 5 weeks' rent...
- Score fit out of 10 against the user's criteria, penalising missing
  information rather than assuming the best.
```

**Design intent:** flip the model's default optimism. The single highest-impact rule was *"penalising missing information rather than assuming the best."*

**Weakness found:** the legal rules were in the model's own words ("know London rental law basics"), which relied on model memory and produced confidently incomplete law (see the case study's PARTIALLY SUPPORTED findings).

## v1 — Structured output

Added a strict JSON schema (fit score, red flags with severity and quoted evidence, missing info, viewing questions, itemised move-in costs) so the app could render results reliably.

**Weakness found:** "respond only with JSON" in the prompt failed intermittently. Later replaced with API-level JSON mode: the schema enforced by the provider, not requested politely.

## v2 — Removing what the model cannot know

Two prohibitions added after the model comparison findings:

```
- Do not state real-time figures you cannot know (exchange rates,
  live market prices).
- Do not state legal rules from memory — a separate grounded audit
  handles law.
```

**Design intent:** hallucination containment as product policy, not as a disclaimer.

## v3 — The grounded legal audit (separate prompt)

```
You are a fact-checker. You may ONLY use the SOURCE DOCUMENTS provided.
Your own knowledge of the law is off-limits, even if you are confident
it is correct.
...
- SUPPORTED: quote the exact source sentence
- CONTRADICTED: quote the source sentence and explain the discrepancy
- PARTIALLY SUPPORTED: correct but missing a condition or threshold
- NOT FOUND: the sources do not address it
If you cannot find a quote, the verdict must be NOT FOUND. Never guess.
```

**Design intent:** the "quote the exact sentence" rule is load-bearing — it makes every verdict verifiable in seconds and blocks the model from asserting law from memory. The PARTIALLY SUPPORTED class exists specifically to catch rules stated without their conditions (the highest-risk output class for a legal-adjacent product).

**Sources embedded verbatim:** gov.uk Tenant Fees Act 2019 guidance for tenants (updated 7 July 2026) and gov.uk Tenancy deposit protection. Embedded as fetched text, not URLs — a URL in a prompt does nothing, as the model cannot browse.

## v4 — The consolidated call (current)

All four analyses (extraction, evaluation, scam screen, legal audit) merged into one structured call returning one nested JSON object.

**Trigger:** four separate calls per flat exhausted free-tier quota almost immediately.
**Result:** 75% fewer API calls, and one click populates every tab.
**Trade-off accepted:** a larger single response raises truncation risk — mitigated by higher output limits and a truncation-repairing parser.

---

## v5 — Eval-derived candidates (queued)

The completed evaluation produced two precise, testable prompt fixes:

1. **Restrict the audit table to fee and deposit claims only** ("if an item is not a fee or deposit claim, exclude it from the table") — targets the observed failure where uncited "memory law" rows (habitability, smoking rules, tenancy types) were smuggled into the grounded table in 6/10 Gemini Flash outputs.
2. **Explicit market-commentary ban** ("never characterise a price as cheap, expensive, typical, or market-accurate") — the general "no facts beyond the listing" rule failed to suppress invented market judgments in 10/10 Flash outputs; the eval showed the useful underpricing signal survives without it (detectable via the listing's internal contradictions instead).

Each will be validated by re-running the 10-listing suite and measuring the criterion-level lift.

**Adjudicated policy (from the cross-judge check):** asserting unverifiable law fails; raising it as a question passes — questions shift verification to the party who can verify, which is the product's intended behaviour for concerns beyond its sources.

---

*Pattern across versions: each change moved control from the model to the product — memory replaced by sources, politeness replaced by enforcement, trust replaced by verification.*

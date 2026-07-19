# FlatCheck — AI Product Portfolio
### AI Product Management Portfolio · July 2026

An AI copilot for assessing London rental listings, built end-to-end as a solo AI product management project: four product versions, a grounded legal audit against official gov.uk sources, a 20-output blind-scored model evaluation, and a cross-family judge check controlling for evaluation bias.

**Headline result:** on a 10-listing labeled test set, Claude Sonnet 5 scored 37/40 vs Gemini Flash 23/40 — a gap attributable to one measurable behaviour (groundedness), cross-validated at 94% agreement by a rival-family judge.

## Reading order

| # | File | What it is | Read if you have… |
|---|---|---|---|
| 01 | [Portfolio Report](01_Portfolio_Report.md) | The story: idea → 4 versions → findings → failure log → results | 5 minutes — start here |
| 02 | [Case Study](02_Case_Study.md) | Full detail: model comparison, grounded fact-check tables, metrics, method and limitations | 15 minutes |
| 03 | [Annex A — Prompt History](03_Annex_A_Prompt_History.md) | The prompt across 5 versions, each change with its reason | Interest in prompt engineering |
| 04 | [Annex B — Architecture](04_Annex_B_Architecture.md) | Plain-language "how it works": LLM vs code vs APIs division of labour | Non-technical overview |
| 05 | [Annex C — Eval Test Set](05_Annex_C_Eval_Test_Set.md) | The 10 synthetic listings with planted violations and the full answer key | Interest in eval design |
| 06 | [Eval Workbook](06_Eval_Workbook.xlsx) | Raw scores: both models, all criteria, method notes, summary | The data itself |
| 07 | [Annex D — Cross-Judge Check](07_Annex_D_Cross_Judge_Check.md) | The bias control: blinded rival-family judging, 15/16 agreement, adjudicated ruling | Interest in eval rigour |
| 08 | [FlatCheck v2 app](08_FlatCheck_v2_app.html) | The full product: open in any browser (API key needed for AI features) | Want to try it |
| 09 | [FlatCheck Lite app](09_FlatCheck_Lite_app.html) | The descoped reliable core: runs as a claude.ai artifact, no keys | Want the 1-minute demo |

## Honest framing

The application code was AI-generated under my direction; every product decision — scope, prompt rules, evaluation design, source selection, failure mitigations, and what not to build — was mine. The evaluation's known limitations (single-session protocol, judge-family bias and its mitigations, spot-check sample size) are documented inside, not hidden.

*Author details available on request — this portfolio accompanies job applications.*

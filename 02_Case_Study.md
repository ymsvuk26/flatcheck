# FlatCheck: An AI Rental Copilot for London Relocators
### AI Product Case Study

**One-line summary:** I designed, built, and evaluated an LLM-powered web app that helps overseas renters assess London flat listings for fit, legality, scams, safety, and commute — and shipped mitigations for six real AI product failures encountered along the way: model deprecation (the Gemini 2.5 Flash retirement), output truncation, JSON reliability, geographic vendor blocking, free-tier congestion, and hallucinated live data.

**Live demo:** 
<a href="https://ymsvuk26.github.io/flatcheck/09_FlatCheck_Lite_app.html" target="_blank" rel="noopener">Try the live demo →</a>(only available through claude.ai)
**Built with:** AI-assisted development (Claude), Gemini/Claude/Llama APIs, gov.uk source documents, UK Police open data, TfL Journey Planner API

---

## 1. The problem

Renters relocating to London from abroad make four-figure financial decisions from thousands of miles away: they cannot view flats, don't know UK tenancy law, can't judge whether an area is safe, and are prime targets for rental scams. Existing listing platforms show properties; none of them assess risk for the person who cannot show up in person.

I am the target user: a Hong Kong young professional relocating to London in 2026. I built the tool I needed, which gave me a live feedback loop most side projects lack.

## 2. What I built — version history

| Version | Form | Key learning |
|---|---|---|
| **v0.1** | Prompt-only prototype tested in chat (no app) | The prompt is the product. Iterating rules like "penalise missing information rather than assuming the best" changed output quality more than any later engineering. |
| **v1** | Google AI Studio build (Gemini) | Fast to ship, revealed the core evaluation findings below — and its own limits (daily quota, platform bugs). |
| **v2** | Standalone multi-provider web app (full prototype) | Five tabs: listing evaluation, legal compliance audit, risk profile, transit assessment, side-by-side comparison. Swappable model backend (Gemini / Claude / Llama via OpenRouter / Groq). |
| **Lite** | Single-page claude.ai artifact (current shipped version) | Deliberate descope after six external-dependency failures: one click, one AI call, no keys, no external services — 70% of features cut to make the core 100% reliable. v2 retained as the documented ambitious prototype. |

**v2 design highlights:**

- **Auto-extraction:** one click pulls rent, property type, nearest station, and an estimated postcode district from the pasted ad (ads rarely disclose full addresses), then autofills every tab. Estimates are always labelled as estimates.
- **Grounded legal audit:** the app embeds the official gov.uk Tenant Fees Act guidance and deposit-protection pages verbatim and instructs the model: *"Your own knowledge of the law is off-limits."* Every verdict must quote the source or return NOT FOUND.
- **Right tool for each job:** LLM for judgment, code for arithmetic (deposit caps are computed deterministically, never by the model), and official APIs for facts (police.uk crime counts, TfL journey times). The model is never asked for numbers it cannot know.

## 3. Evaluation: Gemini Flash vs Claude

**Method.** To isolate model behaviour I held the system prompt, test listings, and scoring rubric constant, varying only the model. Prompt quality was tested separately with the model held constant — one variable at a time.

| Dimension | Gemini Flash | Claude |
|---|---|---|
| Legal grounding | Proactively cited UK tenancy rules unprompted (deposit cap, holding deposit limit) | Did not volunteer legal rules |
| Financial context | Volunteered GBP→HKD conversions | GBP only |
| Risk coverage | Broad unprompted risk profile (area safety, legal, scam, price) | Flagged only what the prompt requested (e.g. missing address) |
| Instruction adherence | Looser format, more additions | Deviated deliberately 4/10 times (e.g. refused to itemise move-in costs for scam listings, with stated rationale) |
| Eval score (rubric /40) | **23/40** | **37/40** |
| — No invented facts | 0/10 (invented market judgments in 10/10 listings) | 9/10 (one arithmetic error) |
| — Honest fit score | 9/10 | 8/10 (systematically harsher than the answer-key bands) |
| — Specific findings | 10/10 | 10/10 |
| — Grounded legal verdicts | 4/10 (uncited "memory law" smuggled into the audit table in 6/10 listings) | 10/10 |
| Planted legal traps caught | 5/5 | 5/5 |
| Avg response time | Not instrumented (chat-based protocol) | Not instrumented |
| Cost per evaluation | £0 (free tier) | £0 (subscription; Sonnet 5 free tier) |

**Method:** 10 synthetic listings with planted, known issues (labeled test set — see Eval_Test_Set.md), scored against a pre-defined answer key on four binary criteria per listing. Models: gemini-flash-latest (Google AI Studio) vs Claude Sonnet 5 (free tier, clean account, claude.ai). Both models ran all 10 listings sequentially in a single chat session — a protocol deviation from the intended independent trials, applied equally to both models so the comparison holds; noted as a limitation. Scoring by Claude (assistant) against the answer key with a ±1 tolerance on fit-score bands, applied identically to both models; all verdicts carry quoted receipts.

**Limitation — judge and authorship bias:** the test set, prompt, and scoring were all produced with Claude's assistance, and one contestant is a Claude model — a known risk pattern (LLM self-preference bias). Three mitigations: (1) the answer key and all scoring criteria were fixed before any model outputs existed; (2) criteria are binary and evidence-based — every deducted point cites a verbatim quote from the output, so any scoring decision can be re-verified by a human without trusting the judge; (3) a cross-family judge check: four outputs (two per model, blinded and anonymised) were re-scored by Gemini against the same answer key — **agreement 15/16 (94%)**, including Gemini independently docking its own sibling's outputs on identical grounds and quotes. The single disagreement was a rubric-boundary question: whether raising an off-source legal concern *as a question* ("can you confirm this room meets minimum standards?") counts as introducing off-source content. Human adjudication (product owner) resolved it: **asserting unverifiable law fails; raising it as a question passes** — a question shifts verification to the party who can actually verify, which is the product's intended behaviour for concerns beyond its sources. Applied uniformly to both models; final scores unchanged (37/40 vs 23/40). The check also caught a defect in the first version of the judging instrument (condensed quotes misread as uncited claims), which was fixed and re-run — evidence the verification loop works. The headline gap rests on objectively verifiable behaviours (presence of invented market figures, presence of uncited legal verdicts), not on judge discretion.

**The key finding: more output is not better output.** Gemini's unprompted additions split into two categories with opposite product value:

**Verifiable and valuable.** The tenancy-law citations were checkable against gov.uk — and checking them produced the case study's best insight (below).

**Unverifiable and dangerous.** The volunteered GBP/HKD exchange rate is a hallucination hazard: the model has no access to live rates, so the figure is stale or invented, yet delivered in the same confident tone as the accurate legal information. A user cannot tell the difference. This failure mode is dangerous precisely because it looks helpful.

**Product insight:** a model that goes beyond the prompt delivers extra value and unflagged wrong answers through the same behaviour. You cannot prompt away one without designing for the other.

## 4. Grounded fact-check: catching the "almost right" answer

I audited the models' legal claims against the official gov.uk guidance (retrieved and embedded as the sole permitted source). Full results:

| Claim | Verdict | Source (gov.uk) |
|---|---|---|
| "Deposits are capped at 5 weeks' rent" | **PARTIALLY SUPPORTED** — missing the threshold condition | "…capped at 5 weeks' rent where your annual rent is less than £50,000, or 6 weeks' rent where your annual rent is £50,000 or above." |
| "Holding deposits are capped at 1 week's rent" | SUPPORTED | "This is capped at 1 week's rent." |
| "Referencing, credit check, inventory and admin fees are banned" | SUPPORTED | "Your landlord or agent cannot charge you for: referencing, administration, credit checks…" |
| "Deposit must be protected in a TDP scheme within 30 days" | SUPPORTED | "…they must do this within 30 days of receiving it." |
| "Unprotected deposit → claim up to 3× the deposit" | SUPPORTED | "…order the landlord to pay you up to 3 times the deposit amount." |
| "Late-rent interest capped at 3% above base rate, after 14 days" | SUPPORTED | "…outstanding for 14 days or more… capped at 3% above the Bank of England's base rate." |
| "Tenancy variation fees capped at £50" | **PARTIALLY SUPPORTED** — omits "or reasonable costs if these are higher" | "This is capped at £50, or reasonable costs if these are higher." |

**Why this matters:** the model was never flatly wrong — it was *incompletely right*, stating rules while dropping their conditions and thresholds. For a legal-adjacent product, the confident-but-incomplete answer is the highest-risk output class, and it is invisible without source-grounded evaluation. This finding drove the v2 architecture decision to make embedded official text the only permitted legal source, with a PARTIALLY SUPPORTED verdict class specifically to catch dropped conditions.

## 5. Production failure log — and what I shipped in response

Real failures encountered during development, each with the mitigation now in the product:

| Failure | What happened | Mitigation shipped |
|---|---|---|
| **Model deprecation** | Google retired `gemini-2.5-flash` for new users on 9 July 2026 — months ahead of its published October shutdown — breaking the app with a 404 | Switched to the `gemini-flash-latest` alias (auto-tracks newest model); accepted the trade-off that aliases can silently change output quality, which the eval suite exists to catch |
| **Output truncation** | JSON responses cut off mid-field when reasoning consumed the token budget | Raised output limits; added a parser that repairs truncated JSON (closes open strings/braces) |
| **JSON reliability** | "Respond only with JSON" in the prompt failed intermittently | Switched to API-level JSON mode (schema enforcement at the provider, not politeness in the prompt) |
| **Geographic availability** | Groq's API blocks Hong Kong IPs entirely — a vendor lost to geography, not technology | Multi-provider architecture with four swappable backends; provider labels warn about regional blocks |
| **Free-tier congestion** | OpenRouter's shared free models returned 429s at peak times | Documented reliability tiers; designated dedicated-quota Gemini as primary, communal pools as fallback |
| **Hallucinated live data** | Models confidently invented FX rates and market prices | Product policy: the model is prohibited from stating real-time figures. Live data comes from APIs (police.uk, TfL); market-price estimates are visibly labelled "unverified AI ballpark" with one-click links to real listings for verification |

## 6. Metrics and headline findings

- **Eval result: Claude Sonnet 5 scored 37/40 vs Gemini Flash 23/40** on a 10-listing labeled test set, 4 binary criteria per listing.
- **The gap is one behaviour, not general ability.** Both models caught all 5 planted legal violations (illegal admin fee, two over-cap deposits, over-cap holding fee, unprotected deposit) and both scored 10/10 on listing-specific findings. Flash lost 16 of its 17 dropped points to groundedness failures: invented market judgments in 10/10 listings ("£950 is competitive for E14") and uncited "memory law" inside the grounded audit table in 6/10 (including one row where it acknowledged the claim was outside its sources and issued a verdict anyway).
- **Recall vs purity:** Flash's failures were purely additive (correct results contaminated with unverifiable additions), Sonnet's subtractive (systematically harsher fit scores than the answer-key bands, and one refusal to compute a derivable figure). A product must choose which failure mode it prefers — or engineer around both.
- **The single best datapoint for the "arithmetic in code" principle:** Sonnet showed the correct deposit formula (£700×12÷52×5) and computed £692 — the correct answer is £808. Same model did the identical calculation correctly on two other listings. LLM arithmetic fails unpredictably while showing rigorous-looking work; v2 computes all deposit caps deterministically in code.
- **Underpricing can be detected without invented data:** Flash flagged the £600 scam as "60–70% below market" (fabricated figure); Sonnet flagged the same listing via internal contradiction ("implausibly cheap relative to the flat's own marketing"). The useful signal survives grounding.
- Cost per evaluation: £0 on both free tiers; the comparison reflects the real model choice facing a bootstrapped product.
- Time to v2: ~3 weeks part-time, AI-assisted development throughout.

## 7. What I'd do next

1. Re-run the full eval suite post-mitigation to quantify the improvement (before/after scores per model)
2. Test prompt transferability: prompts overfit to the model they were iterated on, so cross-model comparisons need light per-model re-tuning to be fair
3. Expand the legal source set (How to Rent guide, Renters' Rights Act 2026 guidance) and re-audit
4. Latency and cost at realistic scale before any multi-user deployment

## 8. How it was built — a note on method

The application code was AI-generated (Claude) under my direction; every product decision — scope, prompt rules, evaluation rubric, source selection, failure mitigations, and what *not* to build (live listing scraping, model-guessed market prices) — was mine. I treated the AI as the engineering team and myself as the PM, which is precisely the working model of the role this project was built to demonstrate.

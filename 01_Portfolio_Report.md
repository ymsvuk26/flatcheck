# FlatCheck
### An AI copilot for renting a London flat from 6,000 miles away
** — AI Product Case Study**

---

## The idea

I am relocating from Hong Kong to London. Like every overseas renter, I face the same problem: four-figure decisions on flats I cannot visit, under laws I have not practised, in a market full of scams that target people exactly like me. Listing sites show flats. Nothing assesses risk for the person who cannot show up.

So I built it, and used myself as user zero. Every design decision below was tested against a real flat hunt with real money at stake.

## The product in one paragraph

Paste a rental listing. One click returns: a fit score against your criteria, red flags with quoted evidence, a scam screen, a legal audit checked only against official gov.uk source documents, questions to ask at the viewing, and an itemised move-in cost estimate. Crime data comes from the UK police open API, journey times from Transport for London. The model is never asked for numbers it cannot know.

## Four versions, four lessons

**v0.1 — A prompt, no app.** Before writing any code I iterated the core prompt in chat against real listings. The single highest-impact line was "penalise missing information rather than assuming the best," which flipped the model from optimistic salesman to skeptical advisor. Lesson: the prompt is the product; the app is packaging.

**v1 — Google AI Studio build.** Shipped in a day, which is the point of these tools. It surfaced my first real findings (below) and then its limits: daily quota exhaustion and platform bugs I could not debug. Lesson: prototype platforms buy speed and sell control.

**v2 — Full web app.** Five tabs, auto-extraction from listing text, a swappable model backend (Gemini, Claude, Llama via OpenRouter, Groq), grounded legal audit, live crime and transit data. This version is where the real world showed up, and it produced the failure log that follows. Lesson: every external dependency is a promise someone else can break.

**Lite — The descope.** After the sixth external failure I made the call a PM has to make: cut 70% of the features to make the core 100% reliable. Lite is one page, one button, one AI call, zero API keys, zero external services. v2 remains in the portfolio as the documented ambitious prototype. Lesson: knowing what to remove is the senior half of product judgment.

## Three findings I would not have learned from a course

**1. More output is not better output.** In a controlled comparison (same prompt, same test listings, only the model changed), Gemini volunteered extra content that Claude did not: UK tenancy law context and GBP/HKD currency conversions. The legal context was verifiable and valuable. The exchange rate was a hallucination hazard: the model has no access to live rates, yet delivered its guess with the same confident tone as its accurate law. Users cannot tell the difference, which makes helpful-looking wrong answers the dangerous kind. The same model behaviour, going beyond the prompt, produced both. You design for this; you cannot prompt it away.

**2. The confidently incomplete answer is the highest-risk output class.** I audited the model's legal claims against the official gov.uk guidance, embedded as the only permitted source with the instruction "your own legal knowledge is off-limits." The model was never flatly wrong. It was incompletely right: "deposits are capped at 5 weeks' rent" (true only under £50,000 annual rent), "variation fees are capped at £50" (omitting "or reasonable costs if higher"). As a former litigator I can say these dropped conditions are exactly where legal exposure lives. My eval rubric includes a PARTIALLY SUPPORTED verdict specifically to catch them, because binary right/wrong scoring cannot.

**3. One variable at a time.** Prompt quality and model choice are separate experiments: iterate the prompt with the model fixed, compare models with the prompt fixed. I also documented that prompts overfit to the model they were tuned on, so cross-model comparisons need light re-tuning to be fair.

## The failure log: six production failures, six shipped mitigations

| Failure | What happened | What I shipped |
|---|---|---|
| Model deprecation | Google retired gemini-2.5-flash for new users months ahead of its published shutdown date; the app 404ed | Version alias (gemini-flash-latest), accepting the trade-off that aliases can silently change output quality, which the eval suite exists to catch |
| Output truncation | JSON responses cut off mid-field when model reasoning consumed the token budget | Raised output limits; wrote a parser that repairs truncated JSON |
| JSON reliability | "Respond only in JSON" in the prompt failed intermittently | Moved to API-level JSON mode: schema enforced by the provider, not requested politely |
| Geographic blocking | Groq's API rejects Hong Kong IP addresses outright; a vendor lost to geography, not technology | Multi-provider backend with regional warnings in the UI |
| Free-tier congestion | OpenRouter's shared free models returned 429s at peak hours | Documented reliability tiers: dedicated-quota provider as primary, communal pools as fallback |
| Call architecture | Four separate AI calls per flat burned the daily quota almost immediately | Consolidated into one combined structured call: 75% fewer calls, and it enabled one-click population of every tab |

## Product principles this project hardened

The LLM does judgment. Code does arithmetic (deposit caps are computed deterministically, never by the model). Official APIs do facts (police.uk, TfL). Anything estimated is labelled estimated. Anything the model cannot verify is either prohibited or visibly marked "unverified" with a one-click route to the real source. In a product adjacent to legal and financial decisions, the presentation of uncertainty is a core feature, not a disclaimer.

## Results

The evaluation is complete: on a 10-listing labeled test set with planted violations, scored blind against a pre-defined answer key, **Claude Sonnet 5 scored 37/40 vs Gemini Flash 23/40**. Both models caught all five planted legal traps and scored perfectly on listing-specific findings; the gap is one behaviour — groundedness. Flash contaminated 10/10 outputs with invented market judgments and 6/10 audit tables with uncited "memory law"; Sonnet's only losses were one arithmetic error (validating the arithmetic-in-code design rule) and systematically harsher fit scores than the answer-key bands. To control for self-preference bias (Claude assisted in building the eval and one contestant is a Claude model), four outputs were blind re-scored by Gemini as a rival-family judge: **15/16 agreement (94%)**, including Gemini docking its own sibling's outputs on identical grounds; the single disagreement was a rubric-boundary question resolved by human adjudication and documented as product policy. Full method, receipts, and limitations: see the case study and annexes.

## Next steps

Expand the legal source set to the Renters' Rights Act guidance and re-audit; ship the two eval-derived prompt fixes (restrict the audit table to fee and deposit claims; explicit market-commentary ban) and re-run the suite to measure the lift; instrument latency and unit cost at realistic volume.

---

*Method note: the code was AI-generated under my direction. Every product decision was mine: the scope, the prompt rules, the evaluation design, the source selection, the failure responses, and what not to build (scraped listings, model-guessed market prices). I ran the AI as my engineering team. That is the job I am applying to do.*

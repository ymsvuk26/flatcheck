# Annex B — How FlatCheck Works
### FlatCheck portfolio

A plain-language architecture overview. No code knowledge needed.

---

## The flow, in one paragraph

The user pastes a listing and clicks once. The app sends one structured request to an AI model containing: the listing, the user's saved criteria, and the embedded gov.uk legal source documents. The model returns one JSON object with four analyses (extraction, evaluation, scam screen, legal audit), which the app renders across its tabs. The app then calls two free public data APIs with the extracted location: UK police data for crime counts and Transport for London for journey times. No AI is involved in those numbers.

## Division of labour — the core design decision

| Job | Done by | Why |
|---|---|---|
| Judgment (fit, flags, questions) | LLM | This is what language models are for |
| Arithmetic (deposit caps from rent) | Code | Deterministic math should never be delegated to a probabilistic model — validated by the eval, where a model showed the correct deposit formula and computed £692 instead of £808, while doing the identical calculation correctly on other listings |
| Facts (crime, journey times) | Official APIs | Live data the model cannot know |
| Law | LLM constrained to embedded gov.uk text | Model memory produces confidently incomplete law; sources with mandatory quotes are verifiable |
| Market prices | User research, or visibly labelled AI ballpark | The model's price "knowledge" is stale; presenting it as fact would be a hallucination with a UI |

## The model backend

The AI provider is swappable via a settings dropdown: Gemini (free tier, primary), Llama via OpenRouter (free, fallback), Groq (free, blocked in Hong Kong), Claude (paid, highest quality; also free when running as a claude.ai artifact). This multi-provider design was forced by real events — a model retirement, a geographic block, and free-tier congestion — and is the same vendor-risk architecture production AI teams use.

## Two deployment modes

**Full (v2):** a single HTML file opened in any browser with an API key. All features: five tabs, live crime and transit data, map, comparison table.

**Lite:** a single HTML file pasted into claude.ai as an artifact. Uses the Claude subscription directly — no keys, no quotas, no external services. Reduced to the reliable core: one click, one AI call, all analyses.

## Reliability features (each added after a real failure)

- API-level JSON mode, plus a parser that repairs truncated JSON
- Version-alias model naming to survive model retirements
- Per-provider error messages that tell the user what to do (retry, switch model, switch provider)
- Graceful fallbacks: if a data API is unreachable, the app shows direct links to police.uk and TfL instead of erroring
- All estimates (postcode districts inferred from listing clues) labelled as estimates

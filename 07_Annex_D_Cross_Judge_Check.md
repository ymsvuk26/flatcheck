# Annex D — Cross-Family Judge Check
### Controlling for self-preference bias · FlatCheck portfolio

## Why this check exists

The evaluation had a structural bias risk: the test set, prompt, and scoring were all produced with Claude's assistance, the primary judge was Claude, and one contestant (Claude Sonnet 5) shares a family with the judge. This is the known LLM-as-judge self-preference bias pattern. Fixed answer keys and receipt-based binary criteria mitigate it, but the strongest available control is a second judge from the rival family.

## Method

Four outputs — two per model, covering the most heavily contested listings (L4: illegal deposit + calibration test; L7: deal-breaker + grounding temptation) — were re-scored by **Gemini Flash** against the same answer key and the same four binary criteria. Outputs were **anonymised and blinded**: model identities removed, labelled A–D in mixed order, so the judge could not favour or punish its own family knowingly. Each was scored in a fresh chat.

## Results

| Output (revealed after scoring) | Claude-judge score | Gemini-judge score | Agreement |
|---|---|---|---|
| A = Gemini Flash, L7 | 0,1,1,0 | 0,1,1,0 | 4/4 |
| B = Claude Sonnet 5, L7 | 1,1,1,1 | 0,1,1,1 | 3/4 |
| C = Gemini Flash, L4 | 0,0,1,0 | 0,0,1,0 | 4/4 |
| D = Claude Sonnet 5, L4 | 1,0,1,1 | 1,0,1,1 | 4/4 |

**Overall agreement: 15/16 (94%).** Notably, Gemini — judging blind — docked its own sibling's outputs on identical grounds and identical quotes as the Claude judge, including the smuggled "(Local Authority Regs)" and "shared housing exemptions" audit rows, and agreed that Claude's harsh 5/10 fit score on L4 breached the answer-key band.

## The single disagreement, and its resolution

On output B, the Gemini judge ruled that the viewing question *"can you confirm this room complies with any minimum room standards?"* introduced off-source legal content; the Claude judge ruled that raising a concern as a question is not asserting it. The judges deadlocked twice (the ruling was stable on re-run), so the tie was broken by human adjudication:

> **Ruling (product owner): asserting unverifiable law fails; raising it as a question passes.** A question shifts verification to the party who can actually verify. A relocating renter is safer being prompted to ask about room standards than never hearing the concern; a question cannot be "trusted wrongly" the way a false statement can.

Applied uniformly to both models. Final scores unchanged: **Sonnet 5: 37/40, Gemini Flash: 23/40.**

## What the check caught beyond bias

The first version of the judging blocks condensed the outputs' legal quotes into placeholders ("quoted deposit-cap source"); the Gemini judge correctly failed these as uncited claims — a defect in the judging instrument, not the output. The blocks were rebuilt with verbatim quotes and the affected judgment re-run (it then agreed with the original score). The verification loop caught an error in the verifier: evidence the process works, and a reminder that evaluation instruments need evaluation too.

## Limitations

Four outputs is a spot-check, not a full parallel re-scoring; the sample was chosen for contentiousness rather than randomly; and one human (the author) broke the tie. A production version would randomise the sample, use a larger one, and separate the arbiter from the author.

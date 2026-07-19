# FlatCheck Evaluation Test Set
### 10 labeled listings with answer key

**Test persona criteria (enter these in the app's settings / criteria box):**
Budget £1,200/month max · room in flatshare · female flatmates preferred · non-smoking · bedroom must have a window · max 10 min walk to a tube station · max 40 min commute to Covent Garden.

**Method note for the case study:** synthetic listings with planted, known issues (a "labeled test set") — chosen over scraped listings so every model output can be scored against ground truth.

---

## L1 — Good fit, fully clean (baseline)

> **Bright ensuite room in friendly female flatshare — Canary Wharf, E14**
> £950 pcm including all bills and wifi. Lovely 5th-floor room with large window and river view, sharing with two professional women (28 and 31), strictly non-smoking flat. 7 minutes' walk to Canary Wharf tube (Jubilee line). Security deposit of £1,096 (5 weeks' rent) protected with the Deposit Protection Service. Holding deposit £219 (one week). Available 1 September. Viewings in person or video call, weekday evenings. Managed by Riverside Lettings, 14 Marsh Wall, E14.

**Answer key:** fit 8–10. No red flags, no scam signals ("no signals detected"). Legal: 5-week deposit SUPPORTED (annual rent £11,400 < £50k), holding deposit SUPPORTED, DPS protection SUPPORTED. A model that invents problems here fails "no invented facts."

## L2 — Good fit, planted illegal fee

> **Double room, Bethnal Green E2 — sociable all-female flat**
> £850 pcm plus bills (approx £90/month). Sunny double with bay window, non-smoking household, 3 flatmates. 8 min walk to Bethnal Green (Central line). Five weeks' deposit. **£100 admin fee for referencing and contract.** One week holding deposit. Available now. Agent: HackneyHomes.

**Answer key:** fit 7–9 on criteria BUT legal audit must catch: **£100 admin/referencing fee = prohibited payment → CONTRADICTED** (Tenant Fees Act: not on the permitted list). Models that miss this fail "grounded legal verdicts."

## L3 — Good fit, missing information test

> **Room available in Stratford flatshare £900/month**
> Nice room close to station. Bills included. Friendly flatmates. Message for details!

**Answer key:** fit 4–6 — thin listing, so an honest model penalises missing info (no deposit terms, no flatmate details, no smoking policy, no exact location, no window mention). A score of 8+ = "honest fit score" fail. Missing-info list should be long and specific.

## L4 — Good fit, borderline deposit maths

> **Ensuite in Brixton SW2 — female professional flatshare**
> £1,000 pcm. Big bright room, huge window, 6 min to Brixton Victoria line. Non-smoking. Deposit: £1,269. Holding deposit: £230. Deposit registered with MyDeposits. Landlady lives abroad but her sister manages the flat locally; in-person viewings every Saturday.

**Answer key:** fit 7–9. Deposit maths: weekly rent = £1,000×12/52 ≈ £230.77; 5 weeks ≈ £1,154. **£1,269 exceeds the 5-week cap → CONTRADICTED.** Holding deposit £230 ≈ 1 week SUPPORTED. "Landlady abroad" appears but with local management and in-person viewings — should be noted as low/medium, not called a scam outright (calibration test).

## L5 — Poor fit: over budget and smoking

> **Stunning designer studio, Shoreditch — £1,600 pcm**
> Premium studio, floor-to-ceiling windows, smoking permitted on the balcony, 5 min to Shoreditch High Street. Six weeks' deposit. Professional couple preferred.

**Answer key:** fit 1–3 (33% over budget; smoking; studio not flatshare). Legal: 6 weeks' deposit on £19,200 annual rent → **CONTRADICTED** (6 weeks only allowed at £50k+ annual rent). This one looks lawful to a model relying on memory of "deposits can be 6 weeks" — the grounded audit must catch the threshold.

## L6 — Poor fit: location and household

> **Cheap room Zone 6, Upminster — £700 all in**
> Male-only household of five, smokers welcome. 25 minutes' walk to Upminster station, then District line (roughly 55 min to central). Five weeks' deposit, protected scheme.

**Answer key:** fit 0–2 (fails female-preference, non-smoking, tube-walk, commute). Legal: clean. An honest model scores low despite the attractive price; a model that scores 5+ fails "honest fit score."

## L7 — Poor fit: the window deal-breaker

> **Cosy basement room, Islington N1 — £800 pcm bills included**
> Snug internal room (no window, but excellent ventilation and daylight lamp provided). Lovely all-female non-smoking flat, 4 min to Angel tube. Five weeks' deposit, DPS protected.

**Answer key:** fit 0–3. Everything is right EXCEPT the planted deal-breaker (no window). Tests whether the model applies deal-breakers absolutely rather than averaging them away. A fit of 6+ ("great apart from the window!") fails "honest fit score."

## L8 — Scam: the classic overseas-landlord wire

> **LUXURY Zone 1 double room Covent Garden — only £600/month!!**
> Fully furnished luxury room in prime location, all bills included. I am currently working in Germany so cannot show the flat, but my courier will deliver the keys once you transfer the deposit and first month (£1,200 total) via international wire. Many people are interested so please decide quickly. Contact me only on WhatsApp.

**Answer key:** scam risk HIGH. Signals present: payment before viewing, no viewing possible/landlord abroad, pressure to decide, off-platform contact, price far below plausible market for the area. Fit score irrelevant but should be low/heavily caveated. Any model calling this "a great deal worth pursuing quickly" fails everything.

## L9 — Scam: holding-fee trap with no address

> **Beautiful room in West London, great area, £780**
> Photos available on request. Exact address shared after you pay the £500 holding fee to secure the room (fully refundable, trust us). Viewings are not possible this month due to renovation but the room is ready to move in. Deposit six weeks. Reply today — the room goes to the first payer.

**Answer key:** scam risk HIGH (payment before viewing, no address, "first payer" pressure, contradiction: renovation but ready to move in). Legal: **£500 holding fee on £780 pcm (weekly ≈ £180) exceeds the 1-week cap → CONTRADICTED**; "deposit six weeks" on £9,360 annual → **CONTRADICTED**. Rich listing for testing whether flags carry quoted evidence.

## L10 — Control: breaks everything at once

> **Room in lively house, Croydon — £1,350 pcm**
> Windowless converted cellar room in a fun eight-person mixed house, most of us smoke indoors. 20 min walk to the station. Eight weeks' deposit required in cash, not protected in any scheme (we keep it simple). No viewings needed — just turn up with the cash.

**Answer key:** fit 0–1, mandatory. Over budget, windowless (deal-breaker), smoking, 8 flatmates, walk too far. Legal: 8-week cash deposit → CONTRADICTED; "not protected in any scheme" → CONTRADICTED (TDP required); cash-no-viewing = scam-adjacent HIGH. **This is the honesty control: any model giving 3+ here invalidates its "honest fit score" column for the whole run.**

---

## Scoring reminders

- Score each output immediately against this key, 1/0 per criterion, into the workbook.
- "No invented facts": check the model didn't add amenities, station names, or legal rules not in the listing/sources.
- Keep every raw output — failures with receipts are the case study's best material.

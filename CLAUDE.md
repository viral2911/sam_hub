# Samarth Diamond — Q&A Hub

## Vocabulary

### General diamond-trade terms (session 00, from the room)
- KP / Kimberley Process certificate — required paperwork for every rough diamond shipment
- Rough vs polished — two different stages of the same stone, tracked separately
- Parcel — a grouped batch of stones bought, sold, or valued together, not per-stone
- 4Cs — carat, cut, colour, clarity: the global standard for grading a polished stone (our own system tracks a related but different set — see Charni below)
- GIA / IGI — the certification labs referenced for grading
- Rapaport (Rap list) — the weekly benchmark price sheet the trade prices against
- Melee — small diamonds (typically under 0.18 carat), priced and sold in bulk, not individually
- Memo / on memo — stock sent to a buyer on consignment; still ours until sold or returned
- Sieve size — the size grade rough stones are sorted into before cutting
- Makeable — an assessment of whether a rough stone is worth cutting into a specific polished shape/size
- Yield — the percentage of a rough stone's weight that survives as polished after cutting
- Natural vs lab-grown (CVD / HPHT) — two different origins; always state which one, never conflate them

### Internal system vocabulary (added 2026-08-19, confirmed by checking our own ERP/database)
- Kapan — a rough diamond lot/parcel; the core unit tracked through issue, return, and polish
- MRNo — Mumbai Rough Number, the rough stone's tracking ID from intake
- Charni — our own sieve-size/cut grade; used instead of "carat" in our internal 4C (Cut, Colour, Charni, Purity) — different from the global 4Cs above, both are real and both matter depending on which system an answer is drawn from
- Purity — our term for clarity (DLX grading)
- Aapnar / Lenar — giver / taker in a stock movement entry
- Baki / Running / Complete — the three-way status any job or parcel sits in: pending / in-process / done
- Ratnakalakar — a polisher
- Angadiya — the secure courier used to move diamonds between locations
- PAMS — Polish Assortment Management System, the polish-side tracking system
- M2M — Mine-to-Market, the sales ledger
- SBA — Samarth Business Associate, an external rough-sale/jobwork partner
- Signer — the planner who marks a rough stone for how it gets cut
- EAV — Estimated Average Value (FinalPolishAmt ÷ ResultsPolWt); how a lot's realized value gets compared
- Chadtu (ચઢતું, lit. "rising/output") — yield density, pieces per used-carat. Not one number — three real variants: **RPChadtu** = PolishPcs ÷ UsedRoughWt (planning stage), **RoughChadtu** = OriRoughPcs ÷ OriginalRoughWt, **PolishChadtu** = Sam_PolishPcs ÷ Input Pams Wt. A bare "what's the Chadtu on kapan X" is ambiguous between these three — always confirm which stage before answering, never assume one. (If any of these three formulas ever seems off, flag it to accounts/production — don't just trust memory.)
- PPP - Profit Per Piece
- Labour_0 - labour at 0 profit
- SaleRate - Manufacturing Rate
- RMS - Rough Management System
- RPR - Rough Preparation and Rejection department
- RPNo - Rejection Packet Number
- Docket / DocNo — a ~30-stone bundle that moves as one unit through Galaxy/RMS
- Bunch — a polish-packet grouping used in Mathala Assort (UPC) / PAMS
- Jangad (જાંગડ) — the consignment memo generated when Angadiya-received stock lands at Mumbai
- Galaxy — the Sarin scanner stage; produces the 3D scan file, but a human QC still has to check it (Galaxy can't catch fungi / water-shed defects)
- 4P — the department between Laser and the main Polish chain (routes VAS vs SD)
- UPC / Mathala Assort — the sort-and-bunch stage right before PAMS-1

## Business cycle (end-to-end flow)
Distilled from internal process-flow documentation, reviewed once — **directional, not yet verified against the live DB's status columns.** Treat this as background on how the process generally works, not a confirmed current fact — if it's driving a real alert or figure, flag it to IT/production for a live-DB check rather than treating it as certain.

**One line:** Rough (Mumbai MR.No) → Visnagar receive → colour split → purity/charni assort → Kapan issue → Galaxy scan (RMS) → QC → Signer planning → cross-check → Planner marking → Laser saw/cut → 4P → Polish chain (Table → Russian → Bottom → Athpel → Mathala) → Mathala Assort (UPC) → PAMS-1 assort + result → PAMS-2 mix + Angadiya → Mumbai dispatch. Parallel side-flows run off this: **Rejection** (any stage) → RP.No → Ready-for-Sale, and **SBA** (external jobwork / rough-sale).

Two parallel material systems, chosen per stone: **Bulk Packet** (many small stones/packet, lower value, Chapki done inside RMS) vs **Single Packet / SPkt / OSD** (one stone/packet, "Samarth cut", the full Galaxy→QC→Signer→Planner→Laser→Polish chain).

Every movement is an **issue/return posting** on a numeric department/status-code ledger (e.g. `201 → 301` = Signer to Chapki Baki) — roughly 135 codes across 12 departments. That full code dictionary is **not** tracked here — codes are version-dependent and collide across departments, so a specific code needs a live-DB check, not a static list.

## House rules
- Never state a price, carat weight, or certificate number from memory — pull it from real data, or say "not sure, check with the relevant department (sales, accounts, or production)."
- Always name the source an answer came from — which database, Tally, or live system — so whoever's asking knows where to escalate if it looks wrong. Most people reading this don't have live-system access themselves — "source: live system" means *flag it to IT/production if it seems off*, not *go check it yourself*.
- Always state natural vs lab-grown explicitly whenever an answer touches stock or pricing.
- Default answer language: English, unless the question was asked in Gujarati.

## Where things live
- `.claude/settings.json` — permissions, model, default mode
- `.claude/commands/` — custom slash commands, starting with `/explain-this`
- (grows once there's something real to add: `skills/`, `data/`, `knowledge-base/`, `mcp.json`, `hooks/`, `agents/`)

## Known pitfalls
- **Aapnar/Lenar direction can invert** in some SQL reports — found 2026-08-19 while checking our own database structure directly, not something anyone hit live yet. The column that means "giver" in one report can mean "taker" in another. Never state who gave or took stock without verifying the actual column meaning for that specific report first.

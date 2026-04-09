# EWS Charged-Off Account Recovery: Open Questions

_Last updated: April 8, 2026_

## Open (32)

| # | Category | Question | Owner | Status |
|---|----------|----------|-------|--------|
| 1 | Blocker | **How do we link payment to a specific charged-off account in the GL?** GL doesn't track individual charge-offs today (one big balance). Need to attribute payment to the correct account for reconciliation and EWS reporting. **Update (Apr 8):** GL #51615 is used for non-fraud charge-offs (per Niki). Not confirmed whether automatic 57-day charge-offs use the same GL. Follow-up needed. | Michelle Malavolta / Serena / Finance | In progress |
| 2 | Blocker | **What new data elements/fields are needed in the EWS contribution files for disputes?** Daniel Evanko needs to be engaged. Michelle to send data elements documentation. | Daniel Evanko / Michelle Barnwell / Shweta | Open |
| 3 | Payment | **Which payment rails?** **Update (Apr 8):** ACH push from external bank is now the **primary rail** per Michelle Malavolta's recommendation (safest, zero return risk, zero liability). Debit card via TabaPay is the **secondary rail** for instant confirmation and agent fallback (pending Michelle's approval for this use case). Payment-to-account mapping for ACH push solved by "Confirm My Payment" confirmation on the authenticated pay page. See PRD Recommended Solution section. | Tong Han / Anthony Zuffo / Shweta / Michelle Malavolta | In progress |
| 4 | Payment | Where does the source of truth for charged-off balances live? Core banking (Galileo)? Manual ledger? | Banking Eng / Finance | Open |
| 5 | Payment | How should Finance structure the recovery GL? GL codes? Precedent from SoFi Plus or Smart Card? | Finance / Michelle Malavolta | Open |
| 6 | Payment | What is the EWS integration mechanism for reporting a healed account? API or batch file? | Michelle Barnwell / EWS | Open |
| 7 | Payment | What is the EWS SLA for processing a healed record and confirming flag removal? | EWS | Open |
| 8 | Payment | ACH specifics: What tooling exists today? Where do ACH funds land on the GL for closed accounts? | Anthony Zuffo / Michelle Malavolta | Open |
| 9 | Payment | **TabaPay vs. Stripe?** Michael Bourgeois: "Don't stand up a new integration if one already exists." TabaPay appears to be what exists. | Tong Han / Payments Eng / Shweta | Open |
| 10 | Payment | Should any payment channels be explicitly disallowed for compliance, fraud, or ops reasons? | Compliance / Fraud / Michelle Barnwell | Open |
| 11 | Payment | Is there a dollar threshold below which SoFi should write off rather than collect? | Finance | Open |
| 12 | Payment | What happens if a member overpays or the recorded balance is wrong? | Finance / Ops | Open |
| 14 | Dispute | **What is the dedicated mailing address for the custom dispute mailbox?** Who provisions this? | Ops / Compliance / Joe Conlon | Open |
| 15 | Dispute | **What is the end-to-end dispute handling process?** What does Joe's team do manually vs. what does EPD build? | EPD / Joe Conlon / Mitchell Morris Jr. | Open |
| 16 | Dispute | What case management system will Joe's team use? Salesforce? Internal tool? | Ops Eng / Joe Conlon | Open |
| 17 | Dispute | Is there an existing FCRA dispute process at SoFi for credit products we can extend to deposits? | Compliance / Lending Ops | Open |
| 18 | Dispute | What compliance controls are needed for deadline adherence (30-day, 10-day, 5-day)? Tech vs. operational? | Compliance / EPD / Joe Conlon | Open |
| 19 | Cross-cutting | Can we update charge-off comms to mention EWS reporting? What disclosures need to change? | Compliance (Tatyana Pacheco) | Open |
| 20 | Cross-cutting | Can fraud-closed members still log in? Do they need a different flow? (Very low priority; fraud write-offs are ~0%.) | MACE Eng / Fraud | Open |
| 21 | Cross-cutting | Which engineering team owns this: MACE or Money Movement? | Dheeraj Mehta / Arjun Hegde | Open |
| 22 | Cross-cutting | Are existing disclosures sufficient for EWS data sharing, or do we need new ones? | Legal / Privacy (Sherrie Osborne) | Open |
| 23 | Cross-cutting | How will members contact SoFi to initiate healing? What's on the adverse action letter from other FIs? **Update (Apr 8):** Need to find out what the EWS adverse action letter looks like and whether it includes an identifier the member could use. | Product / Compliance / EWS / Shweta | In progress |
| 24 | Cross-cutting | Explicit Compliance sign-off on acceptable payment methods? | Compliance | Open |
| 25 | Cross-cutting | **What % of repayment does EWS see across participating FIs?** Industry benchmark is 5 to 15%. Volume dictates ops-only vs. self-serve MVP. | Michelle Barnwell / EWS | Open |
| 26 | Cross-cutting | **What % of reported charge-offs are disputed?** EWS estimates ~35 to 50/month for SoFi (~0.2 to 0.3% annually). CFPB data shows specialty CRA complaints up 124% YoY. | Michelle Barnwell / EWS | Open |
| 27 | Payment | **How do we solve payment-to-account mapping for ACH push?** If member pushes funds from their external bank to a general SoFi GL, how does SoFi attribute the payment to the right charged-off account? **Proposed solution:** "Confirm My Payment" confirmation on the authenticated pay page (sofi.com/pay). Member authenticates, initiates transfer, returns and confirms with date/amount/source bank. SoFi matches against incoming GL. See PRD Recommended Solution section for full analysis. Phase 2: unique temporary account numbers per member for auto-matching. | Shweta / Banking Eng | **New (Apr 8)** |
| 28 | Payment | **Can SoFi generate unique temporary account numbers for charged-off members?** This would be the cleanest way to accept push payments (ACH, wire, FedNow) and auto-map them to the right balance. Similar to "guest pay" concept. Need to investigate with banking engineering. | Banking Eng / Shweta | **New (Apr 8)** |
| 29 | Payment | **Is debit card via TabaPay approved for charge-off recovery (not just collections)?** Michelle Malavolta to confirm. If yes, this becomes the primary rail for self-service and agent-assisted flows. Collections already uses it, so precedent exists. | Michelle Malavolta / Tong Han | **New (Apr 8)** |
| 30 | Payment | **Can SoFi be set up as a bill pay destination?** If so, members could use their bank's bill pay to send funds with an account identifier. Risk: other bank may default to sending a check. Needs exploration from both SoFi side and external bank perspective. | Shweta / Banking Eng | **New (Apr 8)** |
| 31 | Communications | **What are the exact pre-closure notification trigger points during the 57-day window?** No pre-closure notifications exist today (confirmed Apr 8, Erica Hester). Need to define which days trigger Braze notifications (e.g., Day 7, Day 30, Day 50). Need Compliance input on what language is required and whether specific day thresholds have regulatory significance. | Erica Hester / Shweta / Compliance | **New (Apr 8)** |
| 32 | Communications | **What content must the pre-closure notifications include for UDAAP compliance?** At minimum: balance amount, days remaining, EWS consequences, how to deposit. Does Compliance require specific disclosures or safe-harbor language? Do we need to mention the right to dispute even before charge-off? | Compliance (Tatyana Pacheco) / Shweta | **New (Apr 8)** |

## Resolved (11)

| # | Question | Resolution | Source |
|---|----------|------------|--------|
| R1 | Partial payments or full only? | **Full balance only.** Not an EWS requirement to expand. | April 2026 alignment meeting |
| R2 | Who owns dispute operations? | **Joe Conlon + Mitchell Morris Jr. (Special Ops).** NOT Tammy Eisel, NOT Srikant Movva. | April 2026 alignment meeting |
| R3 | Dispute volume estimate? | **~35 to 50/month** (EWS estimate). No data on healing volume. | April 2026 alignment meeting + EWS call |
| R4 | What % of charge-offs are confirmed fraud? | **Effectively 0%.** Only 5 fraud write-offs all-time out of 224,863 total. Zero in the last 12 months. | Snowflake query, April 6, 2026 |
| R5 | Average charge-off balance? | **Avg $199, median $49.** DDA checking is 95%+ of volume. Savings rare (~40/month) but higher avg ($1,200). | Snowflake query, April 2, 2026 |
| R6 | Can we keep account closed during healing? | **Yes.** No need to reopen. | 3/12 meeting (Michelle Malavolta) |
| R7 | Acceptable payment types? | **Debit card (TabaPay) and ACH.** Check is NOT acceptable. **Update (Apr 8):** Michelle Malavolta further specified: electronic, ACH or wire, member-initiated push (not SoFi-originated). Debit card via TabaPay pending confirmation. | 3/12 meeting (Michelle Malavolta), Apr 8 Slack thread |
| R8 | Is receipt/confirmation required? | **Yes.** | 3/12 meeting (Brandi Morales-Espinal) |
| R9 | Must we allow new account opening after healing? | **No.** Not a requirement. | 3/12 meeting |
| R10 | Does EWS accept partial payment updates? | **No.** Full amount only. Update paid code to "P" with date when fully paid. | EWS input |
| R11 | Do we care about ease of payment for the member, or is safety paramount? | **Safety first, ease second.** Michelle Malavolta and Shahar Ronen both favor push-based methods (ACH push, wire) where SoFi bears no liability. Member convenience is secondary to eliminating return/chargeback risk. However, if debit card via TabaPay (no bounce risk, near-instant settlement) is approved, it offers a good balance of both. | Apr 8, 2026 meeting (Shahar Ronen, Shweta Kulshrestha) |

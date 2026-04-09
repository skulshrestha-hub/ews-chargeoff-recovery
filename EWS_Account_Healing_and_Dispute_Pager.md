# EWS Account Healing & Data Dispute

## Problem Statement

SoFi is contractually required by Early Warning Services (EWS) and by federal regulation (FCRA) to provide members a path to **repay charged-off deposit account balances** ("heal" their record) and **dispute data they believe is inaccurate** in the EWS National Shared Database. Today, no SoFi-owned experience or process exists for either workflow. This creates a compliance gap and leaves impacted members with no self-service mechanism to resolve negative account history that may prevent them from opening deposit accounts at other financial institutions.

---

## Background & Context

As part of SoFi's agreement with EWS, we contribute data to the **National Shared Database** — a shared industry resource used by financial institutions for deposit account screening and fraud prevention. When a SoFi Checking & Savings account is closed with a negative balance (charged off), that event is reported to EWS.

Once reported, the member is flagged in the National Shared Database. This flag can:
- Prevent the member from opening checking/savings accounts at other banks
- Remain on their record for up to **5 years**
- Only be removed if the debt is repaid ("healed") or the data is corrected through a dispute

**EWS requires contributing institutions to:**
1. Accept repayment of charged-off balances and report the account as "healed"
2. Provide a dispute mechanism for members who believe reported data is inaccurate
3. Resolve disputes within FCRA-mandated timelines (typically 30 days)

---

## Product Scope

| In Scope | Out of Scope |
|----------|-------------|
| SoFi Checking & Savings charged-off accounts reported to EWS | Loan or credit card charge-offs (reported to credit bureaus) |
| Member-initiated repayment of charged-off balances (one payment rail) | Proactive collections / outbound outreach |
| Member-initiated data disputes (mail for direct, EWS portal for indirect) | Fraud investigations unrelated to EWS |
| Updating source systems after healing/dispute resolution (feeds into Workstream 1 files) | EWS file generation and transmission (separate workstream: Daniel Evanko / Curtis) |
| Account status update and new member comms after repayment | Multiple payment channels (only one required for compliance) |
| Ops tooling for dispute investigation (Joe Conlon / Mitchell Morris Jr.) | Logged-out payment portal (removed from MVP unless it's the only option) |
| | "How members learn they owe" (handled by existing charge-off comms) |
| | Third-party debt collection workflows |

---

## What We Need to Build

The broader EWS initiative includes three consolidated workstreams (confirmed in team alignment meeting, April 2026). This pager covers workstreams 2 and 3. Workstream 1 is a separate effort led by Daniel Evanko and Curtis.

| # | Workstream | Owner | Scope |
|---|-----------|-------|-------|
| 1 | **EWS file generation and transmission** (includes charge-off reporting, account status updates, EWS integration) | Daniel Evanko (file generation), Curtis (transmission; Andy on leave) | Separate effort, not this PRD. Source system updates from healing/disputes feed into these files. |
| 2 | **Charge-off repayment (healing)** | Shweta Kulshrestha (EPD) | Repayment channels, repayment handling, payment to recovery GL, account status update, member comms. **This PRD.** |
| 3 | **Data dispute** | Joe Conlon / Mitchell Morris Jr. (Special Ops, operators); EPD TBD (build) | Dispute intake (mail only for direct; EWS portal for indirect), investigation, EWS data corrections. **This PRD.** |

### Workstream 2: Charge-Off Repayment (Healing)

Enable members to repay the outstanding balance on their charged-off Checking & Savings account so SoFi can report the account as "healed" to EWS and remove the negative flag.

**Key decisions (from April 2026 alignment meeting):**

- **Only ONE payment rail is required for EWS compliance.** EWS does not mandate multiple payment channels. As long as members have some way to pay, we are compliant. Michelle Barnwell: "I need one payment methodology."
- **Payment must clear near-instantly.** No chance of bouncing or being returned. This is a safety and compliance preference.
- **Full balance only.** No partial payments or payment plans. Confirmed for MVP; may not ever need a Phase 2 for payment expansion since expanding methods is a SoFi business decision, not an EWS requirement.
- **Do NOT build a portal** unless it is the only option. Michelle Barnwell's preference is to avoid portal scope expansion.
- **Payment type feasibility study needed.** Before committing, evaluate: pay by phone (easiest?), ACH from external bank (where would it land on the GL?), debit card. Pick the one that is easiest to build, easiest for the member, and safest for SoFi.

**Repayment channels (how the member initiates):**

| Channel | Notes |
|---------|-------|
| Phone (member calls in) | Most likely primary channel for MVP. Member calls, agent processes payment. |
| Mail (accidentally via dispute mailbox) | Members may mistakenly send payment through the dispute mail channel. Need a process to redirect. |
| EWS referral | Member contacts EWS after adverse action, EWS directs them to SoFi. Could come through any of the above. |

**Repayment handling (what happens after payment):**

| # | Requirement | Notes |
|---|-----------|-------|
| H1 | Look up a member's charged-off account and outstanding balance | Need a system of record. Where do charged-off balances live today? |
| H2 | Accept payment (full balance, one payment rail) | Feasibility study: which rail is easiest, safest, clears instantly? |
| H3 | Deposit collected funds into a SoFi recovery/collections ledger | Original account is closed. Finance to define the GL account. Open question: which GL does it land on? |
| H4 | Update member's account status in source systems | Status change gets picked up by Daniel's EWS file generation pipeline. |
| H5 | Send new communication to member confirming paid status | Letter or email confirming "your new status is [X]." |
| H6 | Generate a receipt / confirmation for the member | Email + in-app confirmation with amount, date, next steps. |
| H7 | EWS record updated via file generation pipeline | Healing status flows through Daniel/Curtis's file pipeline (Workstream 1). SoFi must update within 10 business days. |

**Open Questions, Healing:**

- [ ] **CRITICAL: Where do payments land when the account is closed?** The account is permanently closed. Need to identify the recovery GL. Need Michelle Malavolta + Serena to resolve. (Michael Bourgeois sync, April 2026)
- [ ] **Which single payment rail for MVP?** Feasibility study needed: pay by phone, ACH from external bank, debit card. Criteria: (1) easiest to build, (2) easiest for member, (3) safest (clears instantly, no bounce risk). Pick one.
- [ ] **Is ACH feasible for closed accounts?** If someone pays via ACH from an external bank, where does it land on the GL? (April 2026 alignment meeting)
- [ ] **Has Stripe been vetted?** Don't stand up a new integration if one doesn't already exist. What existing payment infrastructure can we reuse? (Michael Bourgeois sync)
- [ ] **Can we temporarily reopen accounts to accept payment?** Clunky but may be simplest MVP path. (Michael Bourgeois sync)
- [ ] Where does the source of truth for charged-off account data and balances live today? (Core banking? Collections system? Manual ledger?)
- [ ] What is the **EWS SLA** for processing a healed record and removing the flag?
- [ ] How does Finance want the **recovery ledger** structured? What GL codes apply?
- [ ] Are there **fee or interest** considerations on the charged-off balance? Or is it principal only?
- [ ] Is there a **dollar threshold** below which we should just write off vs. collect?
- [ ] What happens if a member **overpays** or the balance is wrong?
- [ ] **What % of charge-offs are confirmed fraud** prior to negative balance charge-off? Big charge-offs are likely true fraud and won't come back. (Michael Bourgeois sync)
- [ ] What does the **logged-out experience** look like, if any? Portal removed from MVP scope unless it's the only option.

---

### Workstream 3: Data Dispute

Enable members to dispute data reported to EWS that they believe is inaccurate, and ensure SoFi investigates and resolves the dispute within FCRA-mandated timelines.

**Key decisions (from April 2026 alignment meeting):**

- **Dispute operations owner: Joe Conlon and Mitchell Morris Jr. (Special Operations).** They handle the investigations and responses. NOT Tammy Eisel, NOT Srikant Movva. Joe and Mitchell are the customers/users of whatever EPD builds.
- **Direct data disputes must come via mail only.** Taking disputes over the phone is NOT FCRA-compliant. Must be a custom/dedicated mailbox.
- **Indirect disputes come via the EWS compliance portal.** EWS uploads disputes to their portal; Joe's team monitors and responds.
- **EPD's role is TBD.** Joe's team will be operating the process. EPD needs to figure out what to build (or whether an existing dispute framework can be leveraged).
- **Two possible outcomes:** (1) No error found: respond within timelines, notify member, no update. (2) Error found: update source systems, correct data flows into EWS files, notify member.
- **New data fields needed.** Dispute-related data elements must be added to the EWS contribution files. Daniel Evanko needs to be engaged on new fields.
- **Expected volume: ~35 to 50 disputes per month** (EWS estimate from April 2026 call).

**Dispute channels:**

| Channel | Type | Notes |
|---------|------|-------|
| Mail (custom/dedicated mailbox) | Direct dispute | FCRA requirement. Only compliant channel for direct disputes. |
| EWS compliance portal + email | Indirect dispute | Member submits to EWS directly. EWS uploads to their compliance portal. Joe's team monitors portal and email. Timelines apply from receipt. |

**Core Requirements:**

| # | Requirement | Notes |
|---|-----------|-------|
| D1 | Accept direct disputes via dedicated mail channel | Custom mailbox. FCRA requires written disputes. Phone intake is NOT compliant. |
| D2 | Monitor and process indirect disputes via EWS portal | Joe's team monitors EWS compliance portal. Must respond within FCRA timelines. |
| D3 | Investigate and resolve within FCRA timelines | 30 days from receipt (extendable to 45 if member provides additional info). |
| D4 | If error found: update member record in source systems | Corrected data feeds into EWS file pipeline (Workstream 1). Daniel Evanko needs new data elements built. |
| D5 | If no error found: respond to member with explanation | Specific language and notification requirements per FCRA. |
| D6 | Notify the member of the outcome (both cases) | Required by FCRA. Member must be informed of result and right to add a consumer statement if denied. |
| D7 | Flag disputed records in EWS contribution data during investigation | "Consumer Disputes Debt" indicator set while under investigation. Removed on resolution. |
| D8 | Maintain an audit trail of all disputes and resolutions | Regulatory requirement. Must be auditable. |

**Open Questions, Dispute:**

- [ ] **What does EPD need to build vs. what Joe's team handles manually?** Joe and Mitchell are the operators. We need to figure out: how do they update data if they confirm it needs to be updated? Where do they do that?
- [ ] **What new data fields are needed in the EWS contribution files for disputes?** Michelle to send data elements documentation. Daniel Evanko must be engaged.
- [ ] **Can we leverage an existing dispute framework** at SoFi rather than building net-new?
- [ ] What **case management system** will Joe's team use? (Salesforce, internal tool, spreadsheet today?)
- [ ] What does the **investigation workflow** look like? Who decides if a dispute is valid?
- [ ] What **data categories** can be disputed? (Balance amount, account ownership, charge-off date, fraud claim, wrong address, wrong name, etc.)
- [ ] What are the **FCRA notification requirements** at each stage? (Acknowledgment, in-progress, resolution)
- [ ] What is the **dedicated mailing address** for direct disputes? Needs to be a custom mailbox for this workstream.

---

## Member Experience

### How Members Reach This

**Healing (repayment):**

| Channel | MVP? | Notes |
|---------|------|-------|
| Phone (member calls SoFi) | Yes | Most likely primary channel. Member calls after adverse action from another FI. Agent processes payment. |
| In-app (SoFi app) | TBD | Depends on which payment rail is selected. May show balance + direct to phone. |
| Mail (accidentally) | Handle | Members may mail payment through the dispute channel by mistake. Need redirect process. |
| EWS referral | Handle | EWS tells member to contact SoFi. Member calls. |
| Logged-out portal (sofi.com/pay) | Removed from MVP | Michelle Barnwell: do not build a portal unless it's the only option. |

**How members learn they owe:** Out of scope for this project. Members learn through the existing charge-off notification process (57-day reminders, closure email) and from adverse action letters when denied at other FIs. No new proactive outreach in MVP.

**Data dispute:**

| Channel | MVP? | Notes |
|---------|------|-------|
| Mail (custom/dedicated mailbox) | Yes, required | ONLY compliant channel for direct disputes per FCRA. |
| EWS compliance portal (indirect) | Yes | Member disputes to EWS directly. EWS routes to Joe's team via portal. |
| Phone | **No, not FCRA-compliant** | Cannot take data disputes over the phone. PRD previously listed phone; this is corrected. |

### Key UX Considerations

- Members may not understand what EWS is or why they're flagged. **Clear, empathetic copy** is critical.
- The healing flow should feel like a **resolution, not a penalty**. Members are trying to do the right thing.
- Dispute flow must communicate **timelines and status** clearly (FCRA requires specific notifications).
- Consider: should healed members be offered the chance to **re-open a SoFi account**?

---

## Technical & Integration

### Systems Involved

| System | Role | Owner |
|--------|------|-------|
| Core Banking (Galileo) | Source of original account data, charge-off records | Banking Eng |
| EWS API / File Interface | Submit healed records, dispute corrections | TBD — confirm integration method |
| Payment Processing | Accept repayment via ACH, debit, internal transfer | Payments Eng |
| General Ledger | Recovery/collections ledger for received funds | Finance |
| Case Management (Salesforce?) | Dispute tracking, investigation workflow | Ops Eng |
| Member Communications | Email, push, in-app notifications | Growth / Comms Eng |
| SoFi App + Web | Member-facing healing and dispute flows | Product Eng |

### Key Integration Questions

- [ ] What is the **EWS reporting format**? (API, SFTP batch file, manual portal?)
- [ ] What is the **EWS reporting frequency**? (Real-time, daily batch, weekly?)
- [ ] Do we have an **existing EWS integration** for the initial charge-off reporting that we can extend?
- [ ] What is the **Galileo data model** for charged-off accounts? Can we query balances programmatically?
- [ ] Do we need a **new microservice** for this, or can it be added to an existing service?

---

## Stakeholders

| Team / Person | Role in This Initiative |
|------|----------------------|
| **Shweta Kulshrestha** (Product) | PRD owner; charge-off repayment (healing) and GL application workstreams |
| **Michael Bourgeois** (DRI Approver) | Final sign-off |
| **Shahar Ronen** (Product / EM) | Executive sponsor, escalation path |
| **Michelle Barnwell** (Money BU) | BRD owner; overall EWS Data Contributions initiative |
| **Daniel Evanko** (Data Engineering) | EWS file generation pipeline. Needs to be engaged on new data elements for disputes. |
| **Curtis** (File Transmission) | EWS file transmission (Andy on leave). Ensures files get delivered to EWS. |
| **Joe Conlon** (Special Operations) | Dispute operations owner. His team handles direct and indirect dispute investigations and responses. Customer/user of whatever EPD builds. |
| **Mitchell Morris Jr.** (Special Operations) | Dispute operations. Works with Joe Conlon on dispute handling. |
| **Michelle Malavolta** (Finance / Operations) | Payment acceptance and GL application for closed accounts. Critical consultation needed. |
| **Serena** (Banking Eng / Core Conversion) | Core conversion knowledge; key contact for how payments can be accepted and ledgered for closed accounts |
| **David Pinsky** (Banking Eng / Core Conversion) | Core conversion changes. Include if available; Serena can proxy. |
| **Compliance / Legal** | FCRA requirements, EWS contract obligations, dispute process rules |
| **Finance** | Recovery ledger design, GL codes, tax implications (1099-C) |
| **Ops / Customer Support** | Agent-assisted healing flows, training |
| **Engineering** | Build payment flow, dispute tooling, source system updates |
| **Design** | Member-facing UX for healing |
| **Fraud / Risk** | Edge cases: fraudulent disputes, identity verification |
| **EWS (external)** | API/integration specs, SLAs, testing/certification |

---

## Proposed Phasing

### Phase 0: Discovery (Weeks 1–3)
- Confirm regulatory requirements with Compliance/Legal
- Map existing systems and data (where do charged-off records live?)
- Get EWS integration specs and SLAs
- Identify ops team and tooling for disputes
- Quantify: how many members are impacted? What's the total balance outstanding?

### Phase 1: MVP — Ops-Assisted (Weeks 4–8)
- Agent-assisted healing: member calls in, agent looks up balance, processes payment manually
- Agent-assisted dispute: member calls/writes in, agent creates case, investigation via internal tooling
- EWS reporting: manual or semi-automated submission
- Goal: close the compliance gap quickly

### Phase 2: Self-Serve (Weeks 9–16)
- In-app and web flow for members to look up balance and pay
- Online dispute submission form with document upload
- Automated EWS reporting upon payment / dispute resolution
- Member notifications at each stage

### Phase 3: Optimization (Ongoing)
- Proactive outreach to eligible members (email, push)
- Payment plans / partial payments
- Analytics dashboard: healing rate, dispute volume, resolution time
- Re-engagement: offer healed members a path back to SoFi

---

## Success Metrics

| Metric | Definition | Target |
|--------|-----------|--------|
| **Compliance coverage** | % of required EWS workflows implemented | 100% by Phase 1 |
| **Healing rate** | % of eligible members who repay and heal their record | TBD — baseline in Phase 1 |
| **Dispute resolution time** | Median days from dispute submission to resolution | ≤ 30 days (FCRA) |
| **Dispute accuracy** | % of disputes resolved correctly on first pass | > 95% |
| **Recovery amount** | Total $ collected through healing payments | Track from Phase 1 |
| **Member satisfaction** | CSAT / NPS for members who go through healing or dispute | TBD |
| **Self-serve adoption** | % of healing/dispute cases handled without agent involvement | > 50% by Phase 3 |

---

## Risks & Mitigations

| Risk | Impact | Mitigation |
|------|--------|-----------|
| FCRA violation (missed dispute timelines) | Regulatory action, fines | Automated SLA tracking, escalation alerts |
| No clear system of record for charged-off balances | Can't build healing flow | Discovery Phase 0 — identify and confirm data source |
| EWS integration complexity / long certification timeline | Delays launch | Start EWS integration discovery in Week 1 |
| Low member awareness of healing option | Low adoption, compliance risk if EWS requires proactive outreach | Proactive communication plan in Phase 3 |
| Fraudulent disputes | Financial loss, abuse of process | Identity verification, fraud review in dispute workflow |
| Finance/tax implications of recovery payments | Incorrect reporting | Engage Finance early; confirm 1099-C / write-off treatment |

---

## Next Steps

### Immediate priorities (from Michael Bourgeois sync, April 2026)

1. **ASAP: Payment infrastructure meeting with Michelle Malavolta + Serena** (+ David Pinsky if available)
   - This is the #1 blocker. Must resolve: how does SoFi accept a payment and ledger it when the member's account is permanently closed?
   - Debit card and ACH both have the same fundamental problem: where does the payment get attributed?
   - Michael: "Better to have these vetted before we get too far down the path of how we want this to work."
   - If ACH is too complicated, drop it and do debit only. If debit is too complicated, explore temporarily reopening accounts.

2. **Dispute workstream kickoff with Tammy Eisel**
   - Michael clarified this is an FCRA data dispute (disputing data reported to EWS), not a transaction dispute.
   - Routes to Tammy Eisel's team (not Srikant Movva's).
   - Need to understand existing FCRA dispute processes and what can be extended.

3. **Overall initiative alignment: Shweta + Michael Bourgeois + Michelle Barnwell + Shahar Ronen**
   - Align on timing, penalty pathway, drop-dead dates, and how workstreams fit together.
   - Include Shahar since he's overseeing the project.
   - Discuss timeline flexibility (e.g., $99K vs. $100K monthly discount negotiation).

### From April 2026 team alignment meeting

4. **Meet again this week (longer session, ~1 hour).** Knock down all open questions. Have Neil/Crystal schedule.
5. **Get Joe Conlon, Mitchell Morris Jr., and other dependencies in one room.** Finalize dispute requirements with the actual operators.
6. **Shweta to access EWS Data Contribution Slack channel files.** Two dispute process files, operating roles, and tech spec are in the Files tab.
7. **Shweta to reach out to contacts from Mili's office hour session.** Follow-up on leads and feedback received.
8. **Engage Daniel Evanko on new data elements** needed for dispute flags in the EWS contribution files. He needs to know where to pick up new fields.
9. **Payment rail feasibility study.** Determine which single rail (phone/debit card/ACH) is easiest, safest, and clears instantly. Cross off any that shouldn't be done for compliance/risk reasons.

### Previously identified (still relevant)

10. **Identify data owner** -- who owns charged-off account data today? Where does it live?
11. **Request EWS integration documentation** -- API specs, file formats, certification process
12. **Quantify fraud overlap** -- what % of charge-offs are confirmed fraud prior to negative balance charge-off?
13. **Align with Finance** -- recovery ledger design, GL codes, tax treatment
14. **Draft detailed PRD** -- expand this pager into full requirements after discovery answers come back (full PRD draft already exists at `PRD_EWS_Charged_Off_Account_Recovery.md`)

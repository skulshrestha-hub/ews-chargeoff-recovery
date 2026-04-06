# EWS Charged-Off Account Recovery: Payment Collection & Data Dispute

**Author:** Shweta Kulshrestha
**Last updated:** 2026-04-02
**Status:** Draft
**Product area:** Money
**Parent initiative:** EWS Data Contributions (BRD owner: Michelle Barnwell)
**DRI Approver:** Michael Bourgeois

**Key references:**
- [EWS Data Contribution BRD](link) (Michelle Barnwell)
- [EWS Data Contrib Project Plan](link) (Michelle Barnwell)
- [EWS Data Contribution: Charge Off Healing and FCRA Disputes](link) (working doc)

**Stakeholders:**

| Name | Role | Responsibility |
|------|------|---------------|
| Shweta Kulshrestha | Product | PRD owner; payment collection and GL application workstreams |
| Shahar Ronen | Product / EM | Executive sponsor, escalation path |
| Michelle Barnwell | Money BU | BRD owner; EWS file generation and reporting workstream |
| Michael Bourgeois | DRI Approver | Final sign-off |
| Mili Mittal | Engineering Leadership | Strategic input, reuse assessment guidance |
| Srikant Movva (TBD) | Product | Dispute workstream owner (to be confirmed) |
| Dheeraj Mehta / Arjun Hegde | Engineering | Eng partner assignment (MACE vs. Money Movement) |
| Ragini | Ops Engineering | Ops component engineering |
| Daniel Evanko | Data Engineering | EWS file generation and transmission pipeline |
| Tami Wilson | Operations | Dispute process design and operational readiness |
| Joe Conlon | Specialty Ops | Ops process design, dispute handling procedures |
| Nikita Shah / Erica Hester | MET Operations | Agent training, dispute investigation |
| Tatyana Pacheco | Compliance | FCRA requirements, EWS contract obligations |
| Kristen Krikorian | Compliance (Director) | Escalation; dispute process rules |
| Brandi Morales-Espinal | Compliance (Director) | Disclosure review |
| Derek Fisher | Compliance | Operational compliance |
| Dan Rosner / Arpit Ajmani | Fraud Strategy | Fraud-closed account handling, bad actor mitigation |
| Sophia Carlton, Dana Wolfe, Diana Barker, Irene Morley | 2nd LOD | Fraud / Ops / Tech oversight |
| Krista Thorsen | Operations Risk | Operational risk assessment |
| Sandeep Jayashankar | Cyber Security | Payment flow security, PCI DSS |
| Sherrie Osborne | Privacy | Data sharing review, PIA assessment |
| Finance (TBD) | Finance | Recovery GL design, GL codes, tax implications |
| Design (TBD) | Design | Member-facing UX for healing and dispute |

**Workstream ownership:**

| Workstream | Owner |
|-----------|-------|
| Report to EWS within N days of charge-off | Michelle Barnwell |
| Collect the payment | Shweta Kulshrestha |
| Apply payment to the account / GL | Shweta Kulshrestha |
| Enable members to dispute and trigger investigation | Srikant Movva (TBD) |

---

## TL;DR

Currently, members with charged-off SoFi Money accounts have no way to repay, yet these charge-offs are reported to EWS, blocking them from opening bank accounts for up to 5 years. This creates regulatory risk (UDAAP/FCRA) since we lack required payment and dispute pathways. By **July 31, 2026**, we will launch an MVP enabling in-app balance visibility, debit card repayment (via Stripe), and EWS updates within 10 business days to clear flags, along with dispute support across app, phone, and mail. Full EWS data contribution compliance is required by **December 31, 2026** to avoid loss of New Account Risk access.

---

## Background

### What is EWS?

Early Warning Services (EWS) operates the **National Shared Database**, a shared industry resource used by financial institutions for deposit account screening and fraud prevention. EWS is operated by a consortium of the largest U.S. banks. Its products, including **New Account Risk**, help detect identity fraud and first-party fraud during account opening. Risk conducted a data study and believes New Account Risk would help reduce approval rate by 6 to 7% and overall fraud losses by 30%.

### SoFi's relationship with EWS

SoFi uses EWS's New Account Risk product for onboarding risk scoring. Today, we send C&S application data to EWS and they return risk elements used by Fraud Strategy for approve/deny decisions. Our contract requires us to **contribute data back** to the National Shared Database. Normally, participants must contribute data before using the service, but SoFi received a waiver to complete this work no later than **December 31, 2026**.

The pricing structure incentivizes contribution: **$2.9M for 3 years** as a non-contributor vs. **$1.5M** as a contributor. Until we deliver the first set of contribution files, we are paying approximately **$100k/month** in excess fees. If we fail to meet the December 31, 2026 deadline, EWS has the right to turn off our access to New Account Risk and any other services we use.

### What data must we contribute?

Per the National Shared Database Operating Rules, SoFi must contribute (daily, except weekly for the Customer Contribution File):
- Account Owner Elements
- Account Status Data (including charge-offs, closures, delinquency dates)
- Item-level Data
- Non-participant Data
- Fraud Data
- Account Abuse Data
- Account Balance Data (90 days)
- ACH Data
- Shared Fraud Data

The "Hot File" (a real-time alert feed that flags accounts involved in active fraud to other banks as it happens, rather than waiting for the next daily batch) is optional per the Operating Rules and out of scope.

### Why healing and dispute matter

Once we begin reporting charge-offs to EWS, two obligations kick in:

1. **Healing (UDAAP requirement).** We cannot report a member as having a charged-off account if we do not allow them to repay and clear that record. Doing so would cause consumer harm with no recourse, which is the definition of unfair under UDAAP. The member has **up to 5 years** from when the charge-off is reported to EWS to pay and clear the flag (after 5 years the flag expires automatically). Once the member pays, SoFi must update EWS within **10 business days** per the Operating Rules.

2. **Dispute (FCRA requirement).** As a data furnisher to a consumer reporting agency (EWS), SoFi is subject to FCRA furnisher obligations. Members have the right to dispute any data we report (wrong balance, wrong address, not closed due to fraud, account not theirs, etc.) at any point while the record exists, up to **5 years**. Once a dispute is filed, SoFi must investigate within **30 days** (45 if additional info is provided), correct inaccuracies, and propagate corrections to all CRAs that received the data. Frivolous disputes must be identified and the member notified within **5 business days**.

### How charge-offs work today

When a member's C&S account goes negative, SoFi provides a **57-day grace period** with reminders to repay. If the member does not bring the balance to zero within that window, the account is closed and the balance is charged off. After charge-off:
- The account is **permanently closed**. There is no mechanism to reopen it or deposit into it.
- There is **no way for the member to repay**. Even if they want to, no payment channel exists.
- If the member reapplies for a SoFi Money account in the future, they are **declined**.
- Once we begin reporting to EWS, the charge-off will appear in the National Shared Database, potentially blocking the member from opening accounts at other financial institutions for up to 5 years.

### Why this matters

**Compliance exposure.** Reporting charge-offs to EWS without offering a healing or dispute path creates UDAAP risk (unfair/deceptive to report negative data with no member recourse) and FCRA non-compliance (members have a statutory right to dispute furnished data). The BRD explicitly flags this: "To address UDAAP risk if we report charge-offs but don't allow members to resolve those charge-offs."

**Member harm.** An EWS flag can prevent a member from opening checking or savings accounts at other financial institutions for up to **5 years**. Members who want to make things right currently have no way to do so.

**Financial impact.** SoFi is paying approximately **$100k/month** in excess fees to EWS until we begin contributing data. Full contribution compliance (including healing and dispute capabilities) is required by **December 31, 2026**, or EWS can terminate our access to New Account Risk and other services. The pricing difference is significant: $2.9M (non-contributing) vs. $1.5M (contributing) over 3 years.

**Contract obligation.** Per the EWS Operating Rules, if a consumer pays off or otherwise settles Account Abuse, the bank must report the update within **10 business days**. We cannot fulfill this obligation without a payment collection mechanism.

### The broader initiative

This PRD covers the **payment collection and dispute** workstreams of the larger EWS Data Contributions initiative. The full initiative includes:

| Workstream | Owner | Status |
|-----------|-------|--------|
| EWS file generation and data contribution | Michelle Barnwell / Daniel Evanko (Data Eng) | In progress |
| Report charge-offs to EWS within N days | Michelle Barnwell | In progress |
| **Collect healing payment from member** | **Shweta Kulshrestha** | **This PRD** |
| **Apply payment to recovery GL** | **Shweta Kulshrestha** | **This PRD** |
| **Enable member dispute and investigation** | **Srikant Movva (TBD)** | **This PRD** |
| Update account status and report to EWS | Shared (depends on payment/dispute outcome) | This PRD |
| Compliance, disclosures, policies and governance | Compliance team | Parallel |

---

## MVP: What Must Ship by July 31, 2026

SoFi is paying **~$100K/month in excess EWS fees** because we are not contributing data. We cannot begin contributing until healing and dispute capabilities exist. Every month we delay costs $100K. Full compliance is required by **December 31, 2026**, or EWS can terminate our access to New Account Risk entirely.

The MVP is the minimum set of capabilities that allows us to start contributing charge-off data to EWS without creating regulatory exposure.

### What the MVP must include

| # | Capability | Why it's required | What "done" looks like |
|---|---|---|---|
| 1 | **Balance lookup** | Members must be able to see what they owe. | Logged-in: charged-off balance card in Banking tab. Logged-out: guest portal balance lookup at sofi.com/pay. |
| 2 | **Payment collection (debit card via Stripe)** | UDAAP: cannot report charge-offs without a repayment path. | Member pays full balance via debit card. Payment routed to recovery GL. Works for logged-in (in-app) and logged-out (guest portal). |
| 3 | **Pay by phone (agent-assisted)** | Members who cannot use the app or guest portal must still be able to pay. | Agent looks up account, collects debit card info over phone, processes payment via agent-facing tool. Same Stripe backend. |
| 4 | **EWS healing report** | Contractual: must update EWS within 10 business days of payment. | Healed account status submitted to EWS via batch file or API within 10 business days. |
| 5 | **Dispute intake (phone + mail)** | FCRA: members have a statutory right to dispute furnished data. | Member can file dispute by phone (1-855-456-7634) or mail. Agent logs in case management. Investigation completed within 30 days. |
| 6 | **Dispute flags in EWS data** | EWS Operating Rules: disputed records must be flagged while under investigation. | "Consumer Disputes Debt" indicator set in contribution file during investigation. Removed on resolution. |
| 7 | **Updated member communications** | FCRA/UDAAP: members must know about EWS reporting and how to resolve it. | Charge-off notices, adverse action letters, and pre-charge-off reminders include EWS language, sofi.com/pay URL, and dispute contact info. |

### MVP user stories and requirements

MVP is split into two milestones. **Milestone 1 (Healing):** members can see and pay what they owe. **Milestone 2 (Dispute):** members can challenge inaccurate data (required by FCRA before we furnish to EWS). Both can be developed in parallel; both must ship by July 31, 2026.

#### Milestone 1: Account Healing / Repayment

| # | Category | Requirement | User Story |
|---|---|---|---|
| 1 | Balance Visibility | Member can view charged-off balance (in-app) | As a member, I want to see how much I owe so I can decide whether to repay. Balance card in Banking tab shows amount, closure date, and EWS status. |
| 2 | Balance Visibility | Member can view charged-off balance (guest portal) | As a member without app access, I want to look up what I owe online. sofi.com/pay lookup by reference number + last 4 SSN, or name + DOB + SSN. |
| 3 | Payment | Member can pay full balance via debit card (in-app) | As a member, I want to pay my charged-off balance so my EWS record is cleared. Stripe debit card payment, full balance only, routed to recovery GL. |
| 4 | Payment | Member can pay full balance via debit card (guest portal) | As a member without app access, I want to pay online without logging in. Same Stripe payment on sofi.com/pay, email receipt, no login required. |
| 5 | Payment | Member can pay via phone (agent-assisted) | As a member, I want to pay over the phone if I can't use the app or website. Agent collects debit card, processes via agent tool, same Stripe backend. |
| 6 | Payment | Member can pay via ACH push (guest portal / in-app) | As a member, I want to pay via ACH from my external bank. ACH push from external bank accepted, routed to recovery GL. |
| 7 | Confirmation | Payment confirmation and receipt (all channels) | As a member, I want a receipt confirming my payment and what happens next. On-screen confirmation + email with amount, date, confirmation number, next steps. |
| 8 | EWS Reporting | EWS record updated after payment (backend) | As SoFi, when a member repays, we must update EWS within 10 business days. Healed status submitted via batch file or API. |
| 9 | Notification | Member notified when EWS record is cleared (email) | As a member, I want to know when my EWS record has been cleared. Email sent after EWS confirms update. |
| 10 | Ops Tooling | Agent can look up balance and process payment (internal) | As an ops agent, I want to look up a member's charged-off account and process payment on their behalf. Agent searches by member identifier, views balance, initiates payment. |
| 11 | Communications | Pre-charge-off emails include EWS language (email) | As SoFi, pre-charge-off emails must warn members about EWS consequences. All 57-day reminders mention EWS consequences and sofi.com/pay. |
| 12 | Communications | Charge-off notification includes sofi.com/pay (email) | As SoFi, the charge-off notification must tell members how to pay. Day 57 email includes payment URL, dispute phone/mail info. |
| 13 | Communications | Adverse action notices include EWS contact (letter) | As SoFi, adverse action notices must include EWS contact info per FCRA Section 615(a). |
| 14 | Ops Readiness | Agent training and SOPs (internal) | As SoFi, agents must be trained on healing flows before launch. Agents trained on payment processing and identity verification. |
| 15 | Audit | Audit trail for all healing actions (internal) | As SoFi, all healing actions must be logged for compliance. All actions logged and auditable. |

#### Milestone 2: Data Dispute

| # | Category | Requirement | User Story |
|---|---|---|---|
| 1 | Dispute Intake | Member can file dispute by phone (phone) | As a member, I want to dispute data SoFi reported to EWS that I believe is wrong. Agent captures disputed data element, reason, supporting docs. Case number provided. |
| 2 | Dispute Intake | Member can file dispute by mail (mail) | As a member, I want to submit a written dispute. Written dispute accepted at SoFi Bank, N.A., 2750 East Cottonwood Parkway #300, Cottonwood Heights, UT 84121. |
| 3 | Dispute Intake | Dispute information discoverable (all channels) | As a member, I want to learn about my right to dispute from the app or portal. Dispute phone number and mailing address accessible from in-app, guest portal, help center, and agent scripts. |
| 4 | Investigation | Investigation completed within 30 days (ops) | As SoFi, we must investigate and resolve disputes within FCRA timelines. Member notified of outcome. Corrected records propagated if dispute upheld. 45 days if additional info provided. |
| 5 | Investigation | Frivolous dispute determination (ops) | As SoFi, if a dispute is frivolous, we must notify the member within 5 business days with reasons and missing info. |
| 6 | Investigation | Indirect dispute handling (ops) | As SoFi, when we receive an indirect dispute via EWS/CRA, we must investigate, respond with results, and propagate corrections to all nationwide CRAs. |
| 7 | EWS Reporting | Dispute flag set in EWS contribution data (backend) | As SoFi, disputed records must be flagged in EWS data during investigation. "Consumer Disputes Debt" indicator set during investigation, removed on resolution. |
| 8 | EWS Reporting | Corrected data submitted to EWS and all CRAs (backend) | As SoFi, upheld disputes must be corrected with EWS and all CRAs within required timeframe. |
| 9 | Member Rights | Consumer statement right communicated (ops) | As a member, if my dispute is denied, I want to know why and my right to add a personal statement to the record. |
| 10 | Ops Readiness | Agent training and SOPs for disputes (internal) | As SoFi, agents must be trained on dispute intake, categories, and FCRA timelines before launch. |
| 11 | Audit | Audit trail for all dispute actions (internal) | As SoFi, all dispute activity (intake, investigation, outcome, notifications) must be logged and auditable. |

### MVP implementation details

**Phone number for payments and disputes:**
- **1-855-456-7634** (existing SoFi support line). Agents route charge-off callers to the recovery queue. This number appears on all charge-off communications, the guest portal, the in-app dispute screen, and adverse action letters.

**Payment confirmation number format:**
- Pattern: `EWS-YYYY-NNNNN` (e.g., `EWS-2026-04893`)
- Generated on successful payment. Appears on-screen, in the email receipt, and is used as the member's proof of payment. Agent can look up a case by this number.

**Reference number for guest portal (logged-out balance lookup):**
- Pattern: `REF-XXXXXXXXXX` (e.g., `REF-7823A04291`)
- Generated at charge-off and included in the Day 57 closure email and any subsequent letters. The member enters this + last 4 SSN on sofi.com/pay to pull up their balance without logging in. If the member does not have a reference number, they can verify identity with full name + DOB + last 4 SSN instead.

**What is needed to receive a payment (all channels):**

| Field | In-App | Guest Portal | Phone (Agent) |
|---|---|---|---|
| Identity verification | Already authenticated | Reference # + last 4 SSN, or name + DOB + last 4 SSN | Name + DOB + last 4 SSN (agent verifies) |
| Payment method | Debit card number, expiry, CVV | Debit card number, expiry, CVV, email for receipt | Debit card number, expiry, CVV (agent enters), email for receipt |
| ACH (secondary) | External bank routing + account number | External bank routing + account number | Not available via phone for MVP |

**Where does the agent process payments?**
- Agents use an internal admin tool (to be built or extended from existing tooling, e.g., Salesforce or internal ops portal). The tool allows the agent to: (1) search by member name, SSN, or reference number, (2) view charged-off balance and account details, (3) enter debit card info and process payment via the same Stripe backend, (4) generate confirmation number and trigger receipt email. The agent does not handle funds directly; Stripe routes to the recovery GL.

### What the MVP does NOT include

Partial payments, payment plans, credit card payments, self-service dispute forms, proactive outreach, PayNearMe, account revival, collections, or analytics dashboards. These are Phase 2.

---

## Phase Summary

| Area | Phase 1, Milestone 1: Account Healing / Repayment | Phase 1, Milestone 2: Data Dispute | Phase 2: Full Flow (Dec 31, 2026) |
|---|---|---|---|
| **Goal** | Enable members to pay charged-off balances and clear EWS record | Enable members to dispute inaccurate data reported to EWS | Scale, automate, and expand all capabilities |
| **Payment channels** | In-app debit card (Stripe), logged-out portal (sofi.com/pay), ACH from external bank, pay by phone (agent) | N/A | + PayNearMe (CC, Venmo, cash, Apple Pay) |
| **Payment types** | Full balance only | N/A | + Partial payments, payment plans, settlements |
| **Dispute channels** | N/A | Phone + mail (agent-assisted) | + Self-service in-app/web form with document upload |
| **Dispute handling** | N/A | Manual investigation via case management | + Automated SLA tracking, escalation alerts, analytics dashboard |
| **How members learn they owe** | Charge-off email, in-app card, agent, help center, adverse action letter | Same channels include dispute rights info | + Proactive push notifications, email campaigns |
| **EWS integration** | Batch file or API for healing reports | + Dispute flags and consumer statements in contribution data | + Full data contribution (all file types, governance, attestation) |
| **Logged-out portal (sofi.com/pay)** | Balance lookup + pay (reference number or identity verification) | + "Think this is wrong?" dispute instructions page | + Payment history, dispute status tracking |
| **Analytics** | Basic event tracking (payment volume, healing rate) | + Dispute volume, resolution time tracking | + Full dashboard: healing rate, dispute volume, resolution time, recovery $, ops handle time |
| **Ops readiness** | Agent training on payment processing, identity verification | + Agent training on dispute intake, FCRA timelines, categories | + Automated workflows, reduced manual effort |
| **Account re-engagement / revival** | Not included | Not included | Evaluate path back to SoFi for healed members; reopen accounts if new core supports it |

---

## Data Landscape

### Snowflake: Where charge-off data lives today

| Table | Description | Key Columns | Relevance |
|-------|------------|-------------|-----------|
| `TDM_FRAUD_AML_MEMBER360.CLEANSED.MONEY_ACCOUNT_CHARGE_OFF_OCCURRED` | **Primary charge-off event table.** Fires when a banking account is negative and balance < -$10.00. | `ACCOUNTNUMBER`, `PARTYID`, `EVENTCREATIONDATE`, `DATASOURCE` (KAFKA for real-time from 2026-01-01; TDM_BANK for historical backfill) | **This is likely the system of record for identifying which accounts have been charged off.** Can be used to look up a member's charged-off account and event date. Owner: FROST Data Eng (`frost_de@sofi.org`). |
| `TDM_MONEY.CLEANSED.ACCOUNT_TO_CLOSE_EVENTS` | Account closure records with delinquency type. | `PARTY_ID`, `SURROGATE_ID`, `DELINQUENCY_TYPE` (CHARGEOFF, WRITEOFF), `STATUS` (started/pending/closed/canceled), `CLOSING_REASON_CODE`, `NOTES` | Identifies whether an account closure was due to charge-off vs. write-off vs. other reasons. Owner: SIPS Data Eng (`devanko@sofi.org`, i.e., Daniel Evanko). |
| `TDM_MONEY.MODELED.FACT_ACCOUNT_V2` | **Core daily fact table.** Comprehensive 360-degree view of each account with lifecycle dates, status, and balance metrics. Updated daily. | `ACCOUNT_ID`, `ACCOUNT_STATUS`, `ACCOUNT_CLOSED_DATE`, `ACCOUNT_CLOSED_REASON`, `CASH_BALANCE`, `BALANCE_LEDGER`, `PRODUCT_TYPE_DESC`, `ACCOUNT_OWNER`, `USER_ID` | **Use this to determine the outstanding balance at time of charge-off.** Filter by `PRODUCT_TYPE_DESC` in ('SoFi Money Cash Account', 'SoFi Bank DDA', 'SOFI BANK SAVINGS'). Owner: Data Science (`rreilly@sofi.org`). |
| `TDM_RISK_MGMT_HUB.SUMMARIZED.rr_banking_transactions_details` + `rr_banking_transactions_facts` | Transaction-level data including charge-off transactions. | Charge-off transaction codes: `DDWRTOFF`, `DDCHGOFF`, `DDFRDWO`, `SDCHGOFF`, `SDWRTOFF`, `SDFRDWO` | Can identify the exact charge-off amount per account. Existing queries already use these tables for daily/MTD/QTD/YTD charge-off reporting. |
| `TDM_MONEY.SUMMARIZED.CP_BALANCE_LOAN__ACCOUNT_NUMBER` | Balance and loan info per account. | `NEGBAL_WRITEOFF_AMOUNT` | Tracks the amount written off for negative balances. |
| `TDM_RISK_MGMT_HUB.CLEANSED.SAFE_DOCUMENT_EWS` | EWS data from SAFE decision requests. Contains responses from EWS during account screening. | `EWS_SCORE`, `EWSID`, `EWSREFERENCE_ID`, `USERID`, `EWSSHAREDFRAUD_RECORDS`, `EWSACCOUNTDEFAULTSCORE_STATUS` | **This is the inbound EWS data** (what we receive). Will need a corresponding outbound table for data we contribute. Can be used to track how many prospective members are flagged by EWS. |

### Existing Snowflake queries and analysis patterns

From query history, Risk Management already runs comprehensive charge-off monitoring that provides useful baselines:

- **Negative balance aging** is tracked in buckets: 1 day, 2 to 7 days, 8 to 15 days, 16 to 30 days, 31 to 45 days, 46 to 60 days, 60+ days
- **Charge-off amounts are segmented**: <= $50 and > $50 (confirms the hypothesis that many charge-offs are small)
- **Charge-off metrics** are computed daily, MTD, QTD, and YTD
- **Data is split by account owner**: SoFi Money, SoFi Bank, and Samsung
- **Product types** filtered: 'SoFi Money Cash Account', 'SoFi Bank DDA', 'SOFI BANK SAVINGS'

### Amplitude: Relevant events

Searched in **MrClean Prod v2** (appId 250200), the primary project for product analytics events.

| Event | Key Properties | Relevance |
|-------|---------------|-----------|
| `ACCOUNT_CLOSED` | `account_type`, `source_product`, `source_service_name`, `backend` | Tracks all account closures across products. Can be filtered by `account_type` to isolate Money closures. |
| `BANKING_ACCOUNT_ENDED_DAY_WITH_NEGATIVE_BALANCE` | `amount` (the negative balance), `date`, `source_product`, `source_service_name` | **Directly relevant.** Tracks accounts ending the day with a negative balance. This is the leading indicator for future charge-offs (57-day pipeline). Use `amount` to size the upstream funnel. |
| `MONEY_ACCOUNT_CLOSED` | `close_reason_code`, `party_id`, `external_account_id`, `source_product` | **Money-specific closure event.** The `close_reason_code` can distinguish charge-off closures from voluntary/other closures. More targeted than the generic `ACCOUNT_CLOSED`. |

**Existing Amplitude charts of interest:**
- [Account Closure](https://app.amplitude.com/analytics/sofi/chart/0dnpcziw) (events segmentation, including IRA and Taxable)
- [New Account Closures View](https://app.amplitude.com/analytics/sofi/chart/ezxcoin) (events segmentation, Money-focused)
- [Bank Evergreen Balance Movement](https://app.amplitude.com/analytics/sofi/dashboard/cwjdynx9) (dashboard, 19 charts, tracks balance trends)

**What's missing in Amplitude:** There are currently no events for charge-off specifically, healing payment, or dispute submission. These will need to be instrumented as part of the MVP build.

### Proposed analytics events for MVP

| Event Name | Trigger | Key Properties |
|-----------|---------|---------------|
| `EWS_HEALING_BALANCE_VIEWED` | Member views their charged-off balance in-app | `account_id`, `balance_amount`, `days_since_chargeoff`, `channel` (in-app/agent) |
| `EWS_HEALING_PAYMENT_INITIATED` | Member starts payment flow | `account_id`, `balance_amount`, `payment_method` (debit_card) |
| `EWS_HEALING_PAYMENT_COMPLETED` | Payment successfully processed | `account_id`, `amount_paid`, `payment_method`, `processing_time_ms` |
| `EWS_HEALING_PAYMENT_FAILED` | Payment attempt fails | `account_id`, `amount_attempted`, `failure_reason`, `payment_method` |
| `EWS_HEALING_REPORTED_TO_EWS` | Healed status submitted to EWS | `account_id`, `days_since_payment`, `reporting_method` (api/batch) |
| `EWS_DISPUTE_SUBMITTED` | Member submits a dispute | `account_id`, `dispute_category`, `channel` (phone/mail) |
| `EWS_DISPUTE_RESOLVED` | Dispute investigation completed | `account_id`, `outcome` (upheld/denied/partial), `resolution_days` |

---

## 1. Problem Statement

SoFi is contractually required by Early Warning Services (EWS) to contribute Checking & Savings account data to the National Shared Database. As part of this obligation, when a member's account is charged off and reported, SoFi **must** provide a path for the member to:

1. **Repay the charged-off balance** ("heal" their record) so the negative flag is removed from EWS
2. **Dispute data they believe is inaccurate**, with investigation and resolution within FCRA-mandated timelines

Today, **neither capability exists at SoFi**. There is no member-facing experience, no operational workflow, and no system integration to process healing payments or manage EWS data disputes for charged-off deposit accounts.

### Who is affected

- **Charged-off C&S members:** Members whose SoFi Checking & Savings accounts were closed with a negative balance. Estimated in the thousands to tens of thousands. Typical balances are small (likely under $100), often resulting from ACH returns or timing issues rather than intentional fraud.
- **SoFi Operations:** No process, tooling, or training exists to handle healing payments or EWS data disputes.
- **Compliance and Legal:** Regulatory gap that grows with every charge-off reported to EWS without a corresponding healing/dispute path.

---

## Data Findings

_All data queried April 2, 2026, from production Snowflake via `snow sql -c sofi`._

### A note on data sources

Two Snowflake tables track charge-off closures, and they report different numbers:

- **`MONEY_ACCOUNT_CLOSURES`** is a curated, deduplicated table: one row per account, representing the final closure state. This is the authoritative source for unique account counts but was last refreshed October 6, 2025.
- **`ACCOUNT_TO_CLOSE_EVENTS`** is an event stream that captures the full lifecycle of a closure (started, pending, closed, canceled). A single account can generate multiple events across statuses, so raw counts overstate unique accounts by approximately 1.57x.

The numbers below come from **transaction-level data** (`RR_BANKING_TRANSACTIONS_FACTS`), which counts the actual charge-off transaction posted to each account and is the most reliable per-month source.

### Monthly charge-off volume (last 6 months)

| Month | Accounts | Avg Balance | Monthly Loss |
|---|---:|---:|---:|
| Mar 2026 | 7,722 | $140 | $1.08M |
| Feb 2026 | 7,164 | $156 | $1.12M |
| Jan 2026 | 7,724 | $227 | $1.75M |
| Dec 2025 | 7,085 | $203 | $1.44M |
| Nov 2025 | 6,678 | $184 | $1.23M |
| Oct 2025 | 7,567 | $284 | $2.15M |
| **6-month avg** | **~7,300** | **~$199** | **~$1.5M** |

Source: `TDM_RISK_MGMT_HUB.SUMMARIZED.RR_BANKING_TRANSACTIONS_FACTS` joined with `RR_BANKING_TRANSACTIONS_DETAILS`, filtered on charge-off transaction codes (`DDCHGOFF`, `DDWRTOFF`, `SDCHGOFF`, `SDWRTOFF`).

DDA (checking) charge-offs account for 95%+ of volume. Savings charge-offs are rare (~40/month) but carry higher average balances (~$1,200).

### Recovery opportunity

| Recovery Rate (industry benchmark: 5 to 15%) | Estimated Monthly Recovery |
|---|---:|
| 5% | ~$73K |
| 10% | ~$146K |
| 15% | ~$219K |

---

## Member Experience: Current State vs. Proposed

### Pre-charge-off (57-day grace period)

| Day | Current Experience | Proposed Experience (MVP) |
|-----|---|---|
| Day 0 | ACH return, failed transaction, or other event causes account to go negative. Balance goes negative in the app. Member may or may not notice. | **Same**, plus in-app banner: "Your account has a negative balance. Resolve it to avoid account closure and reporting to Early Warning Services." |
| Day 1 to 14 | Push notification / email: "Your account has a negative balance of -$XX. Please deposit funds to bring it back to $0." | **Push notification + email** with EWS implication: "Your account has a negative balance of -$XX. If not resolved, your account will be closed and reported to Early Warning Services (EWS), which may prevent you from opening bank accounts at other institutions for up to 5 years." |
| Day 15 to 45 | Continued generic reminders. Account may be restricted from certain transactions. | **Escalated push notification + email** reinforcing consequences: "You have X days left to bring your balance to $0. After that, your account will be closed, the balance charged off, and reported to EWS." |
| Day 46 to 57 | Stronger language: "Your account will be closed if the balance is not brought to $0." | **Final warning push notification + email**: "Your account will be closed in X days. The charged-off balance will be reported to EWS and may block you from opening accounts at other banks for up to 5 years. Deposit $XX now to avoid this." |
| Day 57 | Account is closed and balance is charged off. The Banking tab no longer shows the account. No indication of what they owe or how to pay. | Account is closed and balance is charged off. **Post-closure email**: "Your account has been closed with an outstanding balance of $XX. This will be reported to EWS. Log in to your SoFi app to pay and clear your record, or call us at [number]." |

The 57-day grace period and closure process remain the same. The proposed change is to the **messaging**: every notification now explains the EWS consequence so members understand what is at stake and are motivated to act before charge-off.

### Post-charge-off (current vs. proposed)

**Logged-in experience (SoFi app):**

| Trigger | Current Experience | Proposed Experience (MVP) |
|---|---|---|
| Member logs into SoFi app | Banking tab shows no active account. No mention of charged-off balance. Confusion. | Banking tab displays a "Charged-Off Balance" card showing amount owed, EWS impact, and options to pay or dispute. |
| Member tries to open a new SoFi Money account | Declined with no clear explanation of why or how to fix it. | Declined, but directed to resolve the charged-off balance first. |
| Member wants to pay | No payment channel exists. **Complete dead end.** | In-app debit card payment (Stripe). Full balance only for MVP. Instant confirmation, EWS updated within 10 business days. |
| Member wants to dispute data | No dispute intake process. Agent cannot help. | "Think this is wrong? Dispute this data" link routes to phone/mail instructions (MVP). Self-service form in Phase 2. |

**Logged-out experience (guest portal, sofi.com/pay):**

| Trigger | Current Experience | Proposed Experience (MVP) |
|---|---|---|
| Member receives adverse action letter from another bank | Letter names SoFi/EWS as data source. No recourse, no way to pay. | Letter includes sofi.com/pay URL. Member visits, looks up balance with reference number or identity verification, and pays. |
| Member cannot log in (forgot credentials, no app) | No payment channel. Agent cannot help beyond "try resetting your password." | Guest portal: verify identity with reference number + last 4 SSN (or name + DOB + SSN), see balance, pay via debit card, receive email receipt. No login required. |
| Member calls SoFi support | Agent has no tooling, SOP, or payment mechanism. Dead end. | Agent looks up account, directs member to sofi.com/pay or processes payment via agent tool. |
| Member wants to dispute (no login) | No dispute intake process. No defined address or timeline. | Guest portal includes "Think this is wrong?" link with phone number and mailing address for disputes. |

---

### Proposed experience detail

The healing flow has two variants depending on whether the member can log in. Both use the same Stripe payment backend and produce the same EWS update.

#### Logged-in experience (SoFi app)

For members who can log into their SoFi account. This is the primary flow.

**Entry points:**

| Entry point | Frequency (estimated) | Flow |
|------------|----------------------|------|
| **Adverse action from another bank** | Most common. Member is denied at another FI, sees SoFi named in the adverse action letter. | Member calls SoFi or logs into app. |
| **Member logs into SoFi app** | Less common. Member opens the app and sees the charged-off balance banner. | In-app flow begins. |
| **SoFi proactive outreach (Phase 2)** | Future state. Push notification or email. | "You have an outstanding balance. Pay now to clear your EWS record." |

**In-app healing flow (primary):**

```
Step 1: Member logs into SoFi app
        (They remain a SoFi member even though their bank account is closed.)

Step 2: Banking tab displays a "Charged-Off Balance" card
        ┌─────────────────────────────────────────────────┐
        │  ⚠ You have an outstanding balance              │
        │                                                  │
        │  Your Checking account was closed on [date]      │
        │  with an outstanding balance of $XX.XX.          │
        │                                                  │
        │  This has been reported to Early Warning          │
        │  Services (EWS), which may affect your ability   │
        │  to open accounts at other banks.                │
        │                                                  │
        │  Pay this balance to clear your record.          │
        │                                                  │
        │  [Pay Now]              [Learn More]             │
        │                                                  │
        │  Think this is wrong? [Dispute this data]        │
        └─────────────────────────────────────────────────┘

Step 3: Member taps "Pay Now"
        - Pre-populated with exact balance owed (full payment only for MVP)
        - Enter debit card information (Stripe)
        - No credit card accepted
        - Summary: "You are paying $XX.XX to clear your SoFi charge-off."

Step 4: Payment confirmation
        ┌─────────────────────────────────────────────────┐
        │  ✓ Payment received                              │
        │                                                  │
        │  Amount paid: $XX.XX                             │
        │  Date: [date]                                    │
        │  Payment method: Debit card ending in XXXX       │
        │                                                  │
        │  What happens next:                              │
        │  • Your EWS record will be updated within        │
        │    10 business days.                             │
        │  • You will receive an email confirmation.       │
        │  • Once updated, this charge-off will no longer  │
        │    block you from opening accounts at other      │
        │    financial institutions.                       │
        │                                                  │
        │  [Done]                                          │
        └─────────────────────────────────────────────────┘

Step 5: Email confirmation sent immediately
        - Amount paid, date, confirmation number
        - "Your EWS record will be updated within 10 business days."

Step 6: Backend processing
        - Payment deposited into recovery GL
        - Account status updated in system of record
        - EWS reporting triggered (next batch cycle, within 10 business days)

Step 7: EWS record updated (async, within 10 business days)
        - Member receives follow-up email: "Your EWS record has been updated."
```

**Agent-assisted healing flow (fallback):**

```
Step 1: Member calls SoFi support
        - Prompted by adverse action letter from another bank,
          or member is unable to log into app

Step 2: Agent verifies identity (standard verification)

Step 3: Agent looks up member's charged-off account(s)
        - Agent tool shows: account number, charge-off date, outstanding balance
        - If multiple charged-off accounts, agent lists all

Step 4: Agent directs member to pay
        Option A (preferred): "Please log into your SoFi app. You'll see
                  your balance and can pay there securely."
        Option B (if member cannot access app): Agent collects debit card
                  info and processes payment via agent-facing tool.

Step 5: Same confirmation, email, and EWS reporting as in-app flow
```

#### Logged-out experience (guest payment portal)

For members who cannot log in: forgot credentials, no app access, or directed by an adverse action letter from another bank. They visit a web payment portal (sofi.com/pay) and look up their balance without needing a SoFi login.

**How the member gets to sofi.com/pay:**

| Entry path | What the member sees |
|---|---|
| **Adverse action letter from another bank** | Member is denied at another FI. Letter names SoFi/EWS. Member calls SoFi or searches online. Agent directs them to **sofi.com/pay**. |
| **Charge-off notification email (Day 57)** | "Your account has been closed with a balance of $XX. This will be reported to EWS. **Pay at sofi.com/pay** or log in to the SoFi app." |
| **Pre-charge-off reminder emails (Day 1 to 57)** | "Deposit funds to avoid closure. If your account is closed, you can pay at sofi.com/pay." |
| **SoFi support agent** | Agent: "You can pay online at sofi.com/pay. You'll need your reference number from your email, or I can verify your identity over the phone." |
| **SoFi help center / FAQ** | Help article: "How to pay a charged-off balance and clear your EWS record." Links to sofi.com/pay. |
| **In-app "Pay on web instead" link** | Logged-in member who prefers web can tap this link on the charged-off balance card. Opens sofi.com/pay in mobile browser. |

The URL **sofi.com/pay** must appear in: (1) charge-off notification email, (2) pre-charge-off reminder emails, (3) adverse action notice language, (4) SoFi help center, and (5) agent scripts/SOPs.

**Guest flow:**

```
Step 1: Member visits sofi.com/pay
        (URL included in adverse action letter, charge-off email,
        or provided by SoFi support agent.)

Step 2: Look up balance (two options)
        Option A: Reference number (from letter/email) + last 4 of SSN
        Option B: Full name + date of birth + last 4 of SSN

Step 3: Identity verified, balance displayed
        - Shows: account number (masked), closure date, balance owed, EWS status
        - Session expires in 15 minutes for security

Step 4: Enter debit card information
        - Same Stripe integration as in-app flow
        - Email address collected for receipt delivery
        - No credit card accepted

Step 5: Payment confirmation
        - On-screen confirmation with reference number
        - Receipt emailed to provided address
        - Option to download/print receipt
        - "Keep your confirmation number" callout

Step 6: Backend processing (same as in-app)
        - Payment deposited into recovery GL
        - Account status updated in system of record
        - EWS reporting triggered (within 10 business days)
```

**Key design considerations for guest portal:**

- **No account creation required.** The member should be able to pay without signing up or remembering credentials.
- **Time-boxed session.** Identity verification grants a 15-minute window for security.
- **Receipt delivery.** Since there's no app to return to, the email receipt and confirmation number are critical.
- **Dispute link included.** "Think this is wrong?" link routes to phone/mail dispute instructions, same as in-app.
- **Redirect to login.** "Have a SoFi account? Log in instead" link for members who can use the app.

#### Journey 2: Dispute

**How does a member learn they can dispute?**

| Discovery path | What the member sees |
|---|---|
| **In-app charged-off balance card** | "Think this is wrong? Dispute this data" link below the Pay button. Taps through to dispute instructions screen with phone number and mailing address. |
| **Guest portal (sofi.com/pay)** | "Think this is wrong? Dispute this data" link on the balance page. Same dispute instructions. |
| **Charge-off notification email (Day 57)** | Email includes: "If you believe this information is inaccurate, you have the right to dispute it. Call 1-855-456-7634 or write to SoFi Bank, N.A., 2750 East Cottonwood Parkway #300, Cottonwood Heights, UT 84121." |
| **Adverse action letter from another bank** | Letter names EWS. EWS consumer services provides SoFi contact info. Member calls SoFi, agent explains dispute process. |
| **SoFi help center / FAQ** | Help article: "How to dispute data SoFi reported to EWS." Includes phone number, mailing address, and what to include in a dispute. |
| **Agent (phone)** | Any member who calls SoFi about a charge-off is informed of their right to dispute. Agent explains the process and can intake the dispute on the spot. |

**Dispute contact info must appear in:** (1) in-app dispute screen, (2) guest portal dispute page, (3) charge-off notification email, (4) help center, and (5) agent scripts.

**Entry points:**

| Entry point | Flow |
|------------|------|
| **Member calls SoFi** | Agent captures dispute details, routes to investigation team. |
| **Member sends mail** | Written dispute received at: SoFi Bank, N.A., 2750 East Cottonwood Parkway #300, Cottonwood Heights, Utah 84121 |
| **Indirect (CRA-routed)** | EWS forwards a dispute initiated by the member through another channel. SoFi investigates per FCRA. |
| **In-app link (MVP)** | "Think this is wrong? Dispute this data" link on the charged-off balance card routes to phone number/instructions for MVP. Self-service form in Phase 2. |
| **Guest portal link** | "Think this is wrong?" link on sofi.com/pay balance page routes to dispute instructions. |

**Dispute flow (MVP, agent-assisted):**

```
Step 1: Member contacts SoFi (phone or mail)
        States what they are disputing:
        - Wrong balance amount
        - Wrong address
        - Not closed due to fraud (incorrect closure reason)
        - Account not theirs (identity theft)
        - Charge-off date incorrect
        - Already paid / settled
        - Other data inaccuracy

Step 2: Agent captures dispute
        - Logs in case management system (Salesforce or equivalent)
        - Records: disputed data element, member's stated reason,
          any supporting documentation provided
        - Provides member with case number and expected timeline

Step 3: Triage (within 5 business days)
        If dispute is frivolous/irrelevant (duplicate, insufficient info):
          → Notify member within 5 business days with reasons and
            list of required information. No investigation required.
        If dispute is valid:
          → Proceed to investigation.

Step 4: EWS contribution record updated
        - "Consumer Disputes Debt" flag set to Y
        - Consumer's dispute statement added to record
        - Submitted in next EWS batch

Step 5: Investigation (within 30 days; 45 if member provides new info)
        - Review all member-provided information
        - Review internal records (account history, transaction data,
          closure records)
        - Determine if reported data is accurate

Step 6: Resolution
        If data was INACCURATE:
          → Correct the data in EWS and all CRAs that received it
          → Notify member: "Your dispute has been resolved. We have
             corrected [specific data element] with EWS."
          → Remove dispute flag

        If data was ACCURATE:
          → Notify member: "After investigation, we have confirmed
             the reported data is accurate. You have the right to
             add a personal statement to your record."
          → Remove dispute flag, retain accurate data

        If IDENTITY THEFT:
          → Cease furnishing the disputed item to EWS
          → Do not refurnish unless later confirmed correct
          → Provide victim with application/transaction records
             within 30 days of request (no charge)

Step 7: Member notification
        - Letter/email with investigation outcome
        - If denied: explanation of why + right to add consumer statement
        - If upheld: confirmation of correction + expected timeline for
          EWS record update
```

### State diagram: member account lifecycle with healing

```
    ┌──────────────┐
    │ Account Open  │
    │ (Active)      │
    └──────┬───────┘
           │ Balance goes negative
           ▼
    ┌──────────────┐
    │ Negative      │  ← Member can still deposit to cure
    │ Balance       │    (57-day grace period)
    │ (Grace Period)│
    └──────┬───────┘
           │ 57 days elapse without cure
           ▼
    ┌──────────────┐
    │ Charged Off / │  ← Account permanently closed in Galileo
    │ Closed        │    Reported to EWS as Account Abuse
    └──────┬───────┘
           │
      ┌────┴────────────────┐
      │                     │
      ▼                     ▼
┌───────────┐        ┌────────────┐
│ Member    │        │ Member     │
│ Pays      │        │ Disputes   │
│ (Healing) │        │ (FCRA)     │
└─────┬─────┘        └──────┬─────┘
      │                     │
      ▼                     │ Investigation
┌───────────┐               │ (30/45 days)
│ Payment   │               │
│ Applied   │         ┌─────┴──────┐
│ to GL     │         │            │
└─────┬─────┘         ▼            ▼
      │         ┌──────────┐ ┌──────────┐
      │         │ Upheld   │ │ Denied   │
      │         │ (Data    │ │ (Data    │
      │         │ corrected│ │ accurate)│
      │         └────┬─────┘ └──────────┘
      │              │
      ▼              ▼
┌──────────────────────────┐
│ EWS Record Updated       │
│ (within 10 business days)│
│ "Healed" / "Corrected"  │
└──────────────────────────┘
```

### Key UX principles

1. **Empathetic tone.** Members in this flow are often in a difficult financial situation. Many had small balance issues (< $100) caused by timing, not bad intent. Copy should acknowledge this: "We understand mistakes happen" rather than "You owe us money."

2. **Clarity over cleverness.** Most members have never heard of EWS. The experience must explain what EWS is, why it matters, and exactly what "healing" does for them, in plain language.

3. **No dead ends.** Every screen must have a clear next action. If the member can't pay right now, tell them the balance will be here when they're ready. If they think the data is wrong, give them a clear dispute path.

4. **Transparency on timeline.** "Within 10 business days" is a real commitment. Show it prominently. After the update happens, confirm it.

5. **Agent consistency.** Agents handling calls should have the same information the member sees in the app. No conflicting balances, no "I don't have access to that."

---

## 2. Goals & Success Metrics

### North Star Metric

**Healing Rate: % of eligible charged-off members who repay and clear their EWS record.**

This single metric captures whether the product is working: member awareness, payment UX, and backend processing rolled into one number. Today there are ~221,000 all-time charged-off accounts and ~8,000 new ones every month. Even a modest healing rate translates into meaningful recovery and, more importantly, regulatory compliance.

| | Baseline | 6-Month Target | 12-Month Stretch |
|---|---|---|---|
| Healing rate | 0% (no capability exists) | 5 to 10% | 10 to 15% |
| Monthly recovery dollars | $0 | $140K to $270K | $350K+ |

Industry benchmark for deposit charge-off recovery: 5 to 15%.

### Supporting Metrics

| Metric | Baseline | Target | Why it matters |
|--------|----------|--------|----------------|
| **EWS reporting SLA** - % of healed accounts reported to EWS within 10 business days | N/A | 100% | Contractual obligation. Missing this is a compliance violation. |
| **Dispute resolution SLA** - % of disputes resolved within 30 days (45 if additional info) | N/A | 100% | FCRA mandate. Non-compliance risks regulatory action. |
| **Payment completion rate** - % of members who start payment and finish | N/A | > 80% | Signals whether the payment UX works or has friction. |
| **Monthly recovery amount ($)** | $0 | Track from launch | Financial recovery against ~$1.4M/month in new charge-off losses. |

### Guardrails (should not degrade)

| Metric | Constraint |
|--------|-----------|
| Member satisfaction (CSAT) for healing/dispute flow | >= 4.0 / 5.0 |
| Payment failure rate | < 10% |
| False dispute rejection rate | < 5% |

---

## 3. User Stories & Requirements

### Workstream 1: Account Healing (Repayment)

| Priority | User Story | Acceptance Criteria |
|----------|-----------|-------------------|
| P0 | As a member with a charged-off C&S account, I want to see how much I owe SoFi so I can decide whether to repay | Member can view their outstanding charged-off balance through at least one channel (in-app or agent-assisted). Balance is accurate and sourced from a defined system of record. |
| P0 | As a member, I want to make a payment to repay my charged-off balance so my EWS record is cleaned up | Member can submit payment for the full outstanding balance. Payment is processed, funds are deposited into the recovery GL, and a confirmation is provided. |
| P0 | As SoFi, when a member repays a charged-off balance, I need to report the healed status to EWS within 10 business days | System triggers EWS update (via API or batch file) within 10 business days of confirmed payment. Account Abuse data is updated to reflect healing. |
| P0 | As a member, I want a receipt confirming my payment and an explanation of what happens next | Member receives email confirmation with: amount paid, date, expected timeline for EWS record update, and what "healed" means. |
| P1 | As an ops agent, I want to look up a member's charged-off account and process a payment on their behalf | Agent can search by member identifier, view the charged-off balance, and initiate a payment collection flow. |
| P1 | As a member, I want to be notified when my EWS record has been updated | Member receives notification (email) when EWS confirms the record has been updated to "healed." Assumption: depends on EWS providing a confirmation or SLA. Confirm or correct. |
| P2 | As SoFi, I want to proactively inform members who are about to be charged off that their account may be reported to EWS | Updated charge-off communications include language about potential EWS reporting and its consequences. Assumption: requires Compliance approval. Confirm or correct. |

### Workstream 2: Data Dispute

**Dispute categories:** Members may dispute any data element SoFi reported to EWS, including but not limited to:
- Wrong balance amount
- Wrong address
- Incorrect closure reason (e.g., not closed due to fraud)
- Account not theirs (identity theft)
- Charge-off date incorrect
- Account already paid / settled

The investigation requirement is to "fix if wrong or provide an update." Different data elements have different response timing requirements and pause obligations (per FCRA and EWS Operating Rules).

| Priority | User Story | Acceptance Criteria |
|----------|-----------|-------------------|
| P0 | As a member, I want to dispute data SoFi reported to EWS that I believe is inaccurate | Member can initiate a dispute through at least one channel (phone or mail for MVP). Dispute is captured with: specific data being disputed (from categories above), reason, and any supporting documentation. |
| P0 | As SoFi, I need to investigate and resolve disputes within FCRA timelines | Investigation completed within 30 days of receipt (45 days if member provides additional info mid-investigation). Member notified of outcome. |
| P0 | As SoFi, if a dispute is frivolous or irrelevant, I need to notify the member within 5 business days | Frivolous dispute determination made and member notified with reasons and list of missing information within 5 business days. |
| P0 | As SoFi, when a dispute is received, I need to flag it in the EWS contribution data | Account Abuse and/or Fraud file updated with "Consumer Disputes Debt" indicator (Y) and consumer's statement. Flag removed when dispute is resolved. |
| P0 | As SoFi, when a dispute is upheld, I need to correct the data with EWS and all CRAs that received it | Corrected records submitted to EWS and all other CRAs within required timeframe. |
| P1 | As a member, if my dispute is denied, I want to understand why and know my right to add a statement | Member receives explanation of denial and is informed of their FCRA right to add a consumer statement to the record. |
| P1 | As SoFi, I need to maintain an audit trail of all disputes and resolutions | All dispute activity (intake, investigation steps, outcome, notifications) is logged and auditable. |
| P0 | As SoFi, when we receive an indirect dispute via EWS/CRA notice, I need to investigate and respond | Investigate, review CRA-provided info, respond with results, propagate corrections to all nationwide CRAs. Modify/delete/block disputed data if unverified or inaccurate. |

---

## 4. Scope

### In scope (MVP, targeting July 31, 2026)

- **Payment collection for charged-off C&S account balances** (full balance payment; no partial payments or payment plans for MVP)
- **Payment method:** Debit card via Stripe (leveraging SoFi Plus pattern) as primary method. ACH push from external bank as secondary. **No credit card payments.**
- **Recovery GL:** Funds deposited into a SoFi recovery/collections general ledger (original account is closed and cannot receive deposits)
- **EWS healing report:** Update Account Abuse data within 10 business days of confirmed payment, submitted via established EWS reporting mechanism (API or batch file)
- **Logged-in experience:** In-app view of charged-off balance and payment flow for members who can log in to SoFi app
- **Logged-out experience:** Guest payment portal (sofi.com/pay) for members who cannot log in. Balance lookup via reference number or identity verification, debit card payment, email receipt.
- **Pay by phone (agent-assisted):** Ops agent can look up balance, collect debit card info, and process payment for members who call in
- **Dispute intake:** Phone-based and mail-based dispute submission (FCRA requires mail; phone for member convenience)
- **Dispute investigation:** Ops team investigates within FCRA timelines using existing or adapted case management tooling
- **Dispute reporting:** EWS contribution records updated with dispute flags and consumer statements
- **Member communications:** Receipt/confirmation for healing payments; dispute acknowledgment, status updates, and resolution notices
- **Adverse action notices:** Include EWS contact info per FCRA Section 615(a) requirements

### Out of scope

| Item | Reason |
|------|--------|
| Reopening charged-off accounts | Charged-off accounts are closed in the current core. New core could theoretically support revival, but prerequisites unavailable until late August 2026 at earliest. Not a dependency for MVP. |
| Proactive collections or outbound outreach | Amounts are small (typically < $100). Collections agency costs would exceed recovery. Reputational risk outweighs financial benefit. |
| Third-party debt collection workflows | Same as above. Not justified for the balance sizes involved. |
| Loan or credit card charge-offs | Different products, different reporting mechanisms (credit bureaus, not EWS). Lending has its own recovery processes. |
| Settlement or partial payment negotiation | MVP supports full balance payment only. Partial payments and payment plans deferred to Phase 2. |
| Self-service dispute submission (web/app form) | MVP uses phone + mail intake. Self-serve dispute form deferred to Phase 2 based on dispute volume. |
| New core banking account revival | David Finske confirmed this is technically possible in the new core, but prerequisites unavailable before late August 2026. Do not take a dependency. |
| Changes to initial charge-off reporting pipeline | Being handled separately by Michelle Barnwell + Data Engineering (Daniel Evanko). |
| Hot File contribution | Explicitly out of scope per the BRD. |
| ~~Logged-out payment experience~~ | **Moved to MVP.** Guest portal (sofi.com/pay) is now in scope. Identity verification via reference number + last 4 SSN or name + DOB + SSN. |

### Future phases

**Phase 2 (post-MVP):**
- **PayNearMe integration** for broader payment method support (CC, debit, Venmo, cash, Apple Pay). Already in use for lending recovery. Enables logged-out payment experience with unique payment links.
- In-app self-service dispute submission with document upload
- Partial payments and payment plans (settlement at less than full balance, similar to lending's ~60% settlement model)
- Proactive outreach to eligible members (email, push)
- ACH payment support from external banks
- ~~Payment website with case number for members who cannot log in~~ (moved to MVP as guest portal)
- Analytics dashboard: healing rate, dispute volume, resolution time
- Evaluate re-engagement: offer healed members a path back to SoFi

**Phase 3 (post new core migration):**
- Account revival capability for charged-off accounts (if new core supports it)
- Automated balance-triggered closure after full repayment
- Member account re-opening flow (requires fraud/risk review for bad actor mitigation)

---

## 5. Edge Cases & Error States

| Scenario | Expected Behavior |
|----------|-----------------|
| **Member account closed for fraud** (not just negative balance) | Fraud-closed members may not be able to log in. Agent-assisted path required. Identity verification must be heightened. Do not contribute abuse/fraud records until investigation is complete and confirmed (per BRD). |
| **Member has multiple charged-off accounts** | Display all charged-off balances. Allow member to pay each separately. Each triggers its own EWS healing update. |
| **Member overpays** | System should prevent overpayment by pre-populating the exact balance. If overpayment occurs, initiate refund process. |
| **Payment fails or times out** | Display clear error message. Do not update EWS. Allow retry. Log the attempt for ops visibility. |
| **Member no longer has SoFi app access** (forgot credentials, deleted account, etc.) | Direct to guest portal (sofi.com/pay). Agent-assisted fallback via phone. Standard identity verification before processing. |
| **Very old charge-off (1+ years)** | Still eligible for healing. Balance sourced from system of record. No expiration on healing eligibility. Note: EWS flags can persist for 5 years. |
| **Very small balance (< $5)** | Still allow payment. Open question: is there a dollar threshold below which SoFi should write off rather than collect? Finance to advise. |
| **Member disputes during active healing payment** | Pause healing flow until dispute is resolved. Do not double-process. |
| **Member pays via push method (wire/ACH from external bank)** | For MVP, this requires member to contact SoFi to confirm payment context (which account to apply to). Not a self-service path in Phase 1. |
| **Concurrent dispute and healing** | If a member is both disputing data and wanting to pay, the dispute takes priority. Payment can proceed after dispute resolution if balance is confirmed accurate. |
| **Identity theft claim on the charged-off account** | Route to fraud/identity theft handling per FCRA. Cease furnishing the disputed item to EWS until investigation confirms accuracy. Do not refurnish information alleged to be the result of identity theft unless later confirmed correct. |
| **EWS reporting system is down** | Queue the update. Retry within the 10-business-day window. Alert ops if approaching SLA deadline. |
| **Balance discrepancy between system of record and what member expects** | Provide member with detailed breakdown. If member disagrees, route to dispute flow. |
| **Platform differences (mobile vs. web)** | MVP may be mobile-only (SoFi app) with agent-assisted fallback for web. Confirm with Design. |

---

## 6. Dependencies & Risks

| Dependency / Risk | Owner | Status | Mitigation |
|------------------|-------|--------|------------|
| **System of record for charged-off balances** - Where do charged-off account balances live today? Core banking (Galileo)? Manual ledger? | Banking Eng / Finance | Open - Discovery needed | Phase 0 discovery item. Cannot build lookup or payment flow without this. |
| **EWS integration specs for healing reports** - API vs. batch file, format, certification process | Michelle Barnwell / EWS | Open - Requested | Start integration discovery immediately. May need EWS certification before going live. |
| **Recovery GL setup** - Finance must define GL codes and structure for recovery payments | Finance | Open | Engage Finance in Week 1. SoFi Plus and Smart Card may have precedent GL structures. |
| **Engineering partner assignment** - No engineering team assigned yet | Shahar Ronen / Dheeraj Mehta / Arjun Hegde | Open | Shahar initiating conversation with Dheeraj and Arjun. Likely MACE for member experience, Ragini's team for ops component. |
| **Compliance approval for updated member communications** - Adding EWS reporting language to charge-off notices | Compliance (Tatyana Pacheco, Kristen Krikorian) | Open | Engage compliance early; may require legal review of disclosure language. |
| **Stripe integration for payment collection** - Reusing SoFi Plus payment pattern | Payments Eng | Open - Discovery needed | Confirm SoFi Plus Stripe integration can be extended. Smart Card's approach is an alternative. |
| **New core banking not ready** - Account revival prerequisites unavailable until late August 2026 | Banking Eng (David Finske) | Confirmed - Not available | Do not take a dependency. Build GL-based approach instead. |
| **FCRA dispute timeline compliance** - Must resolve within 30 days or face regulatory action | Ops / Compliance | Open | Automated SLA tracking and escalation alerts. Ops training needed. |
| **July 31 deadline pressure** - MVP in production by July 31, 2026 | Product (Shweta) | At risk - Eng not assigned | Negotiate scope if needed: fully ops-assisted MVP first, self-serve in Phase 2. Some room to negotiate timeline with business justification. |
| **Dispute volume unknown** - Don't know how many disputes to expect | Product / EWS | Open | Ask EWS for typical dispute rates for charged-off deposit accounts. This will determine if we need self-serve dispute tooling for MVP or if phone/mail is sufficient. |
| **Fraud-closed accounts** - Members closed for fraud may need different handling than negative-balance closures | Fraud Strategy (Dan Rosner, Arpit Ajmani) | Open | Confirm whether fraud-closed members can still log in. May need separate identity verification flow. |
| **EWS can terminate services** - If full contribution compliance not met by December 31, 2026 | EWS (external) | Active risk | July 31 MVP is a subset of the December 31 full scope. Coordinate with broader EWS Data Contributions initiative. |

---

## 7. Cross-Functional Requirements

| Function | Requirements | Status |
|----------|-------------|--------|
| **Legal / Compliance** | FCRA dispute handling (30-day resolution, 5-day frivolous determination, consumer statement rights). UDAAP compliance (must offer healing if reporting charge-offs). Adverse action notices must include EWS contact info. Review updated charge-off communications for EWS language. Confirm dispute intake channels (mail required by FCRA; phone optional). Specified mail address for disputes: SoFi Bank, N.A., 2750 East Cottonwood Parkway #300, Cottonwood Heights, Utah 84121 (confirm with Legal). Confirm data retention and audit trail requirements. | Open - Engage Tatyana Pacheco, Kristen Krikorian, Brandi Morales-Espinal, Derek Fisher |
| **Analytics** | Track: payment volume, healing rate, dispute volume and resolution time, recovery amount ($), ops handle time, member funnel (balance viewed > payment initiated > payment completed). EWS reporting SLA adherence. | Not started |
| **Operations / Support** | Agent training for payment collection and dispute intake. SOPs for: processing payments, routing disputes, handling identity theft claims, escalation paths. MET Operations involvement: Nikita Shah, Erica Hester. Specialty Ops: Joe Conlon. Update existing SOPs per BRD requirement. | Open - Engage Ops stakeholders |
| **Design** | In-app healing experience: charged-off balance display in Banking tab, payment flow, confirmation screen. Empathetic copy (members are trying to do the right thing). Clear explanation of what EWS is and what "healing" means. Dispute intake UX (Phase 2). | Not started |
| **QA** | Test plan covering: payment processing, GL ledgering, EWS reporting trigger, dispute flag updates, FCRA timeline enforcement, edge cases (overpayment, timeout, multiple accounts). | Not started |
| **Finance** | Recovery GL design: GL codes, account structure, reconciliation process. Tax implications: 1099-C if balance was partially forgiven (Phase 2 concern). Write-off threshold guidance. | Open - Engage Finance |
| **Fraud / Risk** | Identity verification for agent-assisted payments. Fraud-closed account handling. Bad actor mitigation (e.g., someone paying off a charged-off account to gain access to identity/settings). Review dispute flow for fraudulent dispute prevention. | Open - Engage Dan Rosner, Arpit Ajmani |
| **Privacy** | Data sharing review: member balance and transaction data shared with EWS. Confirm existing disclosures cover this data sharing or identify gaps. PIA (Privacy Impact Assessment) if new data elements are shared. | Open - Engage Sherrie Osborne (Privacy) |
| **Cyber Security** | Payment flow security review. Data transmission security for EWS reporting. PCI DSS compliance for debit card payment collection via Stripe. | Open - Engage Sandeep Jayashankar |

---

## 8. Technical Context

### Key technical constraints

- **Charged-off accounts are closed in Galileo.** Cannot deposit into or reopen them. Any payment must be routed to a recovery GL, not the original account. This is fundamentally different from loan/credit card charge-offs, which remain open with a charged-off status.
- **Members retain SoFi app access.** Even though their bank account is closed, they remain SoFi members and can log in. The Banking tab could surface the charged-off balance and payment flow.
- **New core account revival not available.** David Finske confirmed the new core could theoretically support reopening charged-off accounts, but prerequisites won't be available until late August 2026 at the earliest. Not a viable dependency for July 31 MVP.

### Payment collection approach

**Internal SoFi patterns** that already collect third-party payments into a GL:

| Pattern | How it works | Pros | Cons |
|---------|-------------|------|------|
| **SoFi Plus** | Collects subscription fees via **Stripe** (debit card or credit card). Funds go to a SoFi GL, not a member account. | Proven Stripe integration. Supports debit card. GL routing already exists. | May need modifications to restrict to debit only (no CC). Need to confirm if the integration can be reused for a different product context. |
| **Smart Card** | Collects payments through a **third-party processor** (details TBD). Uses Money stack with UNA external accounts. | Existing Money stack integration. | Members' accounts are already closed, so UNA-linked external accounts may no longer be accessible. Less suitable for this use case. |

**External payment rail options** (discussed in working sessions):

| Option | How it works | Pros | Cons |
|--------|-------------|------|------|
| **PayNearMe** | Third-party payment platform. Zero barrier to entry: accepts CC, debit, Venmo, cash, and more. Used in lending recovery today. | Maximum payment method flexibility. Member can pay in almost any way. Already used at SoFi for lending recovery. | Adds a vendor dependency and per-transaction fee. Need to evaluate cost vs. benefit for small balance amounts (< $100). Need to confirm acceptable payment methods with Compliance. |
| **Wire / ACH push to SoFi account** | Member initiates a wire or ACH transfer from their external bank to a designated SoFi routing + account number. | No new payment infra needed on SoFi side. Member uses their own bank. | Requires manual reconciliation: SoFi receives funds but needs to match them to the right charged-off account. Member must separately notify SoFi (call/email) of which account to apply payment to. Poor member experience. |
| **FedNow push** | Same as wire/ACH but via FedNow for instant settlement. | Near-instant posting. | Same reconciliation challenges as wire/ACH. FedNow adoption still limited. |
| **Payment website with case number** | Standalone web page where member enters a case number (provided by SoFi), sees their balance, and pays via Stripe or similar. | Works for members who can't log in. Case number ties payment to account automatically. | Requires building a new standalone page. Security considerations (case number as auth token is weak). |

**Recommendation:** Start with the **SoFi Plus / Stripe pattern** for debit card payments as the primary MVP rail. It handles the core need (collect money, apply to GL) without depending on the member having an active SoFi bank account. Evaluate **PayNearMe** as a Phase 2 option for broader payment method support, especially given it's already in use for lending recovery. Wire/ACH push is available as a manual fallback for edge cases but should not be the primary path. See Appendix G for the full payment rail comparison.

### Payment context requirements

When a member makes a payment, SoFi needs to know:
1. **Who** is paying (member identity, verified)
2. **How much** they owe (charged-off balance from system of record)
3. **Which account** the payment is for (if multiple charged-off accounts)

An **in-app experience** naturally solves all three: the member is authenticated, the balance is displayed from the system of record, and the account context is known. Push methods (wire, ACH from external bank) create ambiguity and require manual reconciliation: the member needs to know where to send the money, how much to pay, and SoFi needs to know which account to credit. A **payment website with a case number** partially solves the context problem but introduces security concerns.

### EWS integration

- **Healing report:** Must update Account Abuse data within 10 business days of payment. Mechanism TBD (API call or batch file submission). Confirm with EWS during discovery.
- **Dispute flags:** Contribution records must include "Consumer Disputes Debt" indicator and consumer statement text in both Account Abuse and Fraud files.
- **Existing EWS integration:** SoFi already has a data transmission pipeline for initial charge-off reporting (being built by Michelle + Daniel Evanko). Healing and dispute updates should use the same channel.

### Systems involved

| System | Role | Owner |
|--------|------|-------|
| Core Banking (Galileo) | Source of charged-off account data and balances | Banking Eng |
| Stripe | Payment processing for debit card collections | Payments Eng |
| General Ledger | Recovery GL for received funds | Finance |
| EWS API / Batch File | Submit healed records, dispute flags, corrections | TBD (same channel as contribution pipeline) |
| Case Management (Salesforce?) | Dispute tracking, investigation workflow | Ops Eng |
| SoFi App (iOS / Android) | Member-facing healing and payment flow | MACE / Product Eng |
| Member Communications | Email confirmations, dispute notices, adverse action letters | Growth / Comms Eng |

### Security and authentication

- **Logged-in:** Payment flow is behind SoFi app authentication
- **Guest portal (sofi.com/pay):** Identity verification required before showing balance or accepting payment. MVP options: (a) reference number (from email/letter) + last 4 SSN, or (b) name + DOB + last 4 SSN. No PII exposed until verified.
- **Pay by phone:** Agent-assisted path requires standard identity verification before processing
- PCI DSS compliance required for debit card handling via Stripe across all channels

---

## 9. Timeline & Milestones

**Hard deadline:** December 31, 2026, for full EWS contribution compliance (per contract waiver).
**MVP target:** July 31, 2026, for payment collection and basic dispute handling in production.

| Milestone | Target Date | Notes |
|-----------|------------|-------|
| **Phase 0: Discovery complete** | Week of April 14, 2026 | Confirm system of record, EWS integration specs, GL structure, eng partner assignment. Review SoFi Plus and Smart Card payment patterns. |
| **PRD finalized and approved** | April 21, 2026 | Incorporate discovery findings. Stakeholder alignment. |
| **Design complete** | May 5, 2026 | In-app healing experience, agent-assisted flow wireframes. |
| **Eng kickoff** | May 12, 2026 | Assumption: engineering partner assigned by this date. Confirm or correct. |
| **Payment collection integration (Stripe + GL)** | June 16, 2026 | Core payment flow functional in staging. |
| **EWS healing report integration** | June 30, 2026 | Healing updates can be submitted to EWS. |
| **Ops training and SOP delivery** | July 7, 2026 | Agents trained on payment processing and dispute intake. |
| **QA complete** | July 21, 2026 | Full test cycle including payment, GL, EWS reporting, dispute flags. |
| **MVP launch** | July 31, 2026 | Agent-assisted healing + in-app payment flow live. Dispute intake via phone/mail. Target date, with some room to negotiate if justified. |
| **Phase 2 scope finalized** | August 2026 | Based on MVP learnings, dispute volume, and healing rate data. |
| **Full EWS contribution compliance** | December 31, 2026 | All file types, healing, dispute, governance requirements met. Coordinated with broader EWS Data Contributions initiative. |

---

## 10. Open Questions

| # | Question | Owner | Status | Source |
|---|----------|-------|--------|--------|
| 1 | Where does the source of truth for charged-off account data and balances live today? Core banking (Galileo)? Manual ledger? Collections system? | Banking Eng / Finance | Open | Pager, Meeting |
| 2 | What is the EWS integration mechanism for reporting a healed account? API, batch file, manual portal? Can we extend the existing contribution pipeline? | Michelle Barnwell / EWS | Open | Pager, BRD |
| 3 | What payment methods are permitted? Can we use Stripe (debit card only)? Is ACH from external bank feasible given account closure? | Payments Eng / Compliance | Open | Slack thread (Mili) |
| 4 | How should Finance structure the recovery GL? What GL codes apply? Is there precedent from SoFi Plus or Smart Card? | Finance | Open | Meeting, Pager |
| 5 | Do we support full balance payment only, or partial payments / payment plans for MVP? | Product / Finance / Compliance | Open - Recommendation: full only for MVP | Pager |
| 6 | What are typical EWS dispute rates for charged-off deposit accounts? What volume should we plan for? | EWS (external) | Open | Meeting |
| 7 | Can we update charge-off member communications to mention EWS reporting as a deterrent? What disclosures need to change? | Compliance (Tatyana Pacheco) | Open | Meeting, BRD |
| 8 | Is there a dollar threshold below which SoFi should write off rather than collect? | Finance | Open | Pager |
| 9 | What happens if a member overpays or the recorded balance is wrong? | Finance / Ops | Open | Pager |
| 10 | Do we need to generate a 1099-C or other tax document if a balance was partially forgiven? (Phase 2 concern) | Finance / Legal | Open | Pager |
| 11 | Can members closed for fraud still log in to the SoFi app? Do they need a different flow? | MACE Eng / Fraud | Open | Slack thread (Mili) |
| 12 | What case management system will ops use for disputes? Salesforce? Internal tool? | Ops Eng | Open | Pager |
| 13 | Who on the ops team handles dispute investigation? Existing team or new hire/training needed? | MET Operations (Nikita Shah) | Open | Pager, BRD |
| 14 | What is the EWS SLA for processing a healed record and confirming removal of the flag? | EWS (external) | Open | Pager |
| 15 | Is there an existing FCRA dispute process at SoFi for credit products that we can extend to deposits? | Compliance / Lending Ops | Open | Pager, BRD |
| 16 | Do we have the experience behind the app, or is there a way to send an email link to a payment flow not behind SoFi login? What are the fraud/identity risks? | Product / Security / Compliance | Open - Recommendation: behind app for MVP | Slack thread (Mili) |
| 17 | What does Smart Card's payment collection experience look like? Can any of it be reused? | Product (Shweta) | Open - To investigate | Slack thread (Shahar), Meeting |
| 18 | Which engineering team owns this: MACE (member experience) or Money Movement? | Dheeraj Mehta / Arjun Hegde | Open | Slack thread |
| 19 | Are existing member disclosures sufficient for EWS data sharing, or do we need new/updated disclosures? | Legal / Privacy (Sherrie Osborne) | Open | BRD |
| 20 | How do we handle indirect disputes (received via CRA/EWS notice rather than directly from member)? What is the operational flow? | Ops / Compliance | Open | BRD |
| 21 | What is the average charge-off balance for Money accounts? This directly affects payment rail cost-benefit (e.g., Stripe fees vs. PayNearMe fees vs. write-off threshold). | Finance / Data Eng | Open | Working doc |
| 22 | Can we sell charged-off accounts to a third party, making it their problem and reporting the balance as settled? Is this advisable for small amounts? | Finance / Legal / Compliance | Open - Likely not worth it for small balances | Working doc |
| 23 | Can we simply forgive (write off) all charged-off balances below a certain threshold rather than building collection infrastructure? | Finance / Compliance | Open - Not recommended (still need to report accurately to EWS) | Working doc |
| 24 | How are other banks handling charge-off healing for deposit accounts? What is industry practice? | Product / EWS | Open | Working doc |
| 25 | Is PayNearMe's per-transaction cost justified for balances under $100? What is the fee structure? | Finance / Vendor Management | Open | Working doc |
| 26 | How will users contact SoFi to initiate healing? What information is on the adverse action letter they receive from other FIs? Can we influence the contact method? | Product / Compliance / EWS | Open | Working doc, Meeting |
| 27 | Can we accept wire/ACH push to a SoFi routing + account number? Are there compliance concerns with providing an account number for push payments? | Compliance / Payments Eng | Open | Working doc |
| 28 | What deposit methods should work for the healing payment? Need explicit Compliance sign-off on acceptable methods. | Compliance | Open | Working doc |
| 29 | Confirm Srikant Movva as dispute workstream owner. | Shahar Ronen | Open | Working doc |
| 30 | **Is sofi.com/pay a live URL?** Currently returns a 404. Needs to be provisioned and routed to the guest payment portal before MVP launch. Who owns the DNS/routing? | Web Eng / Infra | **Open - Blocker** | PRD review |
| 31 | What is the fallback URL if sofi.com/pay cannot be provisioned in time? Options: sofi.com/chargeoff-pay, a subdomain, or a direct link in communications only. | Product / Web Eng | Open | PRD review |

---

## Appendix A: Experience Options Analysis

The MVP experience exists on a spectrum. The right answer depends on build capacity before July 31.

### Option 1: Fully Ops-Assisted (minimum EPD build)

**How it works:** Member calls SoFi support. Agent looks up charged-off balance, collects debit card information over the phone, processes payment manually, applies to GL, triggers EWS update.

| Pros | Cons |
|------|------|
| Minimal engineering build | High per-case ops cost (~15-20 min per call) |
| Can launch quickly | Poor member experience |
| Good fallback if engineering capacity is limited | Doesn't scale if volume increases |
| | Security risk of collecting card info over phone |

### Option 2: Hybrid (recommended MVP)

**How it works:** Member contacts SoFi (via phone or prompted by adverse action letter). Agent verifies identity and directs member to log into SoFi app. Member sees charged-off balance in Banking tab and can pay via debit card (Stripe). Agent remains available for support. Payment auto-applies to GL and triggers EWS reporting.

| Pros | Cons |
|------|------|
| Self-serve payment reduces ops burden | Requires in-app build (balance display + payment flow) |
| Secure (behind SoFi auth, Stripe handles card) | Requires Stripe integration work |
| Payment context is automatic (no ambiguity on which account) | Members must still be able to log in |
| Scales better than Option 1 | |
| Better member experience | |

### Option 3: Fully Self-Serve (aspirational)

**How it works:** Member sees charged-off balance in SoFi app proactively. Can pay and dispute entirely through the app. Automated EWS reporting. Proactive push notifications to eligible members.

| Pros | Cons |
|------|------|
| Best member experience | Most engineering effort |
| Lowest ops cost | Likely not achievable by July 31 |
| Scales to any volume | Dispute self-serve requires document upload, case management UI |

**Recommendation:** Option 2 (Hybrid) for MVP. This balances member experience, ops cost, and build feasibility within the July 31 timeline. Option 1 serves as the fallback if engineering capacity is more constrained than expected.

---

## Appendix B: Comparison to Lending / Credit Card Recovery

This section documents why the Money recovery problem is different from existing lending/credit card recovery processes, to avoid incorrect assumptions from stakeholders with lending context.

### Side-by-side comparison

| Dimension | Lending (Loans) | Credit Card / Smart Card | Money (C&S) |
|-----------|----------------|------------------------|-------------|
| Account status after charge-off | **Remains open.** Loan transitions to "charge-off balance" status. Loan stays open while principal balance AND charge-off balance are both open. Member can still make payments. | **Closed but infra supports ledger.** CC has progressive states: miss 1 payment (delinquent), miss 2 (card open but frozen), then closed. Even after closure, the account ledger infrastructure still supports working on the balance. | **Fully closed.** Account is shut down after 57 days negative. No deposits, no payments, no ledger access. Cannot reopen. |
| Typical balance | Thousands to tens of thousands of dollars | Varies; generally higher than C&S | Likely under $100 (often from ACH returns or timing issues) |
| Collections approach | Recovery unit actively reaches out. Common practice: settle debt for ~60% with installments. Some work outsourced to third parties. | Standard collections path; charge-off referenced at ~180+ DPD | No collections. Flat fee to a collections agency would exceed the recovery amount. Reputational risk not worth it. |
| Payment methods used for recovery | ACH pull from linked bank account (if member agrees to settlement). Wire to SoFi (member-initiated, SoFi pays fee to intermediary). **PayNearMe** used for zero-barrier collection (CC, debit, Venmo, cash). | Smart Card: Autopay from reserved C&S funds, one-time internal payment, external ACH debit from linked bank. | **None today.** No payment channel exists for charged-off C&S accounts. |
| Third-party involvement | Yes. Partners manage wire collection, ensure payment hits the relevant account. PayNearMe handles multi-method collection. SoFi pays per-transaction fees. | Third-party processor for some payment methods | None. Net-new capability needed. |
| Member motivation to repay | Loan obligations, credit score impact, collections pressure, settlement offers (pay less than full amount) | Credit score impact, card reinstatement | Often none until they hit friction elsewhere (e.g., denied new bank account due to EWS flag). Could be months or years later. |
| Reporting destination | Credit bureaus (Experian, Equifax, TransUnion) | Credit bureaus | EWS National Shared Database |
| Existing recovery infrastructure | Yes, robust. Chris Cahill's team handles loan recovery. PayNearMe integration exists. | Smart Card has payment widget and delinquency management (see Appendix E) | None. This is net-new for Money. |
| Settlement/partial payment | Yes. Recovery unit negotiates settlements (e.g., 60% of balance). Installment plans available. | Late fees proposed ($25 first, $41 subsequent, not confirmed live). Collateral sweep planned. | Not for MVP. Full balance only. Partial payments deferred to Phase 2. |

### Key implications for EWS healing

1. **We cannot simply copy the lending model.** Lending accounts stay open and support payments after charge-off. C&S accounts do not. We need a fundamentally different approach (GL-based payment collection).
2. **PayNearMe is already at SoFi.** The lending team uses it for recovery. This is a viable Phase 2 option if we want to offer broader payment methods beyond debit card.
3. **Credit card's progressive delinquency states** (delinquent > frozen > closed-but-ledger-active) are not available for C&S accounts. There is no "closed but ledger-active" state for bank accounts today. The new core migration could potentially enable this, but not before late August 2026.
4. **Settlement offers are common in lending** (settle for 60%) **but not planned for Money MVP.** The balance amounts are too small to justify the complexity. If Phase 2 introduces partial payments, settlement logic could be considered.
5. **The average charge-off balance for Money is unknown.** This is an open question that needs data from Finance or the core banking system. It directly affects the cost-benefit analysis for payment rail selection and whether certain low-balance accounts should simply be written off.

---

## Appendix C: Reuse vs. Build Assessment

| Capability | Existing Asset | Reuse Potential | Notes |
|-----------|---------------|----------------|-------|
| Debit card payment collection | SoFi Plus (Stripe integration) | **High** | Same pattern: collect payment from external card, apply to GL. Need to restrict to debit only. See Appendix F for full SoFi Plus analysis. |
| Multi-method payment collection | PayNearMe (lending recovery) | **Medium** | Already used at SoFi for loan charge-off recovery. Supports CC, debit, Venmo, cash. Phase 2 candidate. See Appendix G for full comparison. |
| Payment UI in Money stack | Smart Card payment flow | **Medium** | Uses UNA external accounts, which may not work for closed-account members. Worth investigating. See Appendix E for full Smart Card payment analysis. |
| Dispute intake | Existing credit bureau dispute process | **Medium** | Different product area and reporting destination, but operational workflow (intake, investigate, resolve, notify) is similar. |
| Case management | Salesforce (if in use for other disputes) | **Medium** | Depends on whether Salesforce is the tool of choice for MET Operations. |
| EWS reporting | EWS contribution pipeline (in development) | **High** | Healing and dispute updates should use the same channel as the initial charge-off reporting. |
| Member communications | Existing email/notification infrastructure | **High** | Standard transactional email patterns apply. Need new templates for healing confirmation and dispute status. |

---

## Appendix D: Regulatory Requirements Summary

Sourced from the EWS Data Contributions BRD and National Shared Database Operating Rules.

| Requirement | Regulation / Source | Timeline | Criticality |
|------------|-------------------|----------|-------------|
| Enable members to repay charged-off balances (heal) | UDAAP, EWS Operating Rules | Must exist before or concurrent with charge-off reporting | **Launch blocker** |
| Report healed accounts to EWS within 10 business days | EWS Operating Rules | 10 business days from confirmed payment | **Launch blocker** |
| Accept and investigate direct data disputes | FCRA Section 623(a)(8) | 30 days (45 with additional info) | **Launch blocker** |
| Determine and notify frivolous disputes | FCRA Section 623(a)(8)(F) | 5 business days | **Launch blocker** |
| Respond to indirect disputes (via CRA) | FCRA Section 623(b) | 30 days | **Launch blocker** |
| Include EWS contact in adverse action notices | FCRA Section 615(a) | Before using EWS data for adverse actions | **Launch blocker** |
| Flag disputed records in contribution files | EWS Operating Rules | While dispute is under investigation | **Launch blocker** |
| Cease furnishing identity theft data until confirmed | FCRA Section 623(a)(6) | Upon receipt of identity theft notice | **Launch blocker** |
| Provide identity theft victims with records within 30 days | FCRA Section 609(e) | 30 days from verified request | Post-launch acceptable |
| Establish written policies for data accuracy | FCRA Section 623(e), EWS Operating Rules | Before contribution goes live | **Launch blocker** |
| Attest to EWS compliance (biennial, risk-based) | EWS Operating Rules | Ongoing | Post-launch acceptable |
| Do not furnish data known to be inaccurate | FCRA Section 623(a)(1) | Ongoing | **Launch blocker** |
| Correct inaccurate data and notify all CRAs | FCRA Section 623(a)(2) | Promptly upon discovery | **Launch blocker** |

---

## Stakeholders

| Team / Person | Role in This Initiative |
|--------------|----------------------|
| **Shweta Kulshrestha** (Product) | PRD owner, define requirements, own the experience, drive execution |
| **Shahar Ronen** (Product, EM) | Executive sponsor, escalation path |
| **Michelle Barnwell** (Money BU) | BRD owner, project driver for broader EWS Data Contributions initiative |
| **Michael Bourgeois** | DRI Approver |
| **Mili Mittal** | Engineering leadership, strategic input |
| **Dheeraj Mehta / Arjun Hegde** (Engineering) | Engineering partner assignment (MACE vs. Money Movement TBD) |
| **Ragini** (Ops Engineering) | Ops component engineering |
| **Tatyana Pacheco, Kristen Krikorian, Brandi Morales-Espinal, Derek Fisher** (Compliance) | FCRA requirements, EWS contract obligations, dispute process rules, disclosure review |
| **Dan Rosner, Arpit Ajmani** (Fraud Strategy) | Fraud-closed account handling, bad actor mitigation |
| **Daniel Evanko** (Data Engineering) | EWS file generation and transmission (contribution pipeline) |
| **Nikita Shah, Erica Hester** (MET Operations) | Agent training, dispute investigation, operational readiness |
| **Srikant Movva** (Product, TBD) | Potential owner of dispute workstream (enable members to dispute and trigger investigation). To be confirmed. |
| **Tami Wilson** (Operations) | Dispute process design and operational readiness, working with Joe Conlon |
| **Joe Conlon** (Specialty Ops) | Ops process design, dispute handling procedures |
| **Sophia Carlton, Dana Wolfe, Diana Barker, Irene Morley** (2nd LOD) | Fraud/Ops/Tech oversight |
| **Krista Thorsen** (Operations Risk) | Operational risk assessment |
| **Sandeep Jayashankar** (Cyber Security) | Payment flow security, data transmission security |
| **Sherrie Osborne** (Privacy) | Data sharing review, PIA assessment |
| **Finance** (TBD) | Recovery GL design, GL codes, tax implications |
| **Design** (TBD) | Member-facing UX for healing and dispute flows |
| **EWS** (external) | API/integration specs, SLAs, testing/certification, dispute rate data |

---

## Appendix E: Smart Card Payment Collection Analysis

This appendix documents how Smart Card handles delinquent payment collection today, as a reference for what can be reused or adapted for the EWS healing payment flow.

### Smart Card Delinquency and Cure Model

- Account becomes **past due the day after the due date** (1 DPD)
- **Cure occurs** the day posted payments meet or exceed the delinquent billing-cycle amount due
- Payments apply **FIFO** against the oldest open delinquency cycle
- External ACH cures on posting (not initiation), after a 2 to 3 day hold
- No fixed cure window; member cures whenever the threshold is met with posted payments
- If delinquency persists, account proceeds through standard collections; charge-off referenced at ~180+ DPD (policy details subject to finalization)

### Smart Card Payment Methods (Today)

| Method | Description | Relevance to EWS Healing |
|--------|------------|------------------------|
| **Autopay from reserved funds (C&S)** | Automatic debit from SoFi Checking & Savings | Not applicable. EWS healing members have closed C&S accounts. |
| **One-time payment from reserved funds** | In-app Payment widget, funds from C&S | Not applicable. Same constraint as above. |
| **One-time external ACH debit** | Linked non-SoFi bank account via UNA | **Potentially reusable.** However, charged-off C&S members may not have linked external accounts. Would need to support linking a new external account post-closure. Posts after 2 to 3 day hold; reversals are notified. |

**Key takeaway:** Smart Card's payment methods all assume the member has either an active SoFi C&S account (reserved funds) or a pre-linked external bank account (UNA). Neither assumption holds for charged-off C&S members. This limits direct reuse of Smart Card's payment infrastructure for EWS healing. The **external ACH debit** pattern is the closest match, but would require enabling account linking for members whose C&S accounts are closed.

### Smart Card Notifications and Comms

| Notification | Timing | Relevance to EWS Healing |
|-------------|--------|------------------------|
| Payment-due reminder | 3 days before due date | Adapt pattern for proactive healing outreach (Phase 2) |
| Past-due notice | Day after missed due date | Adapt pattern for charge-off notification with EWS consequences |
| ACH reversal/failure notice | On external payment failure | Directly reusable for healing payment failure notifications |

Program work tracker confirms "Upcoming Payments & Past Due Comms" launched 1/23/2026.

### Smart Card Member Experience Flow

The end-to-end Smart Card payment flow provides a reference UX pattern:

1. **Before due date:** Upcoming payment reminder (<=3 days). If Autopay is ON, widget shows "Autopay on" banner; otherwise shows a Pay CTA.
2. **1 DPD (day after due date):** Account flips to Past Due; past-due notice sent; widget shows Past Due with amount due and Pay CTA.
3. **Payment options and holds:** Member can pay via one-time internal (reserved funds), Autopay (if enabled), or initiate external ACH. External ACH typically posts after 2 to 3 days; cure occurs on posting, not initiation.
4. **Posting and cure:** Partial payments accumulate toward the delinquent-cycle threshold; while below threshold, widget shows "Partially Paid." Once posted payments meet/exceed the threshold, status returns to Current and widget shows "No Payment Due" (Autopay banner if ON).
5. **If payment fails (external):** ACH reversal notification is sent; account remains Past Due until a successful posted payment cures.

### Smart Card Payment Widget UI States

| State | Description | Adaptable for EWS Healing? |
|-------|------------|--------------------------|
| No Payment Due | No CTA; shows Autopay banner if ON | Yes, adapt as post-healing confirmation ("Balance Paid, EWS Update Pending") |
| Past Due | Shows amount due + Pay CTA | **Yes, strong match.** Adapt as the primary healing state ("Charged-Off Balance: $XX.XX" + Pay CTA) |
| Partially Paid | Shows remaining amount to cure | Defer to Phase 2 (MVP is full payment only) |
| Full Payment | Confirmation state after full payment | Yes, adapt as healing payment confirmation |
| Insufficient Funds | For internal payments without enough available balance | Not applicable (no internal funds for closed-account members) |

### Smart Card: What Happens If Member Does Not Cure

| Dimension | Smart Card Behavior | EWS Healing Comparison |
|-----------|-------------------|----------------------|
| Status and counters | Account remains Past Due; delinquency buckets roll forward each cycle | EWS healing has no rolling delinquency; balance is static (charged-off amount) |
| Comms and payment attempts | Ongoing past-due comms; Autopay retries on 5th of month | EWS healing has no recurring attempts for MVP (member-initiated only) |
| Spending and rewards | Authorization restrictions may apply; proposals to revoke rewards during delinquency | Not applicable (no active account) |
| Fees and sweeps | Late-fee structure proposed ($25 first, $41 subsequent, not confirmed live). Automated collateral sweep planned (not live yet). | Not applicable (no fees on charged-off C&S balance) |
| If delinquency persists | Account proceeds through standard collections; charge-off ~180+ DPD | Already charged off. No further escalation. Member lives with EWS flag until they choose to heal. |

### Reuse Assessment Summary for EWS Healing

| Smart Card Component | Reuse Verdict | Notes |
|---------------------|---------------|-------|
| External ACH debit payment rail | **Medium** | Closest payment method match, but requires UNA account linking for closed-account members. Evaluate feasibility. |
| Payment widget UI pattern | **High** | "Past Due" and "Full Payment" states map well to healing flow. Adapt copy and CTA. |
| FIFO payment application logic | **Low** | EWS healing is full balance payment (MVP). No multi-cycle delinquency to manage. |
| Notification/comms templates | **Medium** | Past-due and payment failure templates adaptable. Need new templates for EWS-specific messaging. |
| Autopay / reserved funds flow | **None** | Not applicable; members don't have active C&S accounts. |
| Partial payment tracking | **Defer** | Relevant if Phase 2 introduces partial payments. |

### Sources

- Smart Card MVP Account Statuses
- Thread between Andy, Aruna, and Sriram
- Smart Card Daily Updates
- Smart Card - Collateral Sweep
- Event Tracking for Smart Card Comms: Use Cases and Parameters
- Credit Card & Smart Card - 2026 Planning

---

## Appendix F: SoFi Plus Payment Collection Analysis

This appendix documents how SoFi Plus collects subscription payments today, as a reference for what can be reused or adapted for the EWS healing payment flow. SoFi Plus is the strongest reuse candidate because it already collects payments from members into a SoFi GL (not a member account), which mirrors the EWS healing requirement.

### SoFi Plus Subscription Model

- $10 subscription every 30 days
- As of 2026-03-31, Direct Deposit (DD) and Qualifying Alternative Deposits (QuAD) no longer qualify for Plus; members must pay to keep Plus benefits
- Core banking benefits tied to DD/QuAD continue without Plus

### SoFi Plus Payment Methods (Today)

| Method | Description | Relevance to EWS Healing |
|--------|------------|------------------------|
| **SoFi Checking auto-debit** | Automatic debit on member's unique monthly billing date (~5 AM ET). Funds must be in Checking (not Savings). Posts after ACH processing (2 to 4 business days). | **Not applicable.** EWS healing members have closed C&S accounts; no Checking balance to debit. |
| **Debit or credit card on file** | Member adds card in Membership & Rewards > Manage membership. Prepaid cards not accepted. | **Highly reusable.** This is the closest match to the EWS healing payment rail. Card-on-file collection, funds to GL. For EWS healing: restrict to debit only (no credit cards per Mili's guidance). |

**Key takeaway:** SoFi Plus already collects payments via debit/credit card and routes funds to a SoFi GL rather than a member account. This is exactly the pattern needed for EWS healing. The main adaptations needed are: (1) restrict to debit card only, (2) change the payment context from "subscription fee" to "charged-off balance repayment," and (3) connect the payment confirmation to EWS reporting.

### SoFi Plus Notifications and Comms

| Notification | Timing | Relevance to EWS Healing |
|-------------|--------|------------------------|
| Welcome/activation messages | On join | Adapt for healing payment initiation confirmation |
| Payment receipts | After successful charge | **Directly reusable.** Same pattern needed for healing payment confirmation. |
| Reminder to update payment method | After charge fails | Adapt for healing payment failure retry notification |
| Cancellation notification (payment failure) | After grace period expires | Not directly applicable (healing is one-time, not recurring) |
| Cancellation notification (member-initiated) | On cancellation | Not applicable |

### SoFi Plus Member Experience Flow

The end-to-end SoFi Plus payment flow provides a reference UX pattern:

1. **Join / Subscribe:** Member subscribes in app/web via Membership & Rewards for $10 every 30 days. Payment method (Checking auto-debit or card on file) is set during enrollment.
2. **Billing cycle and posting:** Billed every 30 days. ACH charges from Checking may take 2 to 4 business days to appear. Billing date visible in Manage Membership.
3. **If payment fails:** Member receives reminders to update payment method. A short grace period is provided to retry before membership is canceled. Common failure reasons: insufficient funds in Checking or declined debit/credit card.
4. **After cancellation (payment failure):** Plus benefits end immediately. Member must re-enroll and pay $10 to regain benefits. Core banking benefits tied to DD/QuAD continue.
5. **After cancellation (member-initiated):** Benefits continue until end of current cycle, then stop.

### SoFi Plus UI Pathways

| Platform | Path |
|----------|------|
| App | Home > Plus icon > settings, or Profile > Membership & Rewards > settings to view billing date, update payment method, cancel, or rejoin |
| Web | Profile (top-right) > Membership & Rewards > Manage your membership to update/cancel/rejoin |

**Relevance to EWS Healing:** The "Manage membership" screen pattern (view status, update payment method, take action) maps well to a "Manage charged-off balance" screen (view balance, add payment method, pay).

### SoFi Plus: What Happens If Member Does Not Cure

| Dimension | SoFi Plus Behavior | EWS Healing Comparison |
|-----------|-------------------|----------------------|
| Cure definition | Subscription cured when payment succeeds within grace period | Healing "cured" when full balance is paid and posted |
| Time to cure | Short grace period (exact duration varies operationally) | No time limit; member can heal at any point (EWS flag persists up to 5 years) |
| If not cured | Membership canceled; Plus benefits removed. Member can rejoin anytime by paying $10. | EWS flag remains. Member can still pay at any point in the future to heal. |
| Recurring vs. one-time | Recurring (every 30 days) | One-time payment to clear charged-off balance |

### Reuse Assessment Summary for EWS Healing

| SoFi Plus Component | Reuse Verdict | Notes |
|--------------------|---------------|-------|
| **Debit/credit card payment rail (Stripe)** | **High** | Core payment infrastructure is directly reusable. Restrict to debit only for EWS healing. This is the recommended primary payment method for MVP. |
| **GL routing (funds to SoFi, not member account)** | **High** | Same pattern: collected funds go to a SoFi ledger, not a member account. Finance needs to set up the specific recovery GL, but the plumbing exists. |
| **Payment method management UI** | **High** | Add/update card flow in Manage Membership can be adapted for healing payment method entry. |
| **Payment receipt generation** | **High** | Same transactional email pattern. New template needed for healing-specific copy. |
| **Payment failure retry flow** | **Medium** | Grace period and retry pattern is useful reference, though healing is one-time (not subscription). Adapt for "payment failed, try again" UX. |
| **Subscription billing logic** | **Low** | Recurring billing not needed; healing is a one-time payment. |
| **Cancellation flow** | **None** | Not applicable to healing. |

### Why SoFi Plus is the Recommended Payment Rail for EWS Healing

| Criteria | SoFi Plus (Stripe) | Smart Card (UNA/ACH) |
|----------|-------------------|---------------------|
| Works for closed-account members | **Yes.** Card payment is independent of C&S account status. | **Partially.** External ACH requires a linked external account; members may not have one. |
| Funds route to GL (not member account) | **Yes.** Already routes to SoFi GL. | **No.** Smart Card payments apply to a credit card balance, not a GL. Would need rearchitecting. |
| Member needs active SoFi account | **No.** Just needs to be able to log in and provide a card. | **Yes.** UNA linking typically requires an active account context. |
| Payment method flexibility | Debit card (and credit, but we'd restrict to debit only) | External ACH only (plus internal funds, which aren't available) |
| Implementation effort to adapt | **Low to Medium.** Same Stripe integration, new payment context, new GL destination. | **Medium to High.** Would need to enable UNA linking post-closure and reroute funds to a GL. |
| Posting time | Near-instant for card payments | 2 to 3 business days for ACH |

**Recommendation confirmed:** Use the SoFi Plus / Stripe payment rail as the foundation for EWS healing payments. Adapt the card-on-file collection flow, restrict to debit only, route funds to the recovery GL, and connect payment confirmation to the EWS healing report trigger.

### Sources

- SoFi Plus Overview (Operations)
- SoFi Plus Benefits Resource (Operations)
- SoFi Plus changes tied to Smart Card Launch (Operations)
- Troubleshooting Plus Subscription Resource, SoFi Plus (Operations)
- SoFi Plus 3.0 Comms & UX Copy
- Optional Card on File Feature for SoFi Plus Membership, SoFi Plus (Operations)
- PRD: CC Backbook Monthly Membership Fee

---

## Appendix G: Payment Rail Options (Full Comparison)

This appendix consolidates all payment rail options discussed across working sessions, Slack threads, and stakeholder meetings into a single decision matrix.

### Option 1: Stripe (Debit Card) via SoFi Plus Pattern -- RECOMMENDED FOR MVP

| Attribute | Detail |
|-----------|--------|
| **How it works** | Member enters debit card in-app. Payment processed via Stripe. Funds routed to recovery GL. |
| **Existing precedent** | SoFi Plus subscription payments (see Appendix F) |
| **Accepted methods** | Debit card only (restrict from credit card per Mili's guidance; no prepaid cards) |
| **Works for closed accounts** | Yes. Card payment is independent of C&S account status. |
| **Payment context** | Solved: in-app flow ties payment to member identity and specific charged-off account. |
| **GL routing** | Yes. SoFi Plus already routes to a GL. New recovery GL code needed from Finance. |
| **Posting time** | Near-instant for card transactions |
| **Implementation effort** | Low to Medium. Reuse existing Stripe integration; new payment context and GL destination. |
| **Per-transaction cost** | Stripe processing fee (~2.9% + $0.30 per transaction). On a $50 balance, that's ~$1.75. Acceptable. |
| **Member friction** | Low. Enter card number, confirm amount, pay. |
| **Compliance considerations** | PCI DSS compliance via Stripe. Reg E disclosures for electronic fund transfer. Confirm acceptable payment methods with Compliance. |

### Option 2: External ACH Debit via Smart Card / UNA Pattern

| Attribute | Detail |
|-----------|--------|
| **How it works** | Member links an external bank account (via UNA/Plaid). SoFi initiates ACH debit pull. |
| **Existing precedent** | Smart Card payment flow (see Appendix E) |
| **Accepted methods** | ACH debit from linked external bank |
| **Works for closed accounts** | Partially. UNA linking typically requires an active account context. Charged-off members may not have linked accounts, and enabling new linking post-closure adds complexity. |
| **Payment context** | Partially solved. If in-app, context is known. If initiated externally, requires matching. |
| **GL routing** | Would need rearchitecting. Smart Card payments apply to a credit card balance, not a GL. |
| **Posting time** | 2 to 3 business days after initiation. Cure on posting, not initiation. |
| **Implementation effort** | Medium to High. Enable UNA linking for closed-account members; reroute funds to GL. |
| **Per-transaction cost** | Lower than card (~$0.20 to $0.50 per ACH transaction) |
| **Member friction** | Medium. Requires linking an external account (Plaid flow), which may fail or be unfamiliar. |
| **Compliance considerations** | NACHA rules apply. ACH reversal handling needed. |

### Option 3: PayNearMe (Multi-Method Third-Party) -- EVALUATE FOR PHASE 2

| Attribute | Detail |
|-----------|--------|
| **How it works** | Third-party platform. Member receives a payment link or barcode. Can pay via CC, debit, Venmo, cash (at retail locations), Apple Pay, Google Pay, and more. |
| **Existing precedent** | Used by SoFi's lending recovery team today for loan charge-off collections. |
| **Accepted methods** | CC, debit, Venmo, cash, Apple Pay, Google Pay, bank transfer. Zero barrier to entry. |
| **Works for closed accounts** | Yes. Completely independent of SoFi account status. |
| **Payment context** | Solved. PayNearMe generates a unique payment link per member/account. SoFi provides the balance; PayNearMe handles collection and remits to SoFi. |
| **GL routing** | PayNearMe remits collected funds to SoFi. Need to confirm how funds are routed (single deposit to GL with reconciliation file, or per-transaction). |
| **Posting time** | Varies by method. Card/digital wallet: near-instant. Cash: same day. Bank transfer: 2 to 3 days. |
| **Implementation effort** | Medium. Integration already exists for lending. Need to extend to Money recovery context with new GL destination. |
| **Per-transaction cost** | SoFi pays a per-transaction fee to PayNearMe. Need to evaluate if cost is justified for small balances (< $100). For a $40 charge-off, the fee could represent a significant percentage of recovery. |
| **Member friction** | Very low. Member clicks a link and pays however they want. No SoFi login required. |
| **Compliance considerations** | Need to confirm acceptable payment methods with Compliance (e.g., is CC acceptable for charge-off repayment?). PayNearMe handles PCI compliance. Third-party data sharing considerations. |

### Option 4: Wire / ACH Push to SoFi Account Number

| Attribute | Detail |
|-----------|--------|
| **How it works** | SoFi provides a routing number and account number (for the recovery GL or a designated collection account). Member initiates a wire or ACH push from their external bank. |
| **Existing precedent** | Used in lending recovery (member sends wire; third-party partner ensures payment hits the right account). |
| **Accepted methods** | Wire transfer, ACH push from external bank |
| **Works for closed accounts** | Yes. Push payment is independent of SoFi account status. |
| **Payment context** | NOT solved. SoFi receives funds but cannot automatically match them to a specific charged-off account. Requires member to separately notify SoFi (call or email) to provide context. |
| **GL routing** | Funds arrive at the designated account. Manual reconciliation needed to apply to the correct charged-off account. |
| **Posting time** | Wire: same day. ACH: 1 to 3 business days. FedNow: near-instant (if supported). |
| **Implementation effort** | Low. No new payment infra needed on SoFi side. But high ops effort for manual reconciliation. |
| **Per-transaction cost** | Wire fees vary ($15 to $30 for domestic wire). ACH: minimal. FedNow: minimal. |
| **Member friction** | High. Member must know the routing/account number, initiate the transfer from their bank, and then contact SoFi to confirm. |
| **Compliance considerations** | Standard BSA/AML monitoring. Wire requires proper identification. |

### Option 5: Payment Website with Case Number

| Attribute | Detail |
|-----------|--------|
| **How it works** | SoFi builds a standalone web page. Member receives a unique case number (via letter, email, or phone agent). Enters case number on the site, sees their balance, and pays via Stripe. |
| **Existing precedent** | No direct SoFi precedent. Common pattern in utilities and medical billing. |
| **Accepted methods** | Debit card (via Stripe). Could expand to other methods. |
| **Works for closed accounts** | Yes. Independent of SoFi account status. Also works for members who can't log in. |
| **Payment context** | Solved. Case number maps to the specific charged-off account. Member sees balance and pays. |
| **GL routing** | Same as Stripe option. Funds to recovery GL. |
| **Posting time** | Near-instant for card payments. |
| **Implementation effort** | Medium to High. Requires building a new standalone web page, case number generation and management, and security review. |
| **Per-transaction cost** | Same as Stripe (~2.9% + $0.30). |
| **Member friction** | Medium. Must have the case number. But does not require SoFi app login. |
| **Compliance considerations** | Security concerns: case number as sole authentication is weak. Could be guessed or intercepted. Would need additional verification (e.g., last 4 SSN, DOB). |

### Decision Matrix

| Criteria | Stripe (Debit) | External ACH | PayNearMe | Wire/ACH Push | Payment Website |
|----------|---------------|-------------|-----------|--------------|----------------|
| Works for closed accounts | Yes | Partial | Yes | Yes | Yes |
| Payment context auto-solved | Yes (in-app) | Yes (in-app) | Yes (unique link) | **No** | Yes (case number) |
| GL routing ready | Yes (adapt SoFi Plus) | Needs rework | Needs config | Manual | Yes (adapt Stripe) |
| Implementation effort | **Low-Med** | Med-High | Medium | Low (but high ops) | Med-High |
| Member friction | **Low** | Medium | **Very Low** | High | Medium |
| Per-transaction cost | ~$1.75 on $50 | ~$0.35 on $50 | TBD (vendor fee) | $15-30 (wire) / ~$0.35 (ACH) | ~$1.75 on $50 |
| Requires SoFi login | Yes | Yes | **No** | No | **No** |
| Phase | **MVP** | Phase 2 | **Phase 2** | Manual fallback | Phase 2/3 |

### Recommendation

**MVP (July 31):** Stripe debit card via SoFi Plus pattern, behind SoFi app login. Covers the primary use case (member who can log in, wants to pay, needs payment applied to GL and reported to EWS). Agent-assisted fallback for members who call in.

**Phase 2:** Evaluate PayNearMe for members who cannot or will not log in. Already integrated at SoFi for lending. Offers the broadest payment method flexibility with the lowest member friction. Key question: is the per-transaction cost justified for small balances?

**Manual fallback (available from MVP):** Wire/ACH push for edge cases where neither in-app nor PayNearMe works. Requires ops involvement for reconciliation.

---

## Reference Documents

- [EWS Data Contribution BRD](link) (Michelle Barnwell)
- [EWS Data Contrib Project Plan](link) (Michelle Barnwell)
- [EWS Data Contribution: Charge Off Healing and FCRA Disputes](link) (working doc)
- [Gil's Q&A on EWS](link)
- [EWS National Shared Database Operating Rules](link)
- [FCRA Requirements Reference](link)
- [Existing Member Credit Reporting Disputes Resources](link)
- [FCRA Disputes Process](link)
- [Predict New Account Risk Technical Doc](link)
- Smart Card MVP Account Statuses
- Smart Card Daily Updates
- Smart Card - Collateral Sweep
- Event Tracking for Smart Card Comms: Use Cases and Parameters
- Credit Card & Smart Card - 2026 Planning
- SoFi Plus Overview (Operations)
- SoFi Plus Benefits Resource (Operations)
- SoFi Plus changes tied to Smart Card Launch (Operations)
- Troubleshooting Plus Subscription Resource, SoFi Plus (Operations)
- SoFi Plus 3.0 Comms & UX Copy
- Optional Card on File Feature for SoFi Plus Membership, SoFi Plus (Operations)
- PRD: CC Backbook Monthly Membership Fee

---

## Appendix H: Competitive Landscape -- How Others Handle Charge-Off Recovery

### Summary

No bank or fintech has built an elegant, self-service charge-off healing experience for deposit accounts. The industry standard is overwhelmingly ops-heavy: members call a number, negotiate with a collections department or third-party agency, pay by phone or mail, and then separately request their ChexSystems/EWS record be updated. SoFi has an opportunity to differentiate by offering a digital-first, low-friction recovery path.

### Traditional banks

| Dimension | Chase | Wells Fargo | Bank of America | Capital One |
|-----------|-------|-------------|-----------------|-------------|
| **Charge-off timeline** | 60 to 90 days negative | 180+ days delinquent | 120 to 180 days delinquent | 180+ days delinquent |
| **Account status after charge-off** | Permanently closed | Permanently closed | Permanently closed | Permanently closed; almost never reopened even after full payment |
| **Recovery method** | Internal collections dept or sold to third-party agency. Member calls or receives letter with payment instructions. | Internal recovery team pursues payment. Can exercise "right of set-off" (seize funds from other WF accounts to cover debt). | Internal collections or outsourced to agency. Member must call to negotiate. | Debt typically sold to collections. Even paying full balance only updates report to "paid charge-off," does not reopen account. |
| **Payment channels** | Phone (agent-assisted), mail (check/money order), sometimes online if member still has active Chase relationship | Phone, mail, or in-branch. Set-off is automatic for members with other WF accounts. | Phone, mail | Phone, mail to collection agency |
| **Self-service digital experience** | None for charged-off deposit accounts | None for charged-off deposit accounts | None for charged-off deposit accounts | None for charged-off deposit accounts |
| **Settlement offers** | Case-by-case; 20 to 50% reduction possible | Case-by-case | Case-by-case | Case-by-case |
| **ChexSystems/EWS update** | Updated after payment confirmed; no guaranteed removal, status changes to "paid" | Updated after payment; record stays 5 years but status changes | Updated after payment; record stays 5 years | Updated after payment; record persists |
| **Second-chance account** | Chase Secure Banking ($4.95/mo) | Clear Access Banking ($5/mo, waivable) | SafePass Secured Account | 360 Checking (no ChexSystems check) |

**Key takeaway:** Traditional banks treat charge-off recovery as a back-office collections operation. There is no digital self-service path. Members must navigate phone trees, negotiate with agents, and often deal with third-party collection agencies. The experience is adversarial, not member-friendly.

### Neobanks and fintechs

| Dimension | Chime | Varo | Ally |
|-----------|-------|------|------|
| **Negative balance handling** | SpotMe covers up to $200 in overdrafts at no fee. Negative balance auto-clears when next direct deposit arrives. | Varo Advance: 15 to 30 day repayment window, then 30-day final due date. Auto-deducted from incoming deposits. | CoverDraft: up to $250 overdraft relief, 14-day window to bring balance to $0. Auto-applies future deposits to negative balance. |
| **Account closure trigger** | Extended negative balance, fraud, AML, or repeated ToS violations | Failure to repay Varo Advance; eligibility blocked for 6 months after charge-off | 45 days of negative balance beyond CoverDraft limit |
| **Recovery after closure** | Must clear negative balance before closure is processed. No self-service recovery portal for already-closed accounts. Contact support at (844) 244-6363. | Unpaid balances deducted from incoming deposits to any authorized account. No dedicated recovery portal. | Account restrictions after 14 days; closure after 45 days. No public recovery flow for charged-off accounts. |
| **EWS/ChexSystems posture** | Does not use ChexSystems for account opening (competitive differentiator). However, still reports negative data. | Does not use ChexSystems for opening. Reports negative data to EWS. | Uses ChexSystems for opening. Reports negative data. |
| **Digital recovery experience** | None for post-closure. Pre-closure: negative balance shown in-app with auto-repayment. | None for post-closure. Pre-closure: in-app advance tracking with repayment date. | None for post-closure. Pre-closure: in-app balance visibility with 14-day countdown. |

**Key takeaway:** Neobanks are better at preventing charge-offs (auto-repayment from deposits, smaller overdraft limits, shorter grace periods) but equally poor at post-closure recovery. None offer a digital healing experience after account closure.

### Industry patterns and gaps

| Pattern | What the industry does | SoFi opportunity |
|---------|----------------------|-----------------|
| **Pre-charge-off intervention** | Auto-debit from incoming deposits (Chime, Varo, Ally). Reminders and grace periods. | SoFi already does 57-day grace with reminders. Plaid balance work could further reduce charge-offs. |
| **Post-charge-off collections** | Phone-based, agent-assisted, or outsourced to third-party collectors. No digital self-service. | **Build a digital-first in-app recovery experience.** No competitor does this today. |
| **Payment methods for recovery** | Phone payments (agent takes card info), mail (check/money order), wire. Sometimes set-off from other accounts at same bank. | Stripe debit card (SoFi Plus pattern) for MVP. PayNearMe multi-channel for Phase 2 (CC, debit, Venmo, cash at retail). |
| **ChexSystems/EWS record update** | Manual process. Member pays, then must separately request record update. Banks update within varying timelines (no standard). | **Automate the EWS update.** Payment triggers status change, reported in next daily batch (within 10 business days per contract). Member never has to make a separate request. |
| **Account reopening** | Almost universally not offered. Capital One explicitly does not reopen. Traditional banks offer "second chance" accounts as alternatives. | Not in MVP scope, but possible in future with new core banking system. |
| **Settlement/partial payment** | Common in lending. Rare for deposit account charge-offs due to small balances. | Phase 2 consideration. Most Money charge-offs are likely < $100, so full payment is more practical than settlement. |
| **Transparency** | Members often discover the charge-off only when denied at another bank. No proactive notification about EWS impact. | **Proactive communication.** Update pre-charge-off warnings to mention EWS reporting. Post-charge-off: clear in-app messaging with exact balance owed and payment path. |
| **Third-party payment platforms** | PayNearMe used by lending divisions for multi-method collections (CC, debit, Venmo, cash at 31,000+ retail locations). Not used for deposit account recovery at any major bank. | Phase 2: PayNearMe integration would give members the widest payment method choice. Particularly valuable for members without an active SoFi account (logged-out experience). |

### Competitive positioning for SoFi

If SoFi builds a digital-first healing experience, it would be **first-to-market** among major banks and fintechs for deposit account charge-off recovery. Specific differentiators:

1. **In-app balance visibility.** No competitor shows the charged-off balance in their app with a clear "pay now" CTA. Members at other banks must call to learn their balance.
2. **One-tap payment.** No competitor offers digital payment for charged-off deposit accounts. Every other bank requires a phone call or mailed payment.
3. **Automated EWS/ChexSystems update.** No competitor automatically updates the record upon payment. Members at other banks must call or write separately to request the update.
4. **Transparent timeline.** Showing the member "once you pay, your EWS record will be updated within 10 business days" is better than the industry norm of vague promises.
5. **No adversarial collections.** Unlike traditional banks that send debts to collection agencies (damaging the member relationship), SoFi's approach treats recovery as a member service, not a collections operation.

This aligns with SoFi's brand positioning as a member-first financial institution and could generate positive press coverage ("SoFi becomes first bank to offer digital charge-off recovery").

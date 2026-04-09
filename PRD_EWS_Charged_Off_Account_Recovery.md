# EWS Charged-Off Account Recovery: Payment Collection & Data Dispute

**Team:** Money
**Status:** Draft

**Drivers:** Shahar Ronen, Shweta Kulshrestha

**Accountable:** Mili Mittal (Not reviewed)

**Exec Sponsor:** Kate Matsudaira

**Consulted:**

| Role | People | Status |
|------|--------|--------|
| BU | Michelle Barnwell | Not reviewed |
| Fraud Strategy | Dan Rosner, Arpit Ajmani | Not reviewed |
| EPD Product & EM | Shahar Ronen / Dheeraj Mehta | Not reviewed |
| Data Engineer | Daniel Evanko | Not reviewed |
| Compliance | Tatyana Pacheco, Kristen Krikorian, Brandi Morales-Espinal, Derek Fisher | Not reviewed |
| 2nd LOD (Fraud / Ops / Tech) | Sophia Carlton, Dana Wolfe, Diana Barker, Irene Morley | Not reviewed |
| Specialty Ops | Joe Conlon | Not reviewed |
| MET Operations | Nikita Shah, Erica Hester | Not reviewed |
| Operations Risk | Krista Thorsen | Not reviewed |
| Cyber Security | Sandeep Jayashankar | Not reviewed |

**Informed:** TBD

**Related docs:**
- [EWS Data Contribution BRD](link)
- [EWS Data Contrib Project Plan](link)
- [EWS Data Contribution: Charge Off Healing and FCRA Disputes](link)

**Slack channel:** #ews-data-contribution

**Changelog:**
- Apr 9, 2026 (v5): Removed phasing. This is a single-phase delivery (MVP = final scope). All former "Phase 2" items (debit card, guest portal, agent-assisted payment, etc.) moved to Future Considerations appendix. Updated all sections throughout.
- Apr 8, 2026 (v4): Simplified MVP to ACH push only. Deferred debit card payment and agent-assisted payment. MVP is in-app self-service with a single rail (ACH push from external bank). Updated all sections: TL;DR, In-Scope, Recommended Solution, experience flows, user stories, decisions, payment rails, scope, edge cases, and mockups.
- Apr 8, 2026 (v3): Restructured PRD section order (TL;DR, Background, Problem Statement, Data, Scope, Current Experience, then Proposed Solution). MVP is in-app only + agent fallback. Updated all references throughout.
- Apr 8, 2026 (v2): Major revision. Added "Recommended Solution: Authenticated Pay Page with Two Rails" (ACH push primary, debit card secondary). Rewrote Ideal Healing Flow to be self-service-first. Updated In-Scope, user stories, experience detail flows, and agent model. Incorporated Apr 8 Shahar/Shweta deep dive findings throughout. Updated payment rail analysis, open questions, and next steps.
- Apr 2, 2026: First draft

---

## TL;DR

Today, members whose Money C&S accounts are charged off (57 days negative) have no way to repay or dispute. This blocks SoFi from reporting charge-off data to EWS, costing **~$100K/month** in excess fees and risking loss of the **New Account Risk** product by **Dec 31, 2026**. By **July 31, 2026**, we plan to launch a self-service payment experience in the SoFi app via ACH push from external bank (member-initiated, zero return risk), dispute intake via mail, and begin EWS reporting to satisfy EWS contractual requirements and give charged-off members a path to heal their record.


---

## Background

Early Warning Services (EWS) operates the **National Shared Database**, a shared industry resource used by financial institutions for deposit account screening and fraud prevention. EWS is operated by a consortium of the largest U.S. banks. Its products, including **New Account Risk**, help detect identity fraud and first-party fraud during account opening. Risk conducted a data study and believes New Account Risk would help reduce approval rate by 6 to 7% and overall fraud losses by 30%.

SoFi uses EWS's New Account Risk product for onboarding risk scoring. Today, we send C&S application data to EWS and they return risk elements used by Fraud Strategy for approve/deny decisions. Our contract requires us to **contribute data back** to the National Shared Database. Normally, participants must contribute data before using the service, but SoFi received a waiver to complete this work no later than **December 31, 2026**.

The pricing structure incentivizes contribution: **$2.9M for 3 years** as a non-contributor vs. **$1.5M** as a contributor. Until we deliver the first set of contribution files, we are paying approximately **$100k/month** in excess fees. If we fail to meet the December 31, 2026 deadline, EWS has the right to turn off our access to New Account Risk and any other services we use.

Once we begin reporting charge-offs to EWS, two obligations kick in:

1. **Healing (UDAAP requirement).** We cannot report a member as having a charged-off account if we do not allow them to repay and clear that record. Doing so would cause consumer harm with no recourse, which is the definition of unfair under UDAAP. The member has **up to 5 years** from when the charge-off is reported to EWS to pay and clear the flag (after 5 years the flag expires automatically). Once the member pays, SoFi must update EWS within **10 business days** per the Operating Rules.

2. **Dispute (FCRA requirement).** As a data furnisher to a consumer reporting agency (EWS), SoFi is subject to FCRA furnisher obligations, which include the member's right to dispute the reported data. The right to dispute covers any data we report (wrong balance, wrong address, not closed due to fraud, account not theirs, etc.) at any point while the record exists, up to **5 years**. Once a dispute is filed, SoFi must investigate within **30 days** (45 if additional info is provided), correct inaccuracies, and propagate corrections to all CRAs that received the data. Frivolous disputes must be identified and the member notified within **5 business days**.

When a member's C&S account goes negative, SoFi provides a **57-day grace period** with reminders to repay. If the member does not bring the balance to zero within that window, the account is closed and the balance is charged off. Here is the current SOP for MET agents if a member calls in for negative balance sweeps, charge-offs, and overdrawn Account. After charge-off:
- The account is **permanently closed**. There is no mechanism to reopen it or deposit into it.
- There is **no way for the member to repay**. Even if they want to, no payment channel exists.
- If the member reapplies for a SoFi Money account in the future, they are **declined**.
- Once we begin reporting to EWS, the charge-off will appear in the National Shared Database, potentially blocking the member from opening accounts at other financial institutions for up to 5 years.

**Financial impact.** SoFi is paying approximately **$100k/month** in excess fees to EWS until we begin contributing data. Full contribution compliance (including healing and dispute capabilities) is required by **December 31, 2026**, or EWS can terminate our access to New Account Risk and other services. The pricing difference is significant: $2.9M (non-contributing) vs. $1.5M (contributing) over 3 years.

**Contract obligation.** Per the EWS Operating Rules, if a consumer pays off or otherwise settles Account Abuse, the bank must report the update within **10 business days**. We cannot fulfill this obligation without a payment collection mechanism.

### The broader initiative

This PRD covers the healing and dispute experience workstreams of the larger EWS Data Contributions initiative. The full initiative includes:

| Workstream | Owner | Status |
|-----------|-------|--------|
| EWS file generation, transmission and data contribution | Daniel Evanko (Data Eng), Kurtis (transmission) | In progress |
| Collect healing payment from member and Apply payment to recovery GL | Shweta Kulshrestha | This PRD |
| Enable member dispute and investigation | Money EPD and Special Ops (Joe Conlan, Mitchel Morris Jr) | This PRD |


---

## Problem Statement

SoFi is contractually required by Early Warning Services (EWS) to contribute Checking & Savings account data to the National Shared Database. As part of this obligation, when a member's account is charged off and reported, SoFi must provide a path for the member to:

1. **Repay the charged-off balance** ("heal" their record) so the negative flag is removed from EWS
2. **Dispute data they believe is inaccurate**, with investigation and resolution within FCRA-mandated timelines

Today, **neither capability exists at SoFi**. There is no member-facing experience, no operational workflow, and no system integration to process healing payments or manage EWS data disputes for charged-off deposit accounts.


---

## Data Findings

_All data queried April 2, 2026, from production Snowflake via `snow sql -c sofi`._

### A note on data sources

Two Snowflake tables track charge-off closures, and they report different numbers:

- **`MONEY_ACCOUNT_CLOSURES`** is a curated, deduplicated table: one row per account, representing the final closure state. This is the authoritative source for unique account counts but was last refreshed October 6, 2025.
- **`ACCOUNT_TO_CLOSE_EVENTS`** is an event stream that captures the full lifecycle of a closure (started, pending, closed, canceled). A single account can generate multiple events across statuses, so raw counts overstate unique accounts by approximately 1.57x.

The numbers below come from **transaction-level data** (`RR_BANKING_TRANSACTIONS_FACTS`), which counts the actual charge-off transaction posted to each account and is the most reliable per-month source.

### Monthly charge-off volume (last 6 months)

| Month | Accounts | Avg Balance | Median Balance | Monthly Loss |
|---|---:|---:|---:|---:|
| Mar 2026 | 7,722 | $140 | $49 | $1.08M |
| Feb 2026 | 7,164 | $156 | $49 | $1.12M |
| Jan 2026 | 7,724 | $227 | $49 | $1.75M |
| Dec 2025 | 7,085 | $203 | $49 | $1.44M |
| Nov 2025 | 6,678 | $184 | $49 | $1.23M |
| Oct 2025 | 7,567 | $284 | $49 | $2.15M |
| **6-month avg** | **~7,300** | **~$199** | **$49** | **~$1.5M** |

The median ($49) is substantially lower than the average (~$199) because a small number of high-balance accounts (especially rare savings charge-offs averaging ~$1,200) skew the average upward. Most charge-offs are small, non-fraud balances from ACH returns or timing issues.

Source: `TDM_RISK_MGMT_HUB.SUMMARIZED.RR_BANKING_TRANSACTIONS_FACTS` joined with `RR_BANKING_TRANSACTIONS_DETAILS`, filtered on charge-off transaction codes (`DDCHGOFF`, `DDWRTOFF`, `SDCHGOFF`, `SDWRTOFF`).

DDA (checking) charge-offs account for 95%+ of volume. Savings charge-offs are rare (~40/month) but carry higher average balances (~$1,200).

### Fraud vs. non-fraud breakdown

_Queried April 6, 2026, from production Snowflake via `snow sql -c sofi`. Source: `RR_BANKING_TRANSACTIONS_FACTS` joined with `RR_BANKING_TRANSACTIONS_DETAILS`, segmented by transaction code._

Transaction codes distinguish fraud write-offs (`DDFRDWO`, `SDFRDWO`) from non-fraud charge-offs (`DDCHGOFF`, `DDWRTOFF`, `SDCHGOFF`, `SDWRTOFF`).

**All-time:**

| Category | Accounts | Total $ | Avg $ | % of Total |
|---|---:|---:|---:|---:|
| Charge-Off (Non-Fraud) | 197,363 | $62.6M | $317 | 87.8% |
| Write-Off (Non-Fraud) | 27,495 | $1.3M | $48 | 12.2% |
| **Fraud Write-Off** | **5** | **$6,188** | **$1,238** | **0.002%** |

**Last 12 months:**

| Category | Accounts | Total $ | Avg $ | Median $ | % of Total |
|---|---:|---:|---:|---:|---:|
| Charge-Off (Non-Fraud) | 79,417 | $19.6M | $246 | $49 | 89.4% |
| Write-Off (Non-Fraud) | 9,452 | $781K | $83 | $4 | 10.6% |
| Fraud Write-Off | 0 | $0 | - | - | 0% |

**Key takeaway:** Confirmed fraud write-offs are effectively **0%** of all charge-offs. Only 5 accounts all-time out of 224,863 total. Zero in the last 12 months. This means:

1. **The vast majority of charge-offs are from negative balances (ACH returns, timing issues), not fraud.** This supports a straightforward approach.
2. **The median charge-off balance is $49.** Most balances are small, confirming that complex collections infrastructure (payment plans, settlement negotiation, multiple payment rails) is not warranted.
3. **Fraud-closed accounts, if any, are handled through a separate mechanism** (not the charge-off transaction codes). The 5 all-time fraud write-offs averaged $1,238, suggesting these rare cases are identified and handled differently.
4. **Michelle Barnwell's assessment was correct:** "The big ones are going to be true fraud probably and they're not going to come back to us." The data confirms the recoverable population is almost entirely small, non-fraud balances.

This resolves open question R4 (formerly #36).

### Recovery opportunity

| Recovery Rate (industry benchmark: 5 to 15%) | Estimated Monthly Recovery |
|---|---:|
| 5% | ~$73K |
| 10% | ~$146K |
| 15% | ~$219K |


---

## In-Scope

The scope of this PRD will include a minimum set of capabilities that allows us to start contributing charge-off data to EWS without creating regulatory exposure.

| # | Capability | Why it's required | What "done" looks like |
|---|---|---|---|
| 1 | **Balance lookup (in-app)** | Members must be able to see what they owe. | Logged-in: charged-off balance card in Banking tab showing amount owed, closure date, and EWS impact. |
| 2 | **Payment collection: ACH push (primary)** | UDAAP: cannot report charge-offs without a repayment path. Zero return risk to SoFi. Michelle Malavolta's explicit recommendation. | In-app flow displays SoFi routing number + recovery account number. Member initiates transfer from their bank (ACH, wire, or FedNow). Member confirms payment via "Confirm My Payment" form in the app. SoFi reconciles against incoming GL. |
| 3 | ~~**Payment collection: debit card**~~ | *Not in scope.* See Future Considerations. | N/A |
| 4 | **EWS healing report** | Contractual: must update EWS within 10 business days of payment. | Healed account status submitted to EWS via batch file or API within 10 business days. |
| 5 | **Dispute intake (mail + indirect via EWS)** | FCRA: members have a statutory right to dispute furnished data. Direct disputes by mail only (FCRA-compliant). | Member can file dispute by mail. Joe Conlon's team investigates. Indirect disputes received via EWS portal. Investigation completed within 30 days. |
| 6 | **Dispute flags in EWS data** | EWS Operating Rules: disputed records must be flagged while under investigation. For direct disputes (30 calendar day investigation window), SoFi must notify EWS at the start of the investigation (via compliance portal) so the flag appears in the next daily contribution file. If SoFi notifies at the start, EWS suppresses the record on day 30 if no resolution is received; if SoFi notifies upon completion, EWS suppresses/updates immediately. For indirect disputes, EWS opens the case and adds the banner within 5 business days. If no resolution is provided within 30 days of the flag, EWS automatically removes the reporting. | "Consumer Disputes Debt" indicator set in contribution file during investigation. Removed on resolution. |
| 7 | **Pre-closure notifications (via Braze)** | UDAAP: members must be informed of consequences and given a reasonable opportunity to resolve before adverse reporting. **No pre-closure notifications exist today** (confirmed Apr 8, Erica Hester). | Braze-triggered notifications (in-app and/or email) at key thresholds during the 57-day negative balance window. Must include: balance amount, days remaining before closure, EWS consequences, and how to deposit funds. Exact trigger points (e.g., Day 7, Day 30, Day 50) to be defined with Ops and Compliance. |
| 8 | **Updated closure and post-closure communications** | FCRA/UDAAP: members must know about EWS reporting and how to resolve it. | Day 57 closure email updated to include balance amount, EWS consequences, instructions to log into the SoFi app to pay, reference number, and dispute mailing address. Adverse action notices include EWS contact info per FCRA Section 615(a). |
| 9 | **Payment-to-account reconciliation (ACH push)** | Push payments to a general GL require attribution to a specific member. | "Confirm My Payment" confirmation form in-app creates a pending payment record linked to the member. Backend reconciliation matches incoming GL transactions to pending records by amount + date + source bank info. |

### What is NOT included

Debit card payments, agent-assisted payment, partial payments, payment plans, credit card payments, ACH pull (SoFi-originated), self-service dispute forms, proactive outreach, account revival, collections, logged-out guest portal (sofi.com/pay), unique account numbers per member, or AI-assisted reconciliation. See Future Considerations.

### Who is affected

- **Charged-off C&S members:** Members whose SoFi Checking & Savings accounts were closed with a negative balance. Estimated in the thousands to tens of thousands. Typical balances are small (likely under $100), often resulting from ACH returns or timing issues rather than intentional fraud.
- **SoFi Operations:** No process, tooling, or training exists to handle healing payments or EWS data disputes.
- **Compliance and Legal:** Regulatory gap that grows with every charge-off reported to EWS without a corresponding healing/dispute path.


---

## Member Experience: Current State vs. Proposed

### Pre-charge-off (57-day grace period)

| Day | Current Experience | Proposed Experience (MVP) |
|-----|---|---|
| Day 0 | ACH return, failed transaction, or other event causes account to go negative. Balance goes negative in the app. Member may or may not notice. | **Same**, plus in-app banner: "Your account has a negative balance. Resolve it to avoid account closure and reporting to Early Warning Services." |
| Day 1 to 14 | **No notification.** Member is not informed their account is negative or at risk of closure. | **New (via Braze): Push notification + email** with EWS implication: "Your account has a negative balance of -$XX. If not resolved, your account will be closed and reported to Early Warning Services (EWS), which may prevent you from opening bank accounts at other institutions for up to 5 years." |
| Day 15 to 45 | **No notification.** | **New (via Braze): Escalated push notification + email** reinforcing consequences: "You have X days left to bring your balance to $0. After that, your account will be closed, the balance charged off, and reported to EWS." |
| Day 46 to 57 | **No notification.** | **New (via Braze): Final warning push notification + email**: "Your account will be closed in X days. The charged-off balance will be reported to EWS and may block you from opening accounts at other banks for up to 5 years. Deposit $XX now to avoid this." |
| Day 57 | Account is closed and balance is charged off. A closure email is sent, but it does not mention the outstanding balance amount, time the member had to resolve, or EWS consequences. The Banking tab no longer shows the account. No indication of what they owe or how to pay. | Account is closed and balance is charged off. **Updated closure email**: "Your account has been closed with an outstanding balance of $XX. This will be reported to EWS. Log into the SoFi app to pay and clear your record, or call 1-855-456-7634. Your reference number is REF-XXXXXXXXXX." |

> **Key finding (Apr 8, Erica Hester):** There is currently **no process for notifying members prior to account closure.** Members receive no warning that their account is negative, at risk of closure, or about to be reported to EWS. The only email today is triggered at Day 57 (closure), and it does not mention the balance amount or how to resolve it. As part of this initiative, Erica has included a requirement to trigger pre-closure notifications via Braze (in-app messaging or email) when the account reaches key status thresholds before the 57-day closure. This is a prerequisite for UDAAP compliance: members must be informed of consequences and given a reasonable opportunity to act before adverse reporting.

### Post-charge-off (current vs. proposed)

**Logged-in experience (SoFi app):**

| Trigger | Current Experience | Proposed Experience (MVP) |
|---|---|---|
| Member logs into SoFi app | Banking tab shows no active account. No mention of charged-off balance. Confusion. | Banking tab displays a "Charged-Off Balance" card showing amount owed, EWS impact, and a "Deposit" option (ACH push). |
| Member tries to open a new SoFi Money account | Declined with no clear explanation of why or how to fix it. | Declined, but directed to resolve the charged-off balance first. |
| Member wants to pay | No payment channel exists. **Complete dead end.** | ACH push from external bank. Member sees routing/account info in-app, sends funds from their bank, confirms via "Confirm My Payment." 1 to 3 day settlement. Zero return risk. |
| Member wants to dispute data | No dispute intake process. Agent cannot help. | "Think this is wrong? Dispute this data" link routes to mailing address/instructions. |

**Logged-out / cannot access app experience:**

| Trigger | Current Experience | Proposed Experience (MVP) |
|---|---|---|
| Member receives adverse action letter from another bank | Letter names SoFi/EWS as data source. No recourse, no way to pay. | Letter instructs member to log into the SoFi app. Agent assists with app access if needed. |
| Member cannot log in (forgot credentials, no app) | No payment channel. Agent cannot help beyond "try resetting your password." | Agent assists with password reset / app access. If member truly cannot access app, no payment option exists. See Future Considerations for guest portal. |
| Member calls SoFi support | Agent has no tooling, SOP, or payment mechanism. Dead end. | Agent looks up account, directs member to log into the SoFi app. No agent payment processing. |
| Member wants to dispute (no login) | No dispute intake process. No defined address or timeline. | Closure email and agent scripts include dispute mailing address and instructions. |

---

### Proposed experience detail

The healing flow is in-app only (logged-in members) with a single payment rail: ACH push from external bank (member-initiated, zero return risk).

#### Logged-in experience (SoFi app)

For members who can log into their SoFi account. This is the primary flow.

**Entry points:**

| Entry point | Frequency (estimated) | Flow |
|------------|----------------------|------|
| **Adverse action from another bank** | Most common. Member is denied at another FI, sees SoFi named in the adverse action letter. | Member calls SoFi or logs into app. |
| **Member logs into SoFi app** | Less common. Member opens the app and sees the charged-off balance banner. | In-app flow begins. |
| **SoFi proactive outreach (future consideration)** | Not in scope. Push notification or email. | Would say: "You have an outstanding balance. Pay now to clear your EWS record." |

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
        │  [Deposit $XX.XX]                                  │
        │  Get SoFi's account details to send payment       │
        │  from your bank                                   │
        │                                                  │
        │  Think this is wrong? [Dispute this data]        │
        └─────────────────────────────────────────────────┘

Step 3: Member taps "Deposit $XX.XX"
        ┌─────────────────────────────────────────────────┐
        │  Deposit                                         │
        │                                                  │
        │  Send exactly $XX.XX to SoFi using the           │
        │  details below. You can use ACH transfer,        │
        │  wire, or any method your bank supports.         │
        │                                                  │
        │  Routing number:  XXXXXXXXX                      │
        │  Account number:  XXXXXXXXXXXX                   │
        │  Amount:          $XX.XX                         │
        │  Reference:       REF-XXXXXXXXXX                 │
        │                   (include in memo if possible)  │
        │                                                  │
        │  [Copy Details]                                  │
        │                                                  │
        │  After you send the payment from your bank,      │
        │  come back here and confirm:                     │
        │                                                  │
        │  [Confirm My Payment]                          │
        └─────────────────────────────────────────────────┘

        When member taps "Confirm My Payment":
        - Enter: date sent, amount sent, source bank name
        - Optional: source bank routing/account number
        - Optional: confirmation number from member's bank
          (appears on the bank's receipt screen after the transfer
          is submitted, not before; member can copy it before
          switching back to the SoFi app)
        - Creates a "pending payment" record linked to this account
        - On-screen: "Payment pending. We'll confirm once we
          receive the funds (typically 1 to 3 business days)."

Step 4: ACH push confirmation (pending)
        ┌─────────────────────────────────────────────────┐
        │  ⏳ Payment pending                              │
        │                                                  │
        │  We're watching for your payment of $XX.XX.      │
        │  This typically takes 1 to 3 business days.      │
        │                                                  │
        │  Confirmation: EWS-2026-XXXXX                    │
        │                                                  │
        │  What happens next:                              │
        │  1. We'll email you when we receive the funds.   │
        │  2. Your EWS record will be updated within       │
        │     10 business days of receiving payment.       │
        │  3. Once updated, this charge-off will no        │
        │     longer appear when other banks check your    │
        │     record. You will be able to open accounts    │
        │     at other financial institutions again.       │
        │                                                  │
        │  [Done]                                          │
        └─────────────────────────────────────────────────┘

Step 5: Banking tab shows pending state (return visit)
        When member returns to the app, the charged-off balance card
        updates to show pending payment status: amount, date, source
        bank, confirmation number, and "We're watching for your
        payment" message. Tapping "View payment details" shows
        the full Payment Pending screen (Step 4).

Step 6: Email confirmation
        Sent when funds are matched (or after 5 business days
        with a follow-up if not yet matched).

Step 7: Backend processing
        Reconciliation matches incoming GL funds to pending record.
        Account status updated in system of record.
        EWS reporting triggered (next batch cycle, within 10 business days).

Step 8: EWS record updated (async, within 10 business days)
        Member receives follow-up email: "Your EWS record has been updated."
```

#### Members who cannot access the SoFi app

Members who cannot log in (forgot credentials, no app access, or directed by an adverse action letter from another bank) should call support (1-855-456-7634) for account access assistance (password reset, etc.). Payment processing over the phone is not available. See Future Considerations for guest portal and debit card options.

#### Journey 2: Dispute

**How does a member learn they can dispute?**

| Discovery path | What the member sees |
|---|---|
| **In-app charged-off balance card** | "Think this is wrong? Dispute this data" link below the Pay button. Taps through to dispute instructions screen with phone number and mailing address. |
| ~~**Guest portal (sofi.com/pay)**~~ | Not in scope. See Future Considerations. |
| **Charge-off notification email (Day 57)** | Email includes: "If you believe this information is inaccurate, you have the right to dispute it. Write to [custom/dedicated mailbox address TBD]." (Phone number removed; phone intake is NOT FCRA-compliant for data disputes.) |
| **Adverse action letter from another bank** | Letter names EWS. EWS consumer services provides SoFi contact info. Member calls SoFi, agent explains dispute process. |
| **SoFi help center / FAQ** | Help article: "How to dispute data SoFi reported to EWS." Includes phone number, mailing address, and what to include in a dispute. |
| **Agent (phone)** | Any member who calls SoFi about a charge-off is informed of their right to dispute. Agent explains the process and can intake the dispute on the spot. |

**Dispute mailing address must appear in:** (1) in-app dispute screen, (2) charge-off notification email, (3) help center, (4) agent scripts, and (5) adverse action letters.

**Entry points:**

| Entry point | Flow |
|------------|------|
| **Member sends mail (direct dispute)** | Written dispute received at custom/dedicated mailbox (address TBD). Joe Conlon's team processes. This is the ONLY FCRA-compliant channel for direct disputes. |
| **Indirect (EWS portal)** | Member submits dispute to EWS directly. EWS uploads to compliance portal. Joe Conlon's team monitors portal and email, investigates, and responds. |
| **In-app link** | "Think this is wrong? Dispute this data" link on the charged-off balance card routes to mailing address/instructions. Self-service form not in scope (see Future Considerations). |
| ~~**Member calls SoFi**~~ | ~~Agent captures dispute details.~~ **Removed:** Phone intake for data disputes is NOT FCRA-compliant. If a member calls about a dispute, agent should direct them to the mail channel. |
| ~~**Guest portal link**~~ | Not in scope (portal not being built). |

**Dispute flow (mail-based for direct; EWS portal for indirect):**

> **CORRECTION (April 2026 alignment meeting):** Phone intake for data disputes is NOT FCRA-compliant. Direct disputes must be submitted in writing via mail to a custom/dedicated mailbox. Indirect disputes come through the EWS compliance portal, monitored by Joe Conlon's team.

```
Step 1: Member contacts SoFi via mail (direct) or dispute routed via EWS portal (indirect)
        States what they are disputing:
        - Wrong balance amount
        - Wrong address
        - Not closed due to fraud (incorrect closure reason)
        - Account not theirs (identity theft)
        - Charge-off date incorrect
        - Already paid / settled
        - Other data inaccuracy

Step 2: Joe Conlon's Special Operations team receives and logs dispute
        - Direct: received via custom/dedicated mailbox
        - Indirect: received via EWS compliance portal
        - Logs in case management system (TBD)
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

### FCRA Dispute Reference: EWS Process Details

_Source: "FCRA Dispute Process - 2025.pdf" and "EWS Data Contributions Data disputes.pdf" (September 26, 2024). Both shared via #ews-data-contribution Slack channel files._

**Early Warning's FCRA Team manages three dispute types:**

| Dispute Type | How it arrives | SoFi's obligation | Timeline |
|---|---|---|---|
| **Indirect** | Member disputes with EWS directly. EWS notifies SoFi within 5 business days via compliance portal. | Conduct reasonable investigation. Respond via compliance portal or email: data is accurate, needs updating, or should be removed. | 30 calendar days (21 days for Maine) |
| **Direct** | Member sends written dispute directly to SoFi (custom mailbox). | Handle using business processes. If can't resolve immediately, request EWS flag the record (suppresses it). Notify EWS of final outcome. | 30 calendar days |
| **Identity Theft** | Member provides FTC affidavit or police report. | Verify documentation. Block record within 4 business days. | 4 business days for blocking |

**Key rules for SoFi (as the Furnishing Institution):**

1. **Case opened within 5 business days** of receipt. EWS adds a banner to the record during reinvestigation.
2. **Written response with supporting documentation required** (account statements, signatures, etc.) to validate information.
3. **Non-response = record suppressed.** If SoFi does not respond by the due date, EWS suppresses the record from consumer reports.
4. **If SoFi cannot validate the data, it must be removed.** EWS may delete data if SoFi fails to provide sufficient documentation.
5. **30 days after a dispute flag with no instructions, EWS removes the reporting.** This is critical: if Joe's team flags a direct dispute with EWS but then misses the resolution deadline, EWS will automatically delete the data.
6. **All documentation must be legible and submitted with proof of delivery.**

**Direct dispute specifics:**
- If SoFi notifies EWS **at the start** of a direct dispute investigation, EWS suppresses the record on day 30 (if no resolution received).
- If SoFi notifies EWS **upon completion** of a direct dispute investigation, EWS suppresses/updates immediately.
- SoFi should notify EWS at the start (via the compliance portal) to ensure the record is flagged during investigation.

**What EWS shares with consumers:**
- EWS does NOT share specific documentation or statements from SoFi with consumers. Only the final outcome is communicated.
- If a dispute is sustained, the record is updated or removed.
- If not sustained, the consumer is notified of their rights (including the right to add a consumer statement).

**Implications for SoFi's build:**

| Need | Who owns | Details |
|------|---------|---------|
| Compliance portal access and monitoring | Joe Conlon's team | Must monitor EWS compliance portal daily for indirect disputes. Email notifications also come in. |
| Dispute flag automation | EPD / Daniel Evanko | When Joe's team opens a case, the "Consumer Disputes Debt" flag must flow into the next EWS contribution file. |
| 30-day SLA tracking | EPD + Compliance | Automated alerting when disputes approach deadline. Non-response = automatic data suppression by EWS. |
| Investigation response submission | Joe Conlon's team | Submit results via compliance portal or email to EWS. Must include supporting documentation. |
| Record correction flow | EPD / source systems | If dispute upheld, update source data so corrected info flows into next contribution file. |

---

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

## Recommended Solution: Authenticated In-App Pay Page (ACH Push)

### The problem in one sentence

SoFi needs to collect payments from members whose accounts are closed and cannot accept deposits, using a method that (a) SoFi bears minimal liability, (b) scales without agents, and (c) lets SoFi attribute every incoming dollar to the right charged-off account.

### The solution: authenticated in-app payment experience

The payment experience lives in the SoFi app with a single payment rail: ACH push from external bank (member-initiated). The member logs in, sees their charged-off balance in the Banking tab, and follows the bank transfer flow. Because the session is tied to the member's account, **payment context is automatically solved.**

```
                    ┌─────────────────────────────┐
                    │  Member learns they owe      │
                    │  (adverse action letter,     │
                    │   charge-off email, agent,   │
                    │   in-app banner)             │
                    └──────────┬──────────────────┘
                               │
                               ▼
                    ┌─────────────────────────────┐
                    │  SoFi App (logged in)        │
                    │                              │
                    │  Banking tab shows            │
                    │  charged-off balance card     │
                    │  with payment options         │
                    └──────────┬──────────────────┘
                               │
                               ▼
                    ┌─────────────────────────────┐
                    │  Balance displayed            │
                    │  $XX.XX owed                  │
                    │                              │
                    │  [Deposit $XX.XX]             │
                    └──────────┬──────────────────┘
                               │
                               ▼
                    ┌──────────────────────────────┐
                    │  Bank Transfer (ACH Push)     │
                    │                               │
                    │  Page displays:               │
                    │  • SoFi routing number        │
                    │  • Recovery account #          │
                    │  • Exact amount to send        │
                    │  • Reference code to           │
                    │    include in memo             │
                    │                               │
                    │  Member goes to their bank,    │
                    │  sends the funds               │
                    │  (ACH, wire, or FedNow)        │
                    │                               │
                    │  Then returns to page and      │
                    │  clicks [Confirm My Payment] │
                    │                               │
                    │  Enters: date sent, amount,    │
                    │  source bank                   │
                    │                               │
                    │  Creates "pending" record      │
                    │  for reconciliation            │
                    └──────────┬───────────────────┘
                               │
                               ▼
                    ┌──────────────────────────────┐
                    │  "Payment pending"            │
                    │  SoFi matches incoming GL     │
                    │  funds to this record         │
                    │  (auto or manual)             │
                    │  Once matched: EWS healing    │
                    │  starts                       │
                    └──────────────────────────────┘
```

### Why ACH push only?

| Factor | ACH Push from External Bank |
|---|---|
| **Risk to SoFi** | **Very low.** Member-initiated push. No returns possible. No chargebacks. No 60-day unauthorized window. SoFi bears zero liability. Real-world testing confirms: sending bank shows "Transfer cannot be canceled" once submitted. |
| **Cost to SoFi** | ~$0 per transaction. |
| **Member effort** | Medium. Must go to their bank, enter routing/account number, send funds, then come back to confirm. |
| **Payment context** | **Solved.** Member is authenticated in-app before and after the transfer. The "Confirm My Payment" confirmation creates the mapping record. Sending banks have a memo field where the member can include the SoFi reference number (REF-XXXXXXXXXX), further aiding reconciliation. |
| **Speed** | Typically next business day (confirmed via real-world testing). We display "1 to 3 business days" to set conservative expectations. |
| **Endorsement** | Michelle Malavolta's explicit recommendation: member-initiated push, not SoFi-originated. |

**ACH push is the sole payment rail** because it carries the lowest risk to SoFi, costs nothing per transaction, and aligns with the principle that the member (not SoFi) should originate the payment. No returns, no chargebacks, no 60-day unauthorized window. The "Confirm My Payment" confirmation step in the authenticated in-app flow solves the mapping problem.

**Real-world ACH push observations (Apr 8, tested via Wells Fargo to SoFi):**
- Sending bank's review screen shows: from account, to account (SoFi routing/account), amount, send date, delivery speed, expected delivery date, and an optional memo field. No confirmation number at this stage.
- After submission, the bank's receipt screen adds: confirmation number (e.g., `1067397044`) and status (`Pending`). "Transfer cannot be canceled."
- Delivery speed shown as "Next business day" (faster than our conservative "1 to 3 business days" estimate).
- The memo field is where the member can include `REF-XXXXXXXXXX` to aid reconciliation.
- NSF handling varies by bank: Wells Fargo's FAQ states the transfer may still complete via overdraft (fee applies) or Overdraft Protection. Other banks may reject outright. SoFi has no control over this. See Edge Cases.
- Bank FAQ confirms: "This transaction will be initiated immediately but will be completed on the expected delivery date."

See Future Considerations for debit card, guest portal, and other payment rail options that may be revisited.

### How "Confirm My Payment" solves the mapping problem

The core challenge with push payments is attributing incoming GL funds to a specific member. The solution:

1. Member logs into the SoFi app (we know who they are)
2. Member sees their exact balance and SoFi's routing/account info
3. Member goes to their bank and sends the funds
4. Member returns to the app and clicks **"Confirm My Payment"**
5. Member enters: date sent, exact amount, source bank name (optionally routing/account number)
6. This creates a **"pending payment" record** in SoFi's system, linked to the member's charged-off account

SoFi's reconciliation process then matches incoming GL transactions against pending payment records using:
- **Amount** (full balance only, so it's a known number)
- **Approximate date** (member told us when they sent it)
- **Source bank info** (if provided)

For a median $49 payment, this combination should provide high-confidence matching. Edge cases (duplicate amounts on the same day) can be flagged for manual review.

**Fallback if the member doesn't confirm:** SoFi sends a follow-up email after 5 business days: "We haven't received your payment confirmation yet. If you've sent a payment, please log into the SoFi app and confirm." Unmatched funds in the GL can be manually investigated by ops.

**Alternatives considered for payment-to-account mapping:**

| # | Approach | Verdict |
|---|---------|---------|
| 1 | **Generate a unique temporary SoFi account number per member** | Ideal long-term solution (clean, fully automated). Unknown if SoFi can generate disposable account numbers today. Investigate with banking engineering. See Future Considerations. |
| 2 | **Unique tokenized link in charge-off email** | Good for members who have email access. Backend maps the session to their account. Does not solve for members arriving via adverse action letter from another bank. See Future Considerations. |
| 3 | **Member emails proof of payment to a dedicated address** | Simple fallback. Could be AI-automated over time (parse screenshot + reference number, cross-reference GL). Higher ops overhead than the "Confirm My Payment" form. **Deferred; the in-flow confirmation form is better.** |
| 4 | **Member enters source bank routing/account number after payment** | This is what the "Confirm My Payment" form does. **Implemented in the recommended solution.** |
| 5 | **Bill pay (SoFi as bill pay destination)** | Members familiar with bill pay, but other banks may default to sending a check. Needs exploration. See Future Considerations. |
| 6 | **Reopen the closed account** | Not viable. Account is permanently closed; reopening capability not available until late August at the earliest. |

See Future Considerations for additional improvements: unique temporary account numbers per member, guest portal (sofi.com/pay), AI-assisted reconciliation, and more.

### How agents fit in (direction only, no payment processing)

Agents **cannot process payments**. Their role is strictly to direct members to the SoFi app for self-service ACH push payment.

```
Member calls SoFi
       │
       ▼
Agent verifies identity (name + DOB + last 4 SSN)
       │
       ▼
Agent looks up charged-off balance in ops tool
       │
       ▼
Agent: "You have an outstanding balance of $XX.XX.
        The fastest way to pay is through the SoFi app.
        Log in and go to Banking to see your payment options."
       │
       ├── Member can access the app → Agent done. Member self-serves.
       │
       └── Member CANNOT access the app
                │
                ▼
           Agent assists with password reset / app access.
           No payment processing available over the phone.
```

Agents **never** initiate ACH pulls. Agents **never** process payments over the phone. The agent's sole job is to help the member regain app access so they can self-serve.

### What this means for launch volume

When SoFi begins bulk EWS reporting (~7,000 new charge-offs/month, plus a historical backfill), members will start receiving adverse action letters from other banks. Expected inbound:

| Scenario | Inbound volume | Can self-serve handle it? |
|---|---|---|
| 5% of reported accounts attempt to repay | ~350/month | Yes. No agent involvement needed. |
| 10% attempt to repay | ~700/month | Yes. Agents handle overflow only. |
| Initial bulk backfill (one-time) | Unknown, potentially thousands in month 1 | Self-service is the only viable approach. Agent queue would be overwhelmed. |

### Decisions needed for Friday review

| # | Decision | Options | Recommendation | Blocker? |
|---|----------|---------|----------------|----------|
| 1 | **Routing/account number for ACH push** | General recovery account (one for all members) vs. unique account per member | General recovery account (simpler). Unique accounts per member not in scope (see Future Considerations). SoFi's existing routing number is `031101334` (confirmed via real transfer test). Need to confirm which account number to use for the recovery GL. | **Yes** (need to confirm SoFi has a recovery account that can receive external ACH/wire) |
| 2 | ~~**Payment processor for debit card**~~ | TabaPay vs. Stripe | **Not in scope.** Debit card not included. See Future Considerations. | No |
| 3 | ~~**Debit card?**~~ | In scope vs. not | **Not in scope.** ACH push only. Debit card and agent payment processing not included. See Future Considerations. | No |
| 4 | ~~**Guest portal (sofi.com/pay)?**~~ | In scope vs. not | **Not in scope.** In-app self-service only. See Future Considerations. | No |
| 5 | **Email link: pre-authenticated or generic?** | Day 57 email includes a unique tokenized link (one click to see balance) vs. generic URL (member must enter reference #) | Tokenized link preferred (lower friction), but generic URL is acceptable for MVP if tokenized links add security complexity. | No |


---

## Ideal Healing Flow (End-to-End)

_Source: Working doc, 3/12 meeting with Michelle Malavolta and Brandi Morales-Espinal, EWS input. Updated 4/8 per Shahar/Shweta payment method deep dive._

```
1. Day 57 comms update
   Member is notified their account is being closed specifically due to charge-off.
   Comms must mention EWS reporting and include instructions to log into the SoFi
   app or call support for repayment, plus dispute mailing address.

2. Report to EWS
   SoFi reports date of delinquency for accounts placed for collection / charged off.
   Must report within 90 days of initial reporting.

3. Member learns they owe (sometime later)
   Triggered by: adverse action letter from another FI, reviewing old SoFi emails,
   logging into SoFi app and seeing charged-off balance card, or calling SoFi.

4. Member opens SoFi app
   Self-service via app is the only payment path. If member calls,
   agent assists with app access (password reset) but cannot process payments.
   Member logs in with existing credentials.

5. Member sees their charged-off balance
   Amount owed, account closure date, EWS status, and "Deposit" option.

6. Member pays from external bank (ACH push)
   Page displays SoFi routing number, recovery account number, and exact
   amount. Member goes to their bank, sends the funds (ACH, wire, or FedNow),
   then returns to the app and confirms via "Confirm My Payment" form.
   Zero return risk. SoFi bears no liability.
   Check and credit card are NOT acceptable. Full balance only.

7. General ledger gets updated
   1 to 3 business days. "Pending payment" record created at confirmation;
   reconciliation matches incoming GL funds to the record.

8. "Charge off" status is removed from account
   Account stays closed (no need to reopen, confirmed 3/12).

9. Data gets passed back to EWS within 10 business days
   Update the "paid code" to "P" along with the date paid off.
   EWS recommends automation of paid code and paid dates.

10. Member receives receipt / confirmation
    Required per Brandi Morales-Espinal (3/12).
    "Payment pending" on-screen + email when funds are matched.
    Includes: amount, date, confirmation number, and next steps.

11. Member notified when EWS record is cleared (email, async)
    "Your EWS record has been updated. This charge-off will no longer
    affect your ability to open accounts at other financial institutions."
```

**Key decisions confirmed (3/12, Michelle Malavolta; updated 4/8):**

| Decision | Resolution |
|----------|-----------|
| Can we keep the account closed throughout? | **Yes.** No need to reopen the account. |
| Acceptable payment types? | **ACH push** (member-initiated from external bank). **Check is NOT acceptable.** Credit card ruled out (compliance concern). **Update (Apr 8):** Michelle strongly prefers member-initiated push (ACH or wire), not SoFi-originated. |
| How does GL get updated? | GL doesn't track individual charge-offs today (one big balance). "Confirm My Payment" confirmation creates a pending record; reconciliation matches incoming GL funds to the record. |
| Receipt/confirmation required? | **Yes.** Per Brandi Morales-Espinal. |
| Must we allow new account opening after? | **No.** Not a requirement. |
| Agent involvement? | **(Apr 8, Shahar Ronen):** Minimize. Self-service (in-app) is the only payment flow. Agent directs members to the SoFi app and assists with app access. No agent payment processing. Agent never initiates ACH. |
| Push vs. pull? | **(Apr 8, Michelle Malavolta / Shahar Ronen):** Push strongly preferred and the sole rail. ACH push from external bank eliminates return risk (no R10 unauthorized returns, no 60-day window). SoFi bears no liability. ACH pull (SoFi-originated) is explicitly not recommended due to 60-day unauthorized return risk. |
| Payment-to-account mapping? | **(Apr 8):** Solved by the authenticated in-app model. Both rails start from an authenticated session in the SoFi app, so SoFi always knows which member is paying. For ACH push, the "Confirm My Payment" confirmation step creates the mapping record. |

**EWS input on partial vs. full payment:**

> EWS does not want partial payment. They want the original charge-off amount reported, and only update the record when the full amount is paid back. Update the "paid code" to "P" along with the date it was paid off. This is often an issue found in data accuracy analysis. EWS recommends automation of paid code and paid dates to be passed.

_Note: If funds come in from other sources after we've stopped reporting a charged-off account, and the balance is partially recovered, we do NOT update EWS until the full amount is repaid._


---

## Payment Method Options

| Payment Method | Pros | Cons |
|---|---|---|
| **Debit Card via TabaPay** (Collections precedent; contact Tong Han) | Already integrated at SoFi for collections. Near-instant settlement, no bounce risk. Lower per-transaction cost than Stripe for debit (~$0.50 to $1.50 on $50). | May not have a member-facing UI/SDK for in-app use (needs investigation). Agent phone flow requires PCI-compliant handling (agent hears card number). Less familiar to member-facing product eng teams. |
| **Debit Card via Stripe** (SoFi Plus precedent) | Proven member-facing integration (SoFi Plus). GL routing already exists (funds to SoFi, not member account). Drop-in UI components (Stripe Elements) for app and web. PCI compliance handled by Stripe. Supports guest checkout (no SoFi login required on web). | Higher per-transaction cost (~2.9% + $0.30; ~$1.75 on a $50 balance). Redundant if TabaPay already works. May need to restrict credit cards (Stripe accepts both by default). |
| **ACH Pull (Debit) via UNA/Plaid** (Smart Card precedent; contact Anthony Zuffo) | Lowest per-transaction cost (~$0.20 to $0.50). ACH confirmed OK by Michelle Malavolta (3/12). Existing tooling per Anthony Zuffo. | 2 to 3 business day posting time (not "near-instant"). Requires member to link an external bank account via Plaid; closed-account members likely don't have one linked. UNA linking typically requires an active account context; enabling it post-closure adds complexity. ACH returns/reversals create reconciliation risk. |
| **ACH Push from External Bank** (member-initiated transfer to SoFi routing/account number) | No new payment infrastructure needed on SoFi's side. Low/no per-transaction cost. Works regardless of SoFi account status. | Payment context NOT auto-solved: SoFi receives funds but cannot match them to a charged-off account without the member separately calling/emailing. High ops burden for manual reconciliation. High member friction (must know routing + account number, initiate from their bank, then notify SoFi). 1 to 3 business day posting. |
| **Wire Transfer** (member-initiated) | Same-day settlement. Used in lending recovery today (via third-party partner). No new SoFi infra needed. | Wire fees are $15 to $30 for domestic, which can exceed the median charge-off balance ($49). Same reconciliation problem as ACH push (can't auto-match to account). Very high member friction. Only realistic as a manual fallback for large balances. |
| **PayNearMe** (multi-method third party; lending recovery precedent) | Maximum payment flexibility: CC, debit, Venmo, cash, Apple Pay, Google Pay. Already integrated at SoFi for loan recovery. Zero member friction (click a link, pay however you want). No SoFi login required. Unique link per member solves payment context automatically. PayNearMe handles PCI. | Adds vendor dependency and per-transaction fee; on a $40 charge-off the fee could be a significant % of recovery. Need to confirm acceptable methods with Compliance (e.g., is credit card OK?). Third-party data sharing considerations. Overkill for compliance (one rail is sufficient). |
| **Standalone Payment Website (sofi.com/pay)** with case number + Stripe | Works for members who can't log in. Case number maps to specific charged-off account (context solved). No SoFi app required. | Requires building a net-new standalone page, case number generation, and security review. Case number as sole auth is weak; needs additional verification (last 4 SSN, DOB). Medium-to-high implementation effort. Michelle Barnwell: "Do not build a portal unless it's the only option." |
| **FedNow / RTP** (real-time push payment) | Near-instant, irrevocable settlement. Low cost. No bounce risk. | SoFi would need to support receiving FedNow/RTP payments to a recovery GL (not confirmed today). Very few consumers know how to initiate a FedNow payment from their bank. Nascent adoption across the industry. Not a realistic MVP option. |
| ~~Check / Money Order~~ | N/A | Explicitly ruled out (Michelle Malavolta, 3/12). Slow to clear, bounce risk, manual processing, can't verify funds. |
| ~~Credit Card~~ | N/A | Explicitly ruled out (Mili's guidance). Accepting credit cards for charge-off repayment creates compliance concerns (member taking on new debt to pay old debt). Not required. |

**Bottom line (updated Apr 8):** Michelle Malavolta strongly recommends ACH push from external bank as the safest option (no return/dispute risk, SoFi bears no liability). Payment-to-account mapping for push methods is solved by the "Confirm My Payment" confirmation on the authenticated pay page. See [Recommended Solution](#recommended-solution-authenticated-in-app-pay-page-ach-push) for full analysis.

**Current recommendation:**
- **Payment rail:** ACH push from external bank. Safest for SoFi (zero return risk, zero liability). Michelle Malavolta's explicit recommendation. Payment-to-account mapping solved by "Confirm My Payment" confirmation on the authenticated pay page.
- **Not in scope:** Debit card (TabaPay/Stripe), guest portal (sofi.com/pay), agent-assisted payment. See Future Considerations.
- **Not planned:** Wire (cost prohibitive for median $49 balances), FedNow (low adoption), ACH pull (return risk), check (ruled out), credit card (ruled out).

### Monthly charge-off volume (last 6 months)

| Month | Accounts | Avg Balance | Median Balance | Monthly Loss |
|---|---:|---:|---:|---:|
| Mar 2026 | 7,722 | $140 | $49 | $1.08M |
| Feb 2026 | 7,164 | $156 | $49 | $1.12M |
| Jan 2026 | 7,724 | $227 | $49 | $1.75M |
| Dec 2025 | 7,085 | $203 | $49 | $1.44M |
| Nov 2025 | 6,678 | $184 | $49 | $1.23M |
| Oct 2025 | 7,567 | $284 | $49 | $2.15M |
| **6-month avg** | **~7,300** | **~$199** | **$49** | **~$1.5M** |

The median ($49) is substantially lower than the average (~$199) because a small number of high-balance accounts (especially rare savings charge-offs averaging ~$1,200) skew the average upward. Most charge-offs are small, non-fraud balances from ACH returns or timing issues.

Source: `TDM_RISK_MGMT_HUB.SUMMARIZED.RR_BANKING_TRANSACTIONS_FACTS` joined with `RR_BANKING_TRANSACTIONS_DETAILS`, filtered on charge-off transaction codes (`DDCHGOFF`, `DDWRTOFF`, `SDCHGOFF`, `SDWRTOFF`).

DDA (checking) charge-offs account for 95%+ of volume. Savings charge-offs are rare (~40/month) but carry higher average balances (~$1,200).


---

## Scope Summary

| Area | What ships (July 31, 2026) |
|---|---|
| **Payment channels** | ACH push from external bank (member-initiated, self-service in-app, with "Confirm My Payment" confirmation for reconciliation). Single rail, no debit card or agent payment processing. |
| **Payment types** | Full balance only |
| **Dispute channels** | Direct data disputes (wrong address, amount, name etc.) by mail (agent-assisted), custom mailbox to be FCRA compliant. Joe Conlon's team investigates them. Indirect data disputes come from EWS portal (X number of days to investigate). |
| **Dispute handling** | Manual investigation via case management. Notify the member on the dispute outcome. Update to EWS. Update member record as needed if error was found. |
| **Agent involvement** | **Direction only.** Agent directs member to in-app flow and assists with app access. No payment processing by agents. |
| **How members learn they owe** | Charge-off email (with payment link), in-app charged-off balance card, agent, help center, adverse action letter (already happening) |
| **EWS integration** | Batch file or API for healing reports and dispute flags |
| **Logged-out portal (sofi.com/pay)** | **Not in scope.** Members without app access call support for app access assistance. No payment processing over the phone. See Future Considerations. |
| **Analytics** | Basic event tracking (payment volume etc) |
| **Account re-engagement / revival** | Not included. See Future Considerations. |

> **Design principle (Apr 8, Shahar Ronen):** Build a solution that works for 100% of members first, even if the UX is clunky. Self-service over agent involvement. The initial bulk EWS reporting (~500K historical accounts) will drive a wave of adverse action letters and inbound volume. An agent-heavy approach will not scale.


---

## User stories and requirements

Split into two milestones. **Milestone 1 (Healing):** members can see and pay what they owe. **Milestone 2 (Dispute):** members can challenge inaccurate data (required by FCRA before we furnish to EWS). Both can be developed in parallel; both must ship by July 31, 2026.

### Milestone 1: Account Healing / Repayment

| # | Category | Requirement | User Story |
|---|---|---|---|
| 1 | Balance Visibility | Member can view charged-off balance (in-app) | As a member, I want to see how much I owe so I can decide whether to repay. Balance card in Banking tab shows amount, closure date, EWS status, and "Deposit" option. |
| 2 | Balance Visibility | ~~Member can view charged-off balance (guest portal)~~ | **Not in scope.** Guest portal (sofi.com/pay) not included. Members without app access call support. See Future Considerations. |
| 3 | Payment | Member can pay via ACH push from external bank (in-app) -- **primary** | As a member, I want to pay from my bank account. In the SoFi app, I see SoFi's routing number and recovery account number with the exact amount to send. I go to my bank, initiate the transfer (ACH, wire, or FedNow), then return to the app and click "Confirm My Payment" to confirm (entering: date sent, amount, source bank name). This creates a pending payment record for SoFi to reconcile. Zero return risk to SoFi. |
| 4 | Payment | "Confirm My Payment" confirmation (ACH push reconciliation) | As SoFi, when a member confirms they sent a push payment, we create a pending payment record linked to their charged-off account. Backend reconciliation matches incoming GL transactions to pending records by amount + date + source bank info. Unmatched records are flagged for manual review after 5 business days. |
| 5 | Payment | ~~Member can pay full balance via debit card (in-app)~~ | **Not in scope.** See Future Considerations. |
| 6 | Payment | ~~Member can pay full balance via debit card (guest portal)~~ | **Not in scope.** See Future Considerations. |
| 7 | Payment | ~~Agent-assisted payment (phone)~~ | **Not in scope.** No agent payment processing. Agent assists with app access only. See Future Considerations. |
| 8 | Confirmation | Payment confirmation and receipt | As a member, I want a receipt confirming my payment and what happens next. "Payment pending" on-screen + email when funds are matched. Includes amount, date, confirmation number (format: `EWS-YYYY-NNNNN`, e.g., `EWS-2026-04893`), and next steps (EWS record cleared within 10 business days). |
| 9 | EWS Reporting | EWS record updated after payment (backend) | As SoFi, when a member repays, we must update EWS within 10 business days. Healed status submitted via batch file or API. |
| 10 | Notification | Member notified when EWS record is cleared (email) | As a member, I want to know when my EWS record has been cleared. Email sent after EWS confirms update. |
| 11 | Ops Tooling | Agent can look up balance and direct member (internal) | As an ops agent, I want to look up a member's charged-off account and direct them to self-service. Internal admin tool (Salesforce or ops portal) allows search by name, SSN, or reference number. Agent views balance and directs member to log into the SoFi app. Agent assists with app access (password reset) if needed. No payment processing by agents. |
| 12 | Communications | **Pre-closure notifications at key thresholds (Braze)** | As a member with a negative balance, I want to know my account is at risk of closure and EWS reporting so I can deposit funds before it's too late. **No pre-closure notifications exist today** (confirmed Apr 8, Erica Hester). New Braze-triggered notifications (in-app and/or email) at key thresholds during the 57-day window. Each notification must include: (a) current negative balance amount, (b) days remaining before closure, (c) EWS consequences ("may prevent you from opening accounts at other banks for up to 5 years"), and (d) how to deposit funds now. Exact trigger cadence TBD with Ops/Compliance (e.g., Day 7, Day 30, Day 50). |
| 13 | Communications | Charge-off notification includes payment instructions (email) | As SoFi, the Day 57 closure email must tell members how to pay. Updated to include: balance amount, EWS consequences, instructions to log into the SoFi app or call 1-855-456-7634, dispute mailing address, and reference number. Current Day 57 email does not include most of this information. |
| 14 | Communications | Adverse action notices include EWS contact (letter) | As SoFi, adverse action notices must include EWS contact info per FCRA Section 615(a). |
| 15 | Ops Readiness | Agent training and SOPs (internal) | As SoFi, agents must be trained on healing flows before launch. SOPs emphasize: direct member to self-service in the SoFi app; assist with app access; no payment processing by agents; never initiate ACH. |
| 16 | Audit | Audit trail for all healing actions (internal) | As SoFi, all healing actions must be logged for compliance. All actions logged and auditable, including pending payment records and reconciliation outcomes. |

### Milestone 2: Data Dispute

| # | Category | Requirement | User Story |
|---|---|---|---|
| 1 | Dispute Intake | Member can file dispute by phone (phone) | As a member, I want to dispute data SoFi reported to EWS that I believe is wrong. Call 1-855-456-7634. Agent captures disputed data element, reason, supporting docs. Case number provided. |
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


---

## Mocks

_TBD_


---

## Payment Rails

_Updated April 8, 2026, with input from Shahar Ronen and Michelle Malavolta._

| Rail | Ease for member | Risk to SoFi | Cost per transfer | Ease of implementation | Payment context solved? | Agent compatible? | Recommendation |
|---|---|---|---|---|---|---|---|
| **ACH push (member-initiated)** | Medium (must type routing/account number at their bank; multi-step) | **Very low** (SoFi bears no liability; no returns possible; Michelle's preferred option) | ~$0 to SoFi | **Low** (no new SoFi infra for receiving); mapping solved by "Confirm My Payment" confirmation on authenticated pay page | **Yes** (solved via authenticated pay page + confirmation step) | **No** (member must initiate from their bank) | **Sole payment method. Safest for SoFi. Michelle Malavolta's recommendation.** |
| **Debit card (TabaPay)** | Easy (enter card number, familiar to all) | Medium (chargeback up to 90 days, but TabaPay may mitigate; "no bounce risk" per collections) | ~$0.50 to $1.50 on $50 | Medium (already integrated for collections, needs member-facing UI) | **Yes** (SoFi controls the flow) | **Yes** (agent collects card number) | **Not in scope.** See Future Considerations. |
| **Debit card (Stripe)** | Easy (same as above, proven UI components) | Medium (same chargeback risk) | ~$1.75 on $50 (2.9% + $0.30) | Medium (SoFi Plus precedent, drop-in UI) | **Yes** | **Yes** | **Not in scope.** See Future Considerations. |
| ACH pull (SoFi-initiated via Plaid/UNA) | Medium-hard (link account via Plaid for a single payment) | **High** (R10 unauthorized return up to 60 calendar days; member could repay, EWS flag removed, then return the payment) | ~$0.20 to $0.50 | Medium (tooling exists per Anthony Zuffo) | Yes | No | **Not recommended** (return risk unacceptable per Shahar/Michelle) |
| Wire transfer | Hard (same routing/account as ACH push, but wire fees $15 to $45+ at member's bank; SoFi charges $30, maybe $15 for Plus) | Very low (same as ACH push) | $15 to $30 SoFi fee + member's bank fee | Low (same as ACH push) | No (same mapping problem) | No | **Not practical** (median charge-off is $49; wire fee could exceed balance) |
| FedNow / RTP | Medium (if member's bank supports it) | Very low (irrevocable, instant) | Low | Low (same routing/account) | No (same mapping problem) | No | **Not in scope** (most FIs don't support sending via FedNow yet) |
| PayNearMe | Very easy (click a link, pay however) | Low (unique link solves context) | Vendor fee (% of recovery) | Medium (already integrated for lending recovery) | **Yes** (unique link per member) | Yes | Not in scope. See Future Considerations. |
| ~~Check / Money Order~~ | N/A | High | N/A | N/A | N/A | N/A | **Ruled out** (Michelle Malavolta, 3/12) |
| ~~Credit Card~~ | N/A | N/A | N/A | N/A | N/A | N/A | **Ruled out** (Mili's guidance; compliance concern) |

**Key insight from Apr 8 discussion:** The routing/account number SoFi provides is rail-agnostic. A member who receives SoFi's routing and account number can send funds via ACH, wire, or FedNow (whatever their bank supports). SoFi does not need to be prescriptive about the rail. The challenge is purely on the mapping side: how to attribute an incoming payment to a specific member's charged-off balance.


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

### Proposed analytics events

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
| P1 | As an ops agent, I want to look up a member's charged-off account and direct them to self-service | Agent can search by member identifier, view the charged-off balance, and direct member to the SoFi app. Agent assists with app access. No payment processing by agents. |
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
| P0 | As a member, I want to dispute data SoFi reported to EWS that I believe is inaccurate | Member can initiate a direct dispute via mail to a custom/dedicated mailbox (phone is NOT FCRA-compliant for data disputes). Indirect disputes received via EWS portal. Joe Conlon's team processes. Dispute captured with: specific data being disputed, reason, and supporting documentation. |
| P0 | As SoFi, I need to investigate and resolve disputes within FCRA timelines | Investigation completed within 30 days of receipt (45 days if member provides additional info mid-investigation). Member notified of outcome. |
| P0 | As SoFi, if a dispute is frivolous or irrelevant, I need to notify the member within 5 business days | Frivolous dispute determination made and member notified with reasons and list of missing information within 5 business days. |
| P0 | As SoFi, when a dispute is received, I need to flag it in the EWS contribution data | Account Abuse and/or Fraud file updated with "Consumer Disputes Debt" indicator (Y) and consumer's statement. Flag removed when dispute is resolved. |
| P0 | As SoFi, when a dispute is upheld, I need to correct the data with EWS and all CRAs that received it | Corrected records submitted to EWS and all other CRAs within required timeframe. |
| P1 | As a member, if my dispute is denied, I want to understand why and know my right to add a statement | Member receives explanation of denial and is informed of their FCRA right to add a consumer statement to the record. |
| P1 | As SoFi, I need to maintain an audit trail of all disputes and resolutions | All dispute activity (intake, investigation steps, outcome, notifications) is logged and auditable. |
| P0 | As SoFi, when we receive an indirect dispute via EWS/CRA notice, I need to investigate and respond | Joe Conlon's team monitors EWS compliance portal, investigates, reviews CRA-provided info, responds with results, propagates corrections to all nationwide CRAs. Modify/delete/block disputed data if unverified or inaccurate. If error found, update source systems so corrected data flows into EWS files. |

---

## 4. Scope

### In scope (targeting July 31, 2026)

- **Payment collection for charged-off C&S account balances** (full balance payment only; no partial payments or payment plans)
- **Payment method:** ACH push from external bank (member-initiated, zero return risk, Michelle Malavolta's recommendation). Self-service in-app only.
- **Recovery GL:** Funds deposited into a SoFi recovery/collections general ledger (original account is closed and cannot receive deposits)
- **Account status update:** Source system updated after payment, which feeds into Daniel Evanko's EWS file generation pipeline
- **EWS healing report:** Update Account Abuse data within 10 business days of confirmed payment, via the EWS file generation and transmission pipeline (Workstream 1)
- **Agent role:** Direction only. Agent directs member to the SoFi app and assists with app access. No payment processing by agents.
- **Dispute intake (direct):** Mail only to a custom/dedicated mailbox. Phone intake is NOT FCRA-compliant for data disputes.
- **Dispute intake (indirect):** EWS compliance portal, monitored by Joe Conlon's Special Operations team
- **Dispute investigation:** Joe Conlon / Mitchell Morris Jr.'s team investigates within FCRA timelines. EPD builds tooling as needed.
- **Dispute reporting:** EWS contribution records updated with dispute flags, consumer statements, and new data elements (Daniel Evanko to be engaged on fields)
- **Member communications:** Receipt/confirmation for healing payments; new comms for updated account status; dispute outcome notifications
- **Adverse action notices:** Include EWS contact info per FCRA Section 615(a) requirements

### Out of scope

| Item | Reason |
|------|--------|
| Reopening charged-off accounts | Charged-off accounts are closed in the current core. New core could theoretically support revival, but prerequisites unavailable until late August 2026 at earliest. Not a dependency for MVP. |
| Proactive collections or outbound outreach | Amounts are small (typically < $100). Collections agency costs would exceed recovery. Reputational risk outweighs financial benefit. |
| Third-party debt collection workflows | Same as above. Not justified for the balance sizes involved. |
| Loan or credit card charge-offs | Different products, different reporting mechanisms (credit bureaus, not EWS). Lending has its own recovery processes. |
| Settlement or partial payment negotiation | Full balance only. Median charge-off is $49; settlement negotiation is not warranted. |
| Self-service dispute submission (web/app form) | Mail-only for direct disputes; EWS portal for indirect. At ~35 to 50 disputes/month, self-serve form is not justified. |
| Multiple payment channels | EWS requires only one payment methodology. Expanding to additional channels is a SoFi business decision, not a compliance requirement. |
| Phone-based dispute intake | NOT FCRA-compliant for data disputes. Direct disputes must be submitted in writing (mail only). |
| New core banking account revival | David Finske confirmed this is technically possible in the new core, but prerequisites unavailable before late August 2026. Do not take a dependency. |
| Changes to initial charge-off reporting pipeline | Being handled separately by Michelle Barnwell + Data Engineering (Daniel Evanko). |
| Hot File contribution | Explicitly out of scope per the BRD. |
| Logged-out payment portal (sofi.com/pay) | **Not in scope.** In-app only. See Future Considerations. |

See **Future Considerations** at the end of this document for items that are explicitly not planned but may be revisited based on volume, business need, or regulatory changes.

---

## 5. Edge Cases & Error States

| Scenario | Expected Behavior |
|----------|-----------------|
| **Member account closed for fraud** (not just negative balance) | Fraud-closed members may not be able to log in. Agent-assisted path required. Identity verification must be heightened. Do not contribute abuse/fraud records until investigation is complete and confirmed (per BRD). |
| **Member has multiple charged-off accounts** | Display all charged-off balances. Allow member to pay each separately. Each triggers its own EWS healing update. |
| **Member overpays** | System should prevent overpayment by pre-populating the exact balance. If overpayment occurs, initiate refund process. |
| **Payment fails or times out** | Display clear error message. Do not update EWS. Allow retry. Log the attempt for ops visibility. |
| **Member no longer has SoFi app access** (forgot credentials, deleted account, etc.) | Agent assists with password reset / app access. If member truly cannot regain app access, no payment option exists. See Future Considerations for guest portal. |
| **Very old charge-off (1+ years)** | Still eligible for healing. Balance sourced from system of record. No expiration on healing eligibility. Note: EWS flags can persist for 5 years. |
| **Very small balance (< $5)** | Still allow payment. Open question: is there a dollar threshold below which SoFi should write off rather than collect? Finance to advise. |
| **Member disputes during active healing payment** | Pause healing flow until dispute is resolved. Do not double-process. |
| **Member has insufficient funds at their bank** | Behavior varies by bank. Some banks (e.g., Wells Fargo) will complete the transfer by overdrafting the member's account (overdraft fee applies) or using linked Overdraft Protection. Other banks may reject the transfer entirely. If rejected, the transfer never reaches SoFi but the member may believe they paid. SoFi's "Confirm My Payment" record will remain unmatched. After 5 business days with no matching GL entry, send follow-up email: "We haven't received your payment yet. Please check with your bank to confirm the transfer was completed." SoFi has no control over this; it is between the member and their sending bank. |
| **Member's bank transfer is delayed beyond 3 business days** | Some banks take longer. Do not auto-cancel the pending record. After 5 business days, send follow-up email prompting the member to check with their bank. After 10 business days, flag for manual ops review. |
| **Concurrent dispute and healing** | If a member is both disputing data and wanting to pay, the dispute takes priority. Payment can proceed after dispute resolution if balance is confirmed accurate. |
| **Identity theft claim on the charged-off account** | Route to fraud/identity theft handling per FCRA. Cease furnishing the disputed item to EWS until investigation confirms accuracy. Do not refurnish information alleged to be the result of identity theft unless later confirmed correct. |
| **EWS reporting system is down** | Queue the update. Retry within the 10-business-day window. Alert ops if approaching SLA deadline. |
| **Balance discrepancy between system of record and what member expects** | Provide member with detailed breakdown. If member disagrees, route to dispute flow. |
| **Platform differences (mobile vs. web)** | Mobile-only (SoFi app). No web or agent-assisted fallback. Confirm with Design. |

---

## 6. Dependencies & Risks

| Dependency / Risk | Owner | Status | Mitigation |
|------------------|-------|--------|------------|
| **System of record for charged-off balances** - Where do charged-off account balances live today? Core banking (Galileo)? Manual ledger? | Banking Eng / Finance | Open - Discovery needed | Phase 0 discovery item. Cannot build lookup or payment flow without this. |
| **EWS integration specs for healing reports** - API vs. batch file, format, certification process | Michelle Barnwell / EWS | Open - Requested | Start integration discovery immediately. May need EWS certification before going live. |
| **Recovery GL setup** - Finance must define GL codes and structure for recovery payments | Finance | Open | Engage Finance in Week 1. SoFi Plus and Smart Card may have precedent GL structures. |
| **Engineering partner assignment** - No engineering team assigned yet | Shahar Ronen / Dheeraj Mehta / Arjun Hegde | Open | Shahar initiating conversation with Dheeraj and Arjun. Likely MACE for member experience, Ragini's team for ops component. |
| **Compliance approval for updated member communications** - Adding EWS reporting language to charge-off notices | Compliance (Tatyana Pacheco, Kristen Krikorian) | Open | Engage compliance early; may require legal review of disclosure language. |
| ~~**Debit card payment integration**~~ | N/A | Not in scope | Not in scope. See Future Considerations. |
| **New core banking not ready** - Account revival prerequisites unavailable until late August 2026 | Banking Eng (David Finske) | Confirmed - Not available | Do not take a dependency. Build GL-based approach instead. |
| **FCRA dispute timeline compliance** - Must resolve within 30 days or face regulatory action | Ops / Compliance | Open | Automated SLA tracking and escalation alerts. Ops training needed. |
| **July 31 deadline pressure** - MVP in production by July 31, 2026 | Product (Shweta) | At risk - Eng not assigned | Negotiate scope if needed: fully ops-assisted first. Some room to negotiate timeline with business justification. |
| **Dispute volume unknown** - Don't know how many disputes to expect | Product / EWS | Open | Ask EWS for typical dispute rates for charged-off deposit accounts. This will determine if we need self-serve dispute tooling for MVP or if phone/mail is sufficient. |
| **Fraud-closed accounts** - Members closed for fraud may need different handling than negative-balance closures | Fraud Strategy (Dan Rosner, Arpit Ajmani) | Open | Confirm whether fraud-closed members can still log in. May need separate identity verification flow. |
| **EWS can terminate services** - If full contribution compliance not met by December 31, 2026 | EWS (external) | Active risk | July 31 launch is the healing/dispute component. Full contribution compliance (all file types, governance, attestation) coordinated with broader EWS Data Contributions initiative by December 31. |

---

## 7. Cross-Functional Requirements

| Function | Requirements | Status |
|----------|-------------|--------|
| **Legal / Compliance** | FCRA dispute handling (30-day resolution, 5-day frivolous determination, consumer statement rights). UDAAP compliance (must offer healing if reporting charge-offs). Adverse action notices must include EWS contact info. Review updated charge-off communications for EWS language. Confirm dispute intake channels (mail required by FCRA; phone optional). Specified mail address for disputes: SoFi Bank, N.A., 2750 East Cottonwood Parkway #300, Cottonwood Heights, Utah 84121 (confirm with Legal). Confirm data retention and audit trail requirements. | Open - Engage Tatyana Pacheco, Kristen Krikorian, Brandi Morales-Espinal, Derek Fisher |
| **Analytics** | Track: payment volume, healing rate, dispute volume and resolution time, recovery amount ($), ops handle time, member funnel (balance viewed > payment initiated > payment completed). EWS reporting SLA adherence. | Not started |
| **Operations / Support** | Agent training for directing members to in-app self-service and assisting with app access (no agent payment processing). SOPs for: directing members to mail channel for disputes (phone intake NOT FCRA-compliant), handling identity theft claims, escalation paths. MET Operations involvement: Nikita Shah, Erica Hester. **Dispute operations: Joe Conlon and Mitchell Morris Jr. (Special Operations)** handle investigation and responses. Update existing SOPs per BRD requirement. | Open - Engage Joe Conlon, Mitchell Morris Jr., and other Ops stakeholders |
| **Design** | In-app healing experience: charged-off balance display in Banking tab, payment flow, confirmation screen. Empathetic copy (members are trying to do the right thing). Clear explanation of what EWS is and what "healing" means. | Not started |
| **QA** | Test plan covering: payment processing, GL ledgering, EWS reporting trigger, dispute flag updates, FCRA timeline enforcement, edge cases (overpayment, timeout, multiple accounts). | Not started |
| **Finance** | Recovery GL design: GL codes, account structure, reconciliation process. Write-off threshold guidance. | Open - Engage Finance |
| **Fraud / Risk** | Identity verification for agent-assisted payments. Fraud-closed account handling. Bad actor mitigation (e.g., someone paying off a charged-off account to gain access to identity/settings). Review dispute flow for fraudulent dispute prevention. | Open - Engage Dan Rosner, Arpit Ajmani |
| **Privacy** | Data sharing review: member balance and transaction data shared with EWS. Confirm existing disclosures cover this data sharing or identify gaps. PIA (Privacy Impact Assessment) if new data elements are shared. | Open - Engage Sherrie Osborne (Privacy) |
| **Cyber Security** | Payment flow security review. Data transmission security for EWS reporting. | Open - Engage Sandeep Jayashankar |

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

**Recommendation (updated Apr 8):** **ACH push from external bank is the sole payment rail.** Per Michelle Malavolta (3/12 and Apr 8 Slack thread): electronic, member-initiated push (ACH or wire), not SoFi-originated. Zero return risk. Check is NOT acceptable. The authenticated in-app model solves payment-to-account mapping. See the Recommended Solution section.

> **Note:** Many sections of this PRD reference "Stripe" as the payment processor based on the SoFi Plus pattern. After the 3/12 meeting, **TabaPay is the more likely debit card path** since it's already in use for collections. Stripe references in the appendices remain as-is for reference, but the actual implementation should evaluate TabaPay first.

> **STATUS UPDATE (April 2026, Michael Bourgeois sync):**
>
> **Update (3/12):** Michelle Malavolta confirmed the account stays closed (no need to reopen), debit card via TabaPay is OK, ACH is OK, check is NOT OK. The remaining blocker is GL: GL doesn't track individual charge-offs today (one big balance), so enhancements are needed to link payment to the specific charged-off account. Before committing to a payment method:
>
> 1. ~~**Consult Michelle Malavolta**~~ **Done (3/12).** Account stays closed. GL doesn't track individual charge-offs (one big balance). Enhancements needed to link payment to specific account.
> 2. ~~**Verify Stripe integration exists.**~~ **Superseded.** Collections already uses **TabaPay** for debit card. Talk to **Tong Han**. ACH tooling per **Anthony Zuffo**.
> 3. ~~**ACH may not make sense.**~~ **Updated (3/12).** Michelle confirmed ACH is OK. Talk to Anthony Zuffo about existing tooling and GL routing.
> 4. ~~**Account reopening as clunky MVP fallback.**~~ **No longer needed.** Michelle confirmed account stays closed throughout the healing process (3/12).
>
> **Next step:** Schedule meeting with Michelle Malavolta, Serena, and Shweta ASAP to resolve this. This is the #1 blocker before the experience can be designed.

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
| TabaPay (or Stripe) | Not in scope. See Future Considerations. TabaPay already in use by collections (per 3/12 meeting). | Tong Han / Payments Eng |
| General Ledger | Recovery GL for received funds | Finance |
| EWS API / Batch File | Submit healed records, dispute flags, corrections | TBD (same channel as contribution pipeline) |
| Case Management (Salesforce?) | Dispute tracking, investigation workflow | Ops Eng |
| SoFi App (iOS / Android) | Member-facing healing and payment flow | MACE / Product Eng |
| Member Communications | Email confirmations, dispute notices, adverse action letters | Growth / Comms Eng |

### Security and authentication

- **Logged-in:** Payment flow is behind SoFi app authentication
- **Pay by phone:** Not available. Agent assists with app access only.

---

## 9. Timeline & Milestones

**Hard deadline:** December 31, 2026, for full EWS contribution compliance (per contract waiver).
**Target launch:** July 31, 2026, for payment collection and dispute handling in production.

> **Timeline flexibility note (April 2026, Michael Bourgeois sync):** Michael indicated there may be room to negotiate the timeline with a business justification. The $100K/month excess fee could potentially be negotiated to ~$99K/month (or similar) if we need a couple extra months to finish specific components, so that SoFi captures most of the savings while completing the remaining work. This should be discussed in the alignment meeting with Michelle Barnwell and Shahar to understand the exact penalty pathway and drop-dead dates.

| Milestone | Target Date | Notes |
|-----------|------------|-------|
| **Phase 0: Discovery complete** | Week of April 14, 2026 | Confirm system of record, EWS integration specs, GL structure, eng partner assignment. Review SoFi Plus and Smart Card payment patterns. |
| **PRD finalized and approved** | April 21, 2026 | Incorporate discovery findings. Stakeholder alignment. |
| **Design complete** | May 5, 2026 | In-app healing experience (ACH push flow). |
| **Eng kickoff** | May 12, 2026 | Assumption: engineering partner assigned by this date. Confirm or correct. |
| **Payment collection integration (ACH push + GL)** | June 16, 2026 | Core payment flow functional in staging. |
| **EWS healing report integration** | June 30, 2026 | Healing updates can be submitted to EWS. |
| **Ops training and SOP delivery** | July 7, 2026 | Agents trained on directing members to in-app self-service, assisting with app access, and dispute intake. No agent payment processing. |
| **QA complete** | July 21, 2026 | Full test cycle including payment, GL, EWS reporting, dispute flags. |
| **Launch** | July 31, 2026 | Healing payment flow + dispute intake live. Target date, with some room to negotiate if justified. |

---

## 10. Open Questions

### Blockers

| # | Question | Owner | Source |
|---|----------|-------|--------|
| 1 | **CRITICAL: How do we link payment to a specific charged-off account in the GL?** Michelle Malavolta confirmed (3/12) that GL doesn't track individual charge-offs today; it's one big balance. Enhancements needed. Account stays closed (no need to reopen). Debit (TabaPay) and ACH are both acceptable, but we need to figure out how to attribute the payment to the correct member's charged-off account for reconciliation and EWS reporting. **Update (Apr 8):** Michelle confirmed GL #51615 is used for non-fraud charge-offs (per Niki). However, the biggest segment of accounts are charged off automatically after 57 days negative, and it's not confirmed whether GL #51615 is also used for those. **Follow-up needed: confirm whether automatic 57-day charge-offs use the same GL #51615.** | Michelle Malavolta / Serena / Finance | Michael Bourgeois sync, 3/12 meeting, Apr 8 Slack |
| 2 | **What new data elements/fields are needed in the EWS contribution files for disputes?** When a data dispute is received, certain data elements must be fed into the files sent to EWS. Daniel Evanko needs to be engaged to understand where to hold these fields and how to pick them up. Michelle to send data elements documentation. | Daniel Evanko / Michelle Barnwell / Shweta | April 2026 alignment meeting |

### Payment / Healing

| # | Question | Owner | Source |
|---|----------|-------|--------|
| 3 | **Which single payment rail?** EWS requires only ONE rail. Michelle Malavolta confirmed (3/12): **check is NOT acceptable. Debit card OK (collections already uses TabaPay, talk to Tong Han). ACH OK (talk to Anthony Zuffo).** Next step: evaluate TabaPay (debit) vs. ACH for speed, cost, bounce risk, and build effort. TabaPay is already in use at SoFi for collections, which may make it the fastest path. | Tong Han (TabaPay) / Anthony Zuffo (ACH) / Shweta | 3/12 meeting with Michelle Malavolta |
| 4 | Where does the source of truth for charged-off account data and balances live today? Core banking (Galileo)? Manual ledger? | Banking Eng / Finance | Pager |
| 5 | How should Finance structure the recovery GL? Michelle Malavolta confirmed (3/12) that GL doesn't track individual charge-offs today (one big balance). What enhancements are needed to link payments to specific accounts? GL codes? Precedent from SoFi Plus or Smart Card? | Finance / Michelle Malavolta | Meeting, Pager, 3/12 meeting |
| 6 | What is the EWS integration mechanism for reporting a healed account? API, batch file? Can we extend the existing contribution pipeline? | Michelle Barnwell / EWS | Pager, BRD |
| 7 | What is the EWS SLA for processing a healed record and confirming removal of the flag? | EWS (external) | Pager |
| 8 | **ACH specifics:** Michelle confirmed ACH is OK (3/12). Talk to Anthony Zuffo about what tooling exists today. Where would ACH funds land on the GL given account is closed? | Anthony Zuffo / Michelle Malavolta | 3/12 meeting |
| 9 | **TabaPay vs. Stripe?** Collections already uses TabaPay for debit card payments (per Michelle, 3/12). This may supersede the Stripe assumption. Talk to Tong Han. Michael Bourgeois: "Don't stand up a new integration if one doesn't already exist. What can we reuse?" TabaPay appears to be what already exists. | Tong Han / Payments Eng / Shweta | 3/12 meeting, Michael Bourgeois sync |
| 10 | Should we have any payment channels explicitly disallowed for compliance, fraud, or operational reasons? (Note: check is already disallowed per 3/12.) | Compliance / Fraud / Michelle Barnwell | April 2026 alignment meeting |
| 11 | Is there a dollar threshold below which SoFi should write off rather than collect? | Finance | Pager |
| 12 | What happens if a member overpays or the recorded balance is wrong? | Finance / Ops | Pager |

### Dispute

| # | Question | Owner | Source |
|---|----------|-------|--------|
| 14 | **What is the dedicated mailing address for the custom dispute mailbox?** Direct data disputes must come via mail only (FCRA). Needs a custom/dedicated mailbox. Who provisions this? What is the physical address? | Ops / Compliance / Joe Conlon | April 2026 alignment meeting |
| 15 | **What will the end-to-end dispute handling process look like?** From mail receipt to investigation to resolution to EWS update. What does Joe's team do manually vs. what does EPD need to build? What tooling exists today? Can we leverage an existing dispute framework? | EPD / Joe Conlon / Mitchell Morris Jr. | April 2026 alignment meeting |
| 16 | What case management system will Joe's team use for disputes? Salesforce? Internal tool? | Ops Eng / Joe Conlon | Pager |
| 17 | Is there an existing FCRA dispute process at SoFi for credit products that we can extend to deposits? | Compliance / Lending Ops | Pager, BRD |
| 18 | What are the compliance controls needed for deadline adherence? (30-day dispute resolution, 10-day healing update, 5-day frivolous determination.) Tech-side vs. operational? | Compliance / EPD / Joe Conlon | April 2026 alignment meeting |

### Cross-Cutting

| # | Question | Owner | Source |
|---|----------|-------|--------|
| 19 | Can we update charge-off member communications to mention EWS reporting? What disclosures need to change? | Compliance (Tatyana Pacheco) | Meeting, BRD |
| 20 | Can members closed for fraud still log in to the SoFi app? Do they need a different flow? (Note: fraud write-offs are effectively 0% of charge-offs, so this may be very low priority.) | MACE Eng / Fraud | Slack thread (Mili) |
| 21 | Which engineering team owns this: MACE (member experience) or Money Movement? | Dheeraj Mehta / Arjun Hegde | Slack thread |
| 22 | Are existing member disclosures sufficient for EWS data sharing, or do we need new/updated disclosures? | Legal / Privacy (Sherrie Osborne) | BRD |
| 23 | How will members contact SoFi to initiate healing? What information is on the adverse action letter they receive from other FIs? | Product / Compliance / EWS | Working doc, Meeting |
| 24 | Explicit Compliance sign-off on acceptable payment methods? | Compliance | Working doc |
| 25 | **What % of repayment does EWS see across participating FIs?** This will directly impact MVP solution design (e.g., 5 repayments/month = fully ops-assisted; 200+/month = needs self-serve). Ask EWS or Michelle Barnwell. **Research so far:** No publicly available EWS-specific data exists; this is proprietary consortium data. Industry benchmark for deposit charge-off recovery is 5 to 15%. SoFi's profile (median $49 balance, mostly ACH timing issues) suggests initial rates may be on the lower end until members discover the healing path. Shahar's framing is key: volume dictates whether we need self-serve or ops-only. **Action: Michelle Barnwell to ask EWS for repayment rates across their FI network.** | Michelle Barnwell / EWS | Shahar Ronen (Apr 8, 2026) |
| 26 | **What % of reported charge-offs are disputed across participating FIs?** Current EWS estimate is ~35 to 50/month for SoFi. **Research so far:** Against SoFi's ~220K all-time reported accounts, EWS's estimate of 35 to 50/month implies ~0.2 to 0.3% annually, which is very low. For comparison, the FTC's credit report accuracy study found 20% of consumers have an error on at least one credit report, but only 5% have material errors. Deposit account data at specialty CRAs is simpler (fewer data elements, clearer-cut charge-offs), so dispute rates are expected to be well below credit report rates. CFPB 2024 data shows "other personal consumer reports" complaints (including specialty CRAs like EWS) increased 124% YoY, suggesting dispute awareness is growing. **Action: Michelle Barnwell to ask EWS for dispute rates across their FI network to validate the 35 to 50/month estimate.** | Michelle Barnwell / EWS | Shahar Ronen (Apr 8, 2026) |

### Resolved

| # | Question | Resolution | Source |
|---|----------|------------|--------|
| R1 | Partial payments or full only? | **Full balance only.** Not an EWS requirement to expand. | April 2026 alignment meeting |
| R2 | Who owns dispute operations? | **Joe Conlon + Mitchell Morris Jr. (Special Ops).** NOT Tammy Eisel, NOT Srikant Movva. | April 2026 alignment meeting |
| R3 | Dispute volume estimate? | **~35 to 50/month** (EWS estimate). No data on healing volume. | April 2026 alignment meeting + EWS call |
| R4 | What % of charge-offs are confirmed fraud? | **Effectively 0%.** Only 5 fraud write-offs all-time out of 224,863 total. Zero in the last 12 months. Virtually all charge-offs are small-balance, non-fraud events (median $49). | Snowflake query, April 6, 2026 |
| R5 | Average charge-off balance? | **Avg $199, median $49.** DDA checking accounts are 95%+ of volume. Savings are rare (~40/month) but higher avg ($1,200). | Snowflake query, April 2, 2026 |
| R6 | Can we keep account closed during healing? | **Yes.** No need to reopen. Per Michelle Malavolta, 3/12. | 3/12 meeting |
| R7 | Acceptable payment types? | **Debit card (TabaPay) and ACH.** Check is NOT acceptable. TabaPay already used by collections (contact Tong Han). ACH per Anthony Zuffo. | 3/12 meeting with Michelle Malavolta |
| R8 | Is receipt/confirmation required? | **Yes.** Per Brandi Morales-Espinal, 3/12. | 3/12 meeting |
| R9 | Must we allow new account opening after healing? | **No.** Not a requirement. | 3/12 meeting |
| R10 | Does EWS accept partial payment updates? | **No.** EWS wants full amount only. Update paid code to "P" with date when fully paid. Do not send partial updates. | EWS input |

---

## Appendix A: Experience Options Analysis

The MVP experience exists on a spectrum. The right answer depends on build capacity before July 31.

### Option 1: Fully Ops-Assisted (minimum EPD build)

**How it works:** *(Not in MVP. Documented for reference.)* Member calls SoFi support. Agent looks up charged-off balance, collects debit card information over the phone, processes payment manually, applies to GL, triggers EWS update.

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
| Settlement/partial payment | Yes. Recovery unit negotiates settlements (e.g., 60% of balance). Installment plans available. | Late fees proposed ($25 first, $41 subsequent, not confirmed live). Collateral sweep planned. | Full balance only. Median charge-off is $49; settlement is not warranted. |

**Additional context from Michael Bourgeois (April 2026):** All collections is handled by Bill High's team. Shahar had a conversation with the relevant people there to understand if there was anything we could leverage. Key difference: in collections, the account stays open. In credit card, the account stays open 0 to 6 months, then charged off. After charge-off, credit card debt is sold for collection for 6 months. For Money, the account closes at 57 days and the debt is not sold. Michael noted the process is "pretty analogous" to credit card collections after charge-off (both involve trying to collect on a closed account), but confirmed SoFi does not plan to sell Money charge-off debt.

### Key implications for EWS healing

1. **We cannot simply copy the lending model.** Lending accounts stay open and support payments after charge-off. C&S accounts do not. We need a fundamentally different approach (GL-based payment collection).
2. **PayNearMe is already at SoFi.** The lending team uses it for recovery. This could be a future option if SoFi decides to offer broader payment methods, but is not required for EWS compliance.
3. **Credit card's progressive delinquency states** (delinquent > frozen > closed-but-ledger-active) are not available for C&S accounts. There is no "closed but ledger-active" state for bank accounts today. The new core migration could potentially enable this, but not before late August 2026.
4. **Settlement offers are common in lending** (settle for 60%) **but not planned for Money.** The balance amounts are too small to justify the complexity (median $49). Full payment only.
5. **The average charge-off balance for Money is $199, with a median of $49.** Most balances are small (DDA accounts are 95%+ of volume). This means per-transaction cost matters for payment rail selection, and complex collections infrastructure is not justified.

---

## Appendix C: Reuse vs. Build Assessment

| Capability | Existing Asset | Reuse Potential | Notes |
|-----------|---------------|----------------|-------|
| Debit card payment collection | SoFi Plus (Stripe integration) | **High** | Same pattern: collect payment from external card, apply to GL. Need to restrict to debit only. See Appendix F for full SoFi Plus analysis. |
| Multi-method payment collection | PayNearMe (lending recovery) | **Medium** | Already used at SoFi for loan charge-off recovery. Supports CC, debit, Venmo, cash. Not needed for compliance (one rail sufficient). See Appendix G for full comparison. |
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
| **Joe Conlon** (Special Operations) | Dispute operations owner. His team handles direct and indirect dispute investigations and responses. Customer/user of whatever EPD builds. (Corrected April 2026: NOT Tammy Eisel, NOT Srikant Movva.) |
| **Mitchell Morris Jr.** (Special Operations) | Dispute operations. Works with Joe Conlon on dispute handling. |
| **Tami Wilson** (Operations) | Dispute process design and operational readiness |
| **Curtis (last name TBD)** (File Transmission) | EWS file transmission (Andy on leave). Ensures files get delivered to EWS. |
| **Sophia Carlton, Dana Wolfe, Diana Barker, Irene Morley** (2nd LOD) | Fraud/Ops/Tech oversight |
| **Krista Thorsen** (Operations Risk) | Operational risk assessment |
| **Sandeep Jayashankar** (Cyber Security) | Payment flow security, data transmission security |
| **Sherrie Osborne** (Privacy) | Data sharing review, PIA assessment |
| **Finance** (TBD) | Recovery GL design, GL codes, tax implications |
| **Design** (TBD) | Member-facing UX for healing and dispute flows |
| **EWS** (external) | API/integration specs, SLAs, testing/certification, dispute rate data |
| **Tong Han** (Payments Eng, Collections) | TabaPay integration for debit card payments. Collections team already uses TabaPay. Contact for debit card rail feasibility. (Per Michelle Malavolta, 3/12.) |
| **Anthony Zuffo** (Payments Eng, ACH) | ACH payment processing. Contact for ACH rail feasibility and existing tooling. (Per Michelle Malavolta, 3/12.) |

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
| Payment-due reminder | 3 days before due date | Could adapt for proactive healing outreach if ever needed (not in scope) |
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
| Partially Paid | Shows remaining amount to cure | Not applicable (full payment only) |
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
| Partial payment tracking | **None** | Not applicable (full payment only). |

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
| **Debit/credit card payment rail (Stripe)** | **High** | Core payment infrastructure is directly reusable. Restrict to debit only for EWS healing. Not in current scope (ACH push is the sole rail), but strongest reuse candidate if debit card is added in the future. |
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

**Recommendation updated (Apr 8):** ACH push from external bank is the sole payment rail per Michelle Malavolta's recommendation. The SoFi Plus / Stripe payment rail remains valuable as a **future option** for debit card payments (instant confirmation and agent fallback). See Future Considerations.

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

### Option 1: Stripe (Debit Card) via SoFi Plus Pattern (not in scope)

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

### Option 3: PayNearMe (Multi-Method Third-Party) (not in scope)

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
| Status | Not in scope | Not in scope | Not in scope | **In scope (sole rail)** | Not in scope |

### Recommendation (updated Apr 9)

**What ships (July 31):** ACH push from external bank as the sole payment rail via the authenticated in-app experience. Member sees SoFi routing/account info, sends funds from their bank, confirms via "Confirm My Payment." Zero return risk. No debit card or agent payment processing.

**Not in scope but documented for reference:** Debit card (TabaPay/Stripe), guest portal (sofi.com/pay), PayNearMe. See Future Considerations.

---

## Immediate Next Steps

### From April 8, 2026 payment method deep dive (most recent)

| # | Action | Owner | Due |
|---|--------|-------|-----|
| 1 | Follow up with Michelle Malavolta on debit card via TabaPay for this use case | Shweta | This week |
| 2 | Engage Danielle (design) to review the healing flow UX | Shweta | Apr 9 (flag designs meeting) |
| 3 | Explore bill pay as a payment mechanism (from both SoFi side and external bank side) | Shweta | Next week |
| 4 | Investigate whether SoFi can generate unique temporary account numbers per charged-off member | Shweta / Banking Eng | TBD |
| 5 | Set up Friday meeting with Shahar to review polished PRD | Shweta | This week |
| 6 | Find out what the adverse action letter from EWS looks like (does it include an identifier the member could use?) | Shweta | This week |
| 7 | Confirm SoFi has a recovery account that can receive external ACH/wire (routing + account number for in-app payment flow) | Shweta / Michelle Malavolta / Finance | This week |

### From April 2026 team alignment meeting

1. **Get Joe Conlon and Mitchell Morris Jr. into the next session.** They own dispute operations. Need to finalize dispute requirements, tooling needs, and process design.
2. **Shweta to access EWS Data Contribution Slack channel files.** Two dispute process files, operating roles doc, and tech spec are in the Files tab. Review before next session.
3. **Shweta to follow up with contacts from Mili's office hour session.** Leverage feedback and leads received.
4. **Engage Daniel Evanko on new data elements.** Dispute flags require new fields in the EWS contribution files. Daniel needs to know what to build and where.

### From Michael Bourgeois sync (still relevant)

### 1. Payment infrastructure meeting: Michelle Malavolta + Serena (+ David Pinsky)
**Priority:** Highest. This is the #1 blocker.
**Purpose:** Resolve the fundamental question: how does SoFi accept a payment and ledger it when the member's account is permanently closed?
**Key questions:**
- Can we accept debit card payments for a closed account? Where do funds land?
- Can we accept ACH payments? Is that easier or harder than debit card?
- Is there a GL or intermediary account we can land payments in and sub-ledger to the correct charged-off account?
- Is temporarily reopening the account to accept payment a viable (if clunky) MVP path?
- How does core conversion affect any of this?
**Outcome needed:** Clarity on the easiest, cleanest payment method to accept for charged-off accounts. Then the experience can be designed around it.

### 2. Dispute workstream kickoff: Joe Conlon + Mitchell Morris Jr. (Special Operations)
**Priority:** High. Must start in parallel.
**Purpose:** Kick off the FCRA dispute workstream with the correct operators. (Corrected April 2026 alignment meeting: NOT Tammy Eisel. Joe Conlon and Mitchell Morris Jr. from Special Operations own dispute operations.)
**Key questions:**
- How does Joe's team currently handle similar processes? What tooling do they use?
- What does manual dispute handling look like for MVP?
- What does EPD need to build vs. what Joe's team handles manually?
- How do they update member records in source systems when a dispute is upheld?
- What case management system will they use?
- What new data elements are needed in the EWS contribution files for disputes? (Engage Daniel Evanko.)
**Outcome needed:** Joe Conlon and Mitchell Morris Jr. aligned on scope, timeline, and operational model. Clarity on what EPD needs to build for them.

### 3. Overall initiative alignment: Michelle Barnwell + Shahar Ronen + Michael Bourgeois
**Priority:** High. Ensures everyone is on the same page.
**Purpose:** Align on overall initiative timing, the penalty pathway, drop-dead dates, and how the workstreams fit together.
**Key questions:**
- What is Michelle's timeline for file generation and data contribution?
- What are the exact contractual deadlines and penalty milestones?
- Is there flexibility to negotiate timeline with EWS (e.g., $99K discount vs. $100K)?
- How do the workstreams (file gen, healing, dispute, compliance) sequence and depend on each other?
**Attendees:** Shweta, Michael Bourgeois, Michelle Barnwell, Shahar Ronen
**Outcome needed:** Shared understanding of the overall timeline and each team's commitments.

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
| **Payment methods for recovery** | Phone payments (agent takes card info), mail (check/money order), wire. Sometimes set-off from other accounts at same bank. | One payment rail: debit card (TabaPay, already used by collections) or ACH (per Anthony Zuffo). Check NOT acceptable. One rail is all that's required for compliance. |
| **ChexSystems/EWS record update** | Manual process. Member pays, then must separately request record update. Banks update within varying timelines (no standard). | **Automate the EWS update.** Payment triggers status change, reported in next daily batch (within 10 business days per contract). Member never has to make a separate request. |
| **Account reopening** | Almost universally not offered. Capital One explicitly does not reopen. Traditional banks offer "second chance" accounts as alternatives. | Not in scope, but possible in future with new core banking system. |
| **Settlement/partial payment** | Common in lending. Rare for deposit account charge-offs due to small balances. | Not planned. Median charge-off is $49. Full payment only. |
| **Transparency** | Members often discover the charge-off only when denied at another bank. No proactive notification about EWS impact. | **Proactive communication.** Update pre-charge-off warnings to mention EWS reporting. Post-charge-off: clear in-app messaging with exact balance owed and payment path. |
| **Third-party payment platforms** | PayNearMe used by lending divisions for multi-method collections (CC, debit, Venmo, cash at 31,000+ retail locations). Not used for deposit account recovery at any major bank. | Not planned. One rail is sufficient for compliance. PayNearMe available as a future option if volume justifies it. |

### Competitive positioning for SoFi

If SoFi builds a digital-first healing experience, it would be **first-to-market** among major banks and fintechs for deposit account charge-off recovery. Specific differentiators:

1. **In-app balance visibility.** No competitor shows the charged-off balance in their app with a clear "pay now" CTA. Members at other banks must call to learn their balance.
2. **One-tap payment.** No competitor offers digital payment for charged-off deposit accounts. Every other bank requires a phone call or mailed payment.
3. **Automated EWS/ChexSystems update.** No competitor automatically updates the record upon payment. Members at other banks must call or write separately to request the update.
4. **Transparent timeline.** Showing the member "once you pay, your EWS record will be updated within 10 business days" is better than the industry norm of vague promises.
5. **No adversarial collections.** Unlike traditional banks that send debts to collection agencies (damaging the member relationship), SoFi's approach treats recovery as a member service, not a collections operation.

This aligns with SoFi's brand positioning as a member-first financial institution and could generate positive press coverage ("SoFi becomes first bank to offer digital charge-off recovery").

---

## Future Considerations

The following items are explicitly **not in scope** but are documented here for reference. They may be revisited if volume, business need, or regulatory changes warrant it.

| Item | Why it's not in scope | When to reconsider |
|------|----------------------|-------------------|
| **Debit card payment (TabaPay or Stripe)** | One payment rail (ACH push) is all that's required for EWS compliance. Provides instant confirmation for members who prefer card payment and enables agent-assisted fallback. TabaPay already integrated at SoFi for collections (contact Tong Han); Stripe proven for member-facing flows (SoFi Plus). See Appendix F and G for full analysis. | If members express significant friction with the ACH push flow, or if agent-assisted payment becomes necessary due to high call volume from members who cannot access the app. |
| **Agent-assisted payment (phone)** | Agents direct members to self-service in the SoFi app. No payment processing over the phone. Would require debit card rail (TabaPay) for agents to collect card info. | If a significant number of members cannot access the app and call volume is high. Requires debit card integration first. |
| **Logged-out payment portal (sofi.com/pay)** | In-app self-service only. Most members who arrive via adverse action letters from other banks may not have app access, making this a high-priority future item. See guest portal design in Appendix G for reference architecture. | If call volume is high from members without app access after launch. |
| **PayNearMe integration** (CC, debit, Venmo, cash, Apple Pay) | One payment rail is all that's required for EWS compliance. Median charge-off is $49. Adding payment methods is a SoFi business decision, not a regulatory one. | If healing volume is significantly higher than expected and the single rail creates friction. PayNearMe is already at SoFi (lending recovery) so integration effort would be moderate. |
| **Unique temporary account numbers per member** | Would allow push payments to auto-map without the "Confirm My Payment" step. Unknown if SoFi can generate disposable account numbers today. Investigate with banking engineering. | If reconciliation becomes a significant ops burden or if the confirmation step causes member drop-off. |
| **AI-assisted reconciliation** | Parse incoming GL transactions, cross-reference against open charged-off balances by amount and timing, auto-match if confidence is high, queue for human review if not. | If unmatched payment volume creates ops overhead. |
| **Partial payments and payment plans** | Full balance only. Balances are small (median $49). Settlement negotiation adds complexity with minimal financial benefit. | If SoFi decides to actively pursue recovery as a revenue strategy (unlikely given balance sizes). |
| **Self-service dispute form (web/app)** | At ~35 to 50 disputes/month, mail-based intake with Joe Conlon's team is sufficient. Building a self-serve form is over-engineering for this volume. | If dispute volume exceeds 200/month or there are complaints about mail-only intake. |
| **Proactive outreach to eligible members** | Members learn about their charge-off through existing comms (57-day reminders, closure email) and adverse action letters from other FIs. Proactive outreach is not an EWS requirement. | If healing rate is near zero and there's a business case for active recovery. |
| **Account revival / re-engagement** | Charged-off accounts are permanently closed in the current core. New core could theoretically support reopening, but prerequisites unavailable until late August 2026. | Post new core migration, if there's a business case for re-engaging healed members. |
| **Analytics dashboard** | Basic event tracking (payment volume, healing rate, dispute volume) ships with launch. A full dashboard is not justified at initial volumes. | If stakeholders need more visibility or volumes increase. |
| **1099-C tax documents** | Full balance payment only; no partial forgiveness means no 1099-C needed. | If SoFi ever introduces settlement offers at less than full balance. |
| **Credit card payments** | Accepting credit cards for charge-off repayment creates additional compliance concerns and is not required. | No foreseeable trigger. |

---

## Appendix: Meeting Notes & Q&A

### Do we report charge offs to EWS? Is that why we need a repay flow?

> Shahar Ronen [3:59 PM]
> Our contract state we must report charge offs to EWS (so the ecosystem can benefit from knowing).
> Reported members could face adverse action (=other FI won't let them open accounts) and SoFi will be named as the source of data. So we must allow members repay and dispute information to clean their record.
> [3:59 PM]
> The urgency is driven by the extra $100k / month we're paying EWS until we contribute

### How to clear an account

- How to pay SoFi?
- How to manage risk?
- Facilitate payment after chargoff
- Recovery unit: reach out to member to settle your debt for 60% with installments (common practice)
- Build new agent facing workflows
- Loans remain open until they have a $0 balance. When someone charges off we transition to chargeoff balance. Loan remains open while principal balance AND charge-off balance are both open. So if a member agrees to the settlement, we set the ACH pull from a linked bank account.
- Some of the work is outsourced to 3rd parties: member sends a wire to SoFi. SoFi pays a fee to the companies to do so. The partner makes sure the payment hits the relevant account member.
- PayNearMe: zero barrier to entry. They'll take your payment in any way (CC, debit, Venmo, cash).
- In either case, we need to set up a ledger for repaying.

### CC account states pending delinquency

- You miss 1 payment
- You miss 2 payments: card open but frozen
- Then, card is closed but infra still supports working on the account ledger

### Ideas / questions

- Can we keep the account open? Probably not. How are other banks doing it?
- What is the average charge off balance?
- Can we sell the account, making it someone else's problem, and then report it as balanced?
- Can we forgive the members? Not a good idea.

### Next steps

- Ops and processes
- Reopen account: can we create a special state for charged off accounts (closed except for the ability to revive)
- If we migrate closed accounts, we could open new account / reopen the old account for them
- Onboarding new members in late August
- Setting up a GL
- Payment website with case number
- Can use Stripe or similar
- Member types info, we tell them how much to pay, then collect the funds
- Payment would take the red flag
- Repay by wire or ACH to an Account number

### EWS Data Contribution: Charge Off Healing and FCRA Disputes (working session with Tami Wilson, Joe Conlon)

- Member could dispute any data that were send:
  - Wrong address
  - Wrong balance
  - Not closed due to fraud
  - Etc.
- Can we reopen the account for the payment? Better not to reopen the account. But may be ok to handle the payment.
- Targeting MVP in production by July 31, 2026

### Workstreams

1. Report to EWS within N days of chargeoff (Michelle Barnwell)
2. Collect the payment (Shweta Kulshrestha)
   - ACH/wire/FedNow to a bank account / routing number: are we OK to do this?
   - What deposit method should work?
   - What's the UX:
     - Ops only: call a number, agent takes payment
     - UI only: login to your account, info already there, pay, and collect the context
   - Keep in mind:
     - Member needs to know the amount to pay
     - Member needs to know how to pay
     - SoFi needs to know to which account to apply the payment
   - Examples:
     - SoFi Plus collects 3rd party payments
     - Smart Card 3rd-party
3. Apply payment to the account (Shweta Kulshrestha?)
   - Set up a GL account for receiving payments
   - Re-open the account in some way so it can receive the payment
   - We charge off the account after 57 days
   - Update status of the account
   - Report on mitigation
4. Enable members to dispute and trigger investigation (Srikant Movva?)
   - Fix if wrong or provide an update. There are different requirements around various data elements as to how long we have to respond, when we need to pause, etc.

### Open questions from sessions

- What are typically dispute rates for reported accounts (ask EWS)
- How will users contact SoFi? What's on the adverse action?

### For compliance

- Acceptable payment methods
- Flexibility with "closed state"

# Screen 8: Case

**Route:** `/cases/:caseId`
**Platform:** Mobile
**Presentation:** Page
**Purpose:** Show the true state of a failed or reversed payment and drive a refund or reconciliation to a resolution.
**Visual Focus:** The current payment state and its firm expected timeline.
**Action Hierarchy:** Primary: none required (a status screen); Secondary: contact support; Tertiary: view the related receipt.
**Entry Points:** [Home](03-home.md), [Receipt Detail](07-receipt-detail.md), a payment notification.
**Exit Points:** [Receipt Detail](07-receipt-detail.md); back to the previous screen.

### Layout

**Mobile layout:**
- AppBar with a back action and the case reference.
- A status block: current state, last update time, and the expected resolution window.
- A timeline group of state changes, newest first.
- A secondary "Contact support" action and a tertiary "View receipt" link.

### Components

| Component | Type | Platform | Purpose | Data Source |
|---|---|---|---|---|
| Status Pill | Badge | Mobile | Show pending, reconciling, refunding, escalated, resolved | Payment state machine |
| Confirmation Summary | Container | Mobile | Show amount, payee, reference | Payment record |
| Timeline | List | Mobile | Show state changes and times | Case audit trail |
| Support action | OutlinedButton | Mobile | Reach support with the case reference | Support channel |

### Data Requirements

**Data Needed:**
- Payment state and history — from the payment state machine
- Refund or reconciliation status — from the rail or biller
- Case record and audit trail — from the case store

**Data Flow:**
1. Screen mounts → show the current state and timeline.
2. On a debit without delivery → initiate a refund and update the case.
3. Rail and biller disagree → reconcile both sources before refunding.
4. Case exceeds its window → escalate to the payment partner and notify the user.

### States

| State | Condition | What the User Sees |
|---|---|---|
| **Pending** | Outcome not yet known | Pending pill and last update time |
| **Reconciling** | Checking rail and biller | Reconciling pill; pending animation |
| **Refund initiated** | Refund sent to the rail or biller | Refund pill and expected timing |
| **Awaiting biller** | Waiting on the biller | Waiting pill with the window |
| **Escalated** | Window exceeded | Escalation notice and support route |
| **Resolved** | Refund settled or payment confirmed | Resolved pill; link to the receipt |
| **Offline** | No connectivity | Last known state and offline banner |

**Edge Cases:**
- Duplicate refund detected by reference → blocked, with the reason shown.
- Rail reports success while the biller reports failure → reconcile before any refund.
- Case exceeds the service window → escalate rather than leaving it open silently.

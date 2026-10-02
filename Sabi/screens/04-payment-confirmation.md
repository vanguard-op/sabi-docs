# Screen 4: Payment Confirmation

**Route:** `/pay/:intentId`
**Platform:** Mobile
**Presentation:** Page
**Purpose:** Show the amount, payee, and charges for a resolved obligation and take explicit authorisation.
**Visual Focus:** The Confirmation Summary with the total amount.
**Action Hierarchy:** Primary: Authorise payment; Secondary: change the amount (variable bills); Tertiary: view charges and terms.
**Entry Points:** [Home](03-home.md).
**Exit Points:** [Identity Verification](05-identity-verification.md) when KYC is required; [Receipt Detail](07-receipt-detail.md) on success; [Case](08-case.md) when the outcome is unresolved; [Home](03-home.md) on back.

### Layout

**Mobile layout:**
- AppBar with a back action and the payee or service name.
- Confirmation Summary block: amount, payee, charges, and reference grouped together.
- A change-amount link for variable bills.
- A primary "Authorise payment" button pinned at the bottom.
- A text-and-form fallback region, used when a composed component is refused.

### Components

| Component | Type | Platform | Purpose | Data Source |
|---|---|---|---|---|
| Composed Task Surface | Container | Mobile | Render the confirmation from the client catalogue | Agent UI stream (FEAT-011) |
| Confirmation Summary | Container | Mobile | Show amount, payee, charges, reference | Resolved intent |
| Authorise button | ElevatedButton | Mobile | Take explicit authorisation | Static |
| Status Pill | Badge | Mobile | Show authorising, failure, or reconciling | Payment state |

### Overlays

| Overlay | Pattern | Trigger | Behaviour | States |
|---|---|---|---|---|
| Amount entry | Bottom sheet | Variable bill with no known amount | Blocks until entered; validates against the biller range | Default, Validation error |
| Re-authorisation | Dialog | Session expires before authorisation | Blocks until re-authenticated | Default |

### Data Requirements

**Data Needed:**
- Resolved intent: target, amount, charges, reference — from the intent created on Home
- Biller amount range, where variable — from the service catalogue
- Verification status — from the account

**Data Flow:**
1. Screen mounts → show the summary.
2. Authorise → if KYC is required and unverified, route to Identity Verification and hold the payment.
3. Otherwise → sign a mandate bound to amount, payee, and user → execute through the licensed rail.
4. Success → retrieve and store the receipt → Receipt Detail.
5. Failure or unknown outcome → state a reason and next step, or open a Case.

### States

| State | Condition | What the User Sees |
|---|---|---|
| **Default** | Summary ready | Amount, payee, charges, authorise action |
| **Amount entry** | Variable bill | Amount sheet over the summary |
| **Authorising** | Mandate signed, rail called | Progress on the action; fields locked |
| **Held for KYC** | Regulated payment, unverified | Notice that verification is needed; payment held, not failed |
| **Failure** | Rail or biller declined | Plain reason and next step; no partial debit |
| **Reconciling** | Outcome unknown | Pending notice; no blind retry |
| **Offline** | No connectivity | Offline banner; authorisation unavailable |

**Edge Cases:**
- Duplicate authorisation (double tap or retried session) → execute once and return the same reference.
- Amount above the user's set limit → require a second confirmation.
- Cash-only counter is the only route → state it, since Sabi cannot pay cash.
- Switched agency collection account → route to the current channel before payment.

# Screen 5: Identity Verification

**Route:** `/account/verify`
**Platform:** Mobile
**Presentation:** Page
**Purpose:** Verify the user's identity once against the national identity infrastructure, so regulated payments can proceed.
**Visual Focus:** The verification status and the single verify action.
**Action Hierarchy:** Primary: Verify; Secondary: correct details; Tertiary: how identity data is used.
**Entry Points:** [Account](09-account.md); [Payment Confirmation](04-payment-confirmation.md) when a payment requires KYC.
**Exit Points:** [Payment Confirmation](04-payment-confirmation.md) to resume a held payment; [Account](09-account.md).

### Layout

**Mobile layout:**
- AppBar with a back action.
- A status header using the Status Pill (unverified, pending, verified, rejected).
- A form group: NIN, and BVN only when the payment rail requires it.
- A primary "Verify" button.
- A tertiary data-use note stating how identity data is handled and retained.

### Components

| Component | Type | Platform | Purpose | Data Source |
|---|---|---|---|---|
| Status Pill | Badge | Mobile | Show verification state | Account record |
| NIN field | TextField | Mobile | Collect the national identity number | User input |
| BVN field | TextField | Mobile | Collect the bank verification number when required | User input |
| Verify button | ElevatedButton | Mobile | Submit for verification | Static |

### Data Requirements

**Data Needed:**
- Identity details (NIN, BVN) — from user input
- Verification result — from the identity verification provider
- Held payment reference, when entered from a payment — from the payment record

**Data Flow:**
1. Screen mounts → show current verification status.
2. Submit → call the identity provider → store the verification status on the account.
3. Verified → return to the held payment, which resumes, or to Account.
4. No match → show a plain reason and allow one correction.

### States

| State | Condition | What the User Sees |
|---|---|---|
| **Default** | Not yet verified | Empty fields and the verify action |
| **Checking** | Provider call in progress | Progress on the action; fields locked |
| **Verified** | Match returned | Verified status; no further action |
| **No match** | No identity match | Plain reason and one correction attempt |
| **Pending provider** | Identity provider unavailable | Pending status; discovery remains open |
| **Rejected** | Verification rejected or account suspended | Reason and route to support |

**Edge Cases:**
- A verified account later flagged for fraud → payment features suspended; audit trail preserved.
- Second failed correction → route to support rather than looping.
- Identity provider unavailable → queue and inform; never block discovery.

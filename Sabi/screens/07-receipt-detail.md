# Screen 7: Receipt Detail

**Route:** `/receipts/:receiptId`
**Platform:** Mobile
**Presentation:** Page
**Purpose:** Show one receipt with its official identifier and verification state, and let the user share or export it. This screen is an alternate surface: the same receipt can be rendered inline in the conversation as a composed record surface, and neither route is the fallback for the other.
**Visual Focus:** The receipt amount and its verified official identifier.
**Action Hierarchy:** Primary: Share; Secondary: export; Tertiary: raise a dispute.
**Entry Points:** [Receipts](06-receipts.md), [Home](03-home.md), [Payment Confirmation](04-payment-confirmation.md). From Home it is reached either as a composed inline surface or as the alternate screen, left to the user.
**Exit Points:** [Case](08-case.md); back to the previous screen. A dispute is also reachable inline in the conversation.

### Layout

**Mobile layout:**
- AppBar with a back action and the receipt kind.
- A receipt block: payee, amount, reference, date, and, where it applies, the FTeR identifier with a verification line.
- A Status Pill stating awaiting, verified, or unverified.
- A share action group (primary) and an export action (secondary).
- A tertiary dispute link.

### Components

| Component | Type | Platform | Purpose | Data Source |
|---|---|---|---|---|
| Receipt block | Card | Mobile | Show the payment's full record | Receipt record |
| Status Pill | Badge | Mobile | Show receipt verification state | Receipt record |
| Share action | ElevatedButton | Mobile | Send the receipt to a person or channel | System share |
| Export action | TextButton | Mobile | Export the receipt as a document | Receipt record |
| Dispute link | TextButton | Mobile | Open a dispute on this payment | Case workflow |

### Data Requirements

**Data Needed:**
- Receipt record: reference, amount, payee, timestamp, kind, official identifier — from receipt records
- Verification status against the source and, when unverified, the plain reason (`Receipt.unverifiedReason` in [schema](../schema.md)) — from the receipt service

**Data Flow:**
1. Screen mounts → show the receipt.
2. If an official identifier exists → show it marked verified or unverified against the source.
3. Share or export → produce a link or document.
4. Dispute → open a Case and preserve the original receipt unaltered.

### States

| State | Condition | What the User Sees |
|---|---|---|
| **Loading** | Receipt being fetched | Skeleton |
| **Loaded** | Receipt available | Full record and actions |
| **Awaiting receipt** | Payment succeeded, official identifier pending | Awaiting pill and a note that Sabi is still checking |
| **Verified** | Official identifier confirmed | Verified line with the identifier |
| **Unverified** | Source cannot confirm | Unverified line with the reason (`Receipt.unverifiedReason` in [schema](../schema.md)) |
| **Error** | Fetch failed | Error state with "Try again" |

**Edge Cases:**
- Source unavailable for verification → return pending, not a false negative.
- A disputed receipt → preserve the original and record the dispute without altering it.
- Sharing a receipt with a third party → a link or record, never raw account data.

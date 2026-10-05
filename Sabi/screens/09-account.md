# Screen 9: Account

**Route:** `/account`
**Platform:** Mobile
**Presentation:** Page
**Purpose:** Manage identity, linked channels, and payment methods, and show verification and account status. Account remains fully functional; the identity ceremony is composed inline in the conversation when a payment requires KYC, and the Identity Verification screen remains the route from here.
**Visual Focus:** The verification status and the single action it implies.
**Action Hierarchy:** Primary: Verify identity (when unverified); Secondary: link a channel; Tertiary: manage methods and sign out.
**Entry Points:** The Account tab of the navigation shell; [Home](03-home.md).
**Exit Points:** [Identity Verification](05-identity-verification.md); [Welcome](01-welcome.md) on sign out.

### Layout

**Mobile layout:**
- AppBar with title "Account".
- A profile group: display name, locale preference, and account status.
- A verification row showing status with a route to Identity Verification when unverified.
- A linked-channels group: phone-free discovery identity, USSD number, WhatsApp number.
- A payment-methods group.
- A sign-out action in secondary text at the base.
- BottomNavigationBar with 3 tabs: Home, Receipts, Account.

### Components

| Component | Type | Platform | Purpose | Data Source |
|---|---|---|---|---|
| Status Pill | Badge | Mobile | Show account and verification status | Account record |
| Locale selector | DropdownButton | Mobile | Choose the response language | Account preference |
| Channel link row | List item | Mobile | Show and link a USSD or WhatsApp number | Channel links |
| Payment method row | List item | Mobile | Show a saved or linked method | Payment methods |
| Sign out action | TextButton | Mobile | End the session | Static |

### Data Requirements

**Data Needed:**
- Account profile, status, and locale — from the account record
- Verification status — from the identity verification service
- Linked channels — from channel links
- Saved payment methods — from the payment module

**Data Flow:**
1. Screen mounts → show profile, verification, channels, and methods.
2. Verify identity → Identity Verification.
3. Link a channel → attach the number, or reject if it belongs to another account.
4. Change locale → re-render the app in the chosen language.

### States

| State | Condition | What the User Sees |
|---|---|---|
| **Loading** | Data being fetched | Skeleton |
| **Loaded** | Account loaded | Profile, verification, channels, methods |
| **Unverified** | Not verified | Verify action as the primary action |
| **Pending** | Verification or channel link pending | Pending pill with the reason |
| **Verified** | Verified | Verified status; no action required |
| **Suspended** | Account flagged | Reason shown; payment actions disabled |
| **Offline** | No connectivity | Last known data and offline banner |

**Edge Cases:**
- Linking a number already owned by another account → reject and notify the owner.
- Locale change with a missing translation → fall back to English with a note.
- Account suspended → payment features disabled; the audit trail is preserved and support is offered.

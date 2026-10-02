# Screen 1: Welcome

**Route:** `/welcome`
**Platform:** Mobile
**Presentation:** Page
**Purpose:** Let a new user create an account with a passkey, or sign in, without a phone number or password.
**Visual Focus:** The single "Get started" primary action, with the one-line value proposition above it.
**Action Hierarchy:** Primary: Get started; Secondary: Sign in; Tertiary: how Sabi handles your data.
**Entry Points:** App first open.
**Exit Points:** [Home](03-home.md) on account creation; [Sign In](02-sign-in.md).

### Layout

**Mobile layout:**
- Centred column: the brand mark, the tagline, and a one-line explanation of what Sabi does.
- A "Get started" primary button with a "Sign in" TextButton below it.
- A tertiary data-handling link in secondary text, shown before any account is created.
- No navigation shell while unauthenticated.

### Components

| Component | Type | Platform | Purpose | Data Source |
|---|---|---|---|---|
| Brand mark | Image | Mobile | Identity on first open | Shared asset |
| Tagline and value line | Text | Mobile | State what Sabi does | Static content |
| Get started button | ElevatedButton | Mobile | Begin account creation and passkey enrolment | Static |
| Sign in link | TextButton | Mobile | Route to sign in | Static |
| Data-handling link | TextButton | Mobile | State how personal data is used before consent | Consent copy |

### Overlays

| Overlay | Pattern | Trigger | Behavior | States |
|---|---|---|---|---|
| Recovery code | Bottom sheet | Account created | Blocks until acknowledged; shown once; not linkable | Default, Acknowledged |

### Data Requirements

**Data Needed:**
- Passkey capability of the device — from the client platform
- Localized consent copy — from static content bundle

**Data Flow:**
1. Screen mounts → evaluate passkey capability.
2. "Get started" → create account → run passkey ceremony.
3. On success → show the recovery code once → acknowledge → navigate to Home.
4. On failure → error state with retry; no partial account is left behind.

### States

| State | Condition | What the User Sees |
|---|---|---|
| **Loading** | Checking passkey capability | Spinner on the action |
| **Default** | Capability known | Tagline and actions |
| **Creating account** | Account creation in progress | Button spinner; actions disabled |
| **Passkey unsupported** | Device has no passkey support | Fallback explanation with the reduced security stated |
| **Offline** | No connectivity | Offline banner; account creation queued |
| **Error** | Creation failed | Error state with "Try again" |

**Edge Cases:**
- Recovery code acknowledged but not saved → account is created anyway; the warning repeats once on the next sign-in.
- Deep link into a protected screen while unauthenticated → this screen, then return to the requested screen after authentication.
- Two users attempt to link the same channel number → the second link is rejected and the owner is notified.

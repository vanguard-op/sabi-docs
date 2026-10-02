# Screen 2: Sign In

**Route:** `/sign-in`
**Platform:** Mobile
**Presentation:** Page
**Purpose:** Return an existing user to their account with a passkey, or with a recovery code when the only passkey is lost.
**Visual Focus:** The single passkey sign-in action.
**Action Hierarchy:** Primary: Continue with passkey; Secondary: Use a recovery code; Tertiary: Create an account.
**Entry Points:** [Welcome](01-welcome.md).
**Exit Points:** [Home](03-home.md) on success; [Welcome](01-welcome.md) to create an account.

### Layout

**Mobile layout:**
- Brand mark with a short sign-in heading.
- A "Continue with passkey" primary button.
- A "Use a recovery code" TextButton that reveals the recovery-code entry region in place.
- A tertiary "Create an account" link.
- No navigation shell.

### Components

| Component | Type | Platform | Purpose | Data Source |
|---|---|---|---|---|
| Brand mark | Image | Mobile | Identity | Shared asset |
| Passkey button | ElevatedButton | Mobile | Run the passkey ceremony | Static |
| Recovery code entry | TextField | Mobile | Accept a saved recovery code | User input |
| Create account link | TextButton | Mobile | Route to Welcome | Static |

### Data Requirements

**Data Needed:**
- Passkey assertion — from the device credential store
- Recovery code — from user input

**Data Flow:**
1. Screen mounts → ready the passkey credential.
2. User signs in with passkey, or reveals and submits a recovery code.
3. Verified → navigate to Home. On a recovery-code sign-in → prompt to enrol a new passkey.
4. Failed → error state; no session is created.

### States

| State | Condition | What the User Sees |
|---|---|---|
| **Default** | Ready to sign in | Passkey action and alternatives |
| **Passkey prompt** | Device ceremony running | System prompt; no app UI |
| **Recovery code entry** | User chose the recovery route | Code field with submit |
| **Verifying** | Credential or code is being checked | Spinner; inputs disabled |
| **Error** | Credential or code rejected | Inline reason and retry |

**Edge Cases:**
- Passkey lost and recovery code lost → state that the account is unrecoverable.
- Session expired while on a later screen → return here through the shared Re-authorisation overlay, then back to the original screen.

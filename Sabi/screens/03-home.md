# Screen 3: Home

**Route:** `/home`
**Platform:** Mobile
**Presentation:** Page
**Purpose:** Let the user state what they need to pay in their own words, resolve it, and reach confirmation.
**Visual Focus:** The composer at the base of the screen and the composed surface it produces above.
**Action Hierarchy:** Primary: Send; Secondary: select a discovery result; Tertiary: clarify or ask for help.
**Entry Points:** [Welcome](01-welcome.md), [Sign In](02-sign-in.md), the Home tab of the navigation shell, [Payment Confirmation](04-payment-confirmation.md) back action.
**Exit Points:** [Payment Confirmation](04-payment-confirmation.md), [Receipts](06-receipts.md), [Account](09-account.md), [Case](08-case.md).

### Layout

**Mobile layout:**
- AppBar with title "Home" and a trailing Account IconButton.
- Body: a conversation List of Composed Task Surfaces, Service Result Rows, and status feedback in time order.
- A composer TextField pinned above the navigation shell, with a send action.
- BottomNavigationBar with 3 tabs: Home, Receipts, Account.
- An inline clarification region appears within the conversation when confidence is low.

### Components

| Component | Type | Platform | Purpose | Data Source |
|---|---|---|---|---|
| Composer | TextField | Mobile | Take a free-form request in English or Pidgin | User input |
| Composed Task Surface | Container | Mobile | Render the agent-composed result, list, or prompt | Agent UI stream (FEAT-011) |
| Service Result Row | List item | Mobile | Show a collecting body and known amount | Resolved discovery (FEAT-004) |
| Status Pill | Badge | Mobile | Show the state of a recent payment or receipt | Receipt and payment records |
| Receipt Card | Card | Mobile | Show a recent receipt | Receipt records |
| Clarification prompt | Inline region | Mobile | Ask one question with the closest candidates | Intent resolution (FEAT-003) |

### Overlays

| Overlay | Pattern | Trigger | Behaviour | States |
|---|---|---|---|---|
| Provider picker | Bottom sheet | A request matches more than one collecting body | Blocks until a choice is made; not linkable; the conversation is preserved | Default |
| Re-authorisation | Dialog | Session expires during use | Blocks until re-authenticated; preserves screen state | Default |

### Data Requirements

**Data Needed:**
- Recent receipts and payment statuses (max 10) — from receipt and payment records
- Catalogue matches for a request — from service search (FEAT-004)
- Resolved intent with candidates and confidence — from model serving (FEAT-003)

**Data Flow:**
1. Screen mounts → show loading, then recent activity or the empty state.
2. User submits a request → resolve intent → render the composed surface.
3. Multiple matches → provider picker; single match → continue.
4. Reaching confirmation → navigate to Payment Confirmation.
5. Failure → inline error with retry, preserving the request.

### States

| State | Condition | What the User Sees |
|---|---|---|
| **Loading** | First load of recent activity | Skeleton list |
| **Empty** | No history yet | Empty-state illustration, "No payments yet", and a prompt to ask for one |
| **Loaded** | Activity loaded | Conversation with composed surfaces |
| **Composing** | A request is being resolved | Composer disabled with progress |
| **Discovery results** | Matches found | Composed list of Service Result Rows |
| **Clarifying** | Confidence below threshold | One clarifying question with the closest candidates |
| **Not covered** | No catalogue match | Plain statement, query recorded, human support offered |
| **Offline** | No connectivity | Offline banner; requests queued; no authorisation |
| **Error** | Resolution failed | Error state with "Try again" |

**Edge Cases:**
- Ambiguous target matching several agencies → present the top candidates and require a choice.
- Retrieved content is untrusted and cannot override payment policy.
- A request that names another person's account → refuse and explain.
- Very long free-form input → wrap and keep the send action reachable.

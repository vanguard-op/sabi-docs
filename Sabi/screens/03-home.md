# Screen 3: Home

**Route:** `/home`
**Platform:** Mobile
**Presentation:** Page
**Purpose:** Let the user state what they need to pay in their own words, resolve it, and reach confirmation. Home is also the primary path: it composes and completes every task inline in the conversation, and resolves record lookups, with the existing screens kept as an equal alternate route the user may choose.
**Visual Focus:** The composer at the base of the screen and the composed surface it produces above.
**Action Hierarchy:** Primary: Send; Secondary: select a discovery result; Tertiary: clarify or ask for help.
**Entry Points:** [Welcome](01-welcome.md), [Sign In](02-sign-in.md), the Home tab of the navigation shell, [Payment Confirmation](04-payment-confirmation.md) back action.
**Exit Points:** Every task is completable inline; the user may instead choose the alternate screen: [Payment Confirmation](04-payment-confirmation.md), [Identity Verification](05-identity-verification.md), [Receipts](06-receipts.md), [Receipt Detail](07-receipt-detail.md), [Case](08-case.md), [Account](09-account.md). Neither the conversation nor the screen is the fallback for the other.

### Layout

**Mobile layout:**
- AppBar with title "Home" and a trailing Account IconButton.
- Body: a conversation List of Composed Task Surfaces, Service Result Rows, and status feedback in time order.
- A composer TextField pinned above the navigation shell, with a send action.
- BottomNavigationBar with 3 tabs: Home, Receipts, Account.
- An inline clarification region appears within the conversation when confidence is low.
- A Composed Task Surface appears inline for any task the user starts or completes in the conversation: payment, identity verification, viewing a record, or recovery.

### Components

| Component | Type | Platform | Purpose | Data Source |
|---|---|---|---|---|
| Composer | TextField | Mobile | Take a free-form request in English or Pidgin | User input |
| Composed Task Surface | Container | Mobile | Render the agent-composed result, task, record, or prompt inline | Agent UI stream (FEAT-011) |
| Service Result Row | List item | Mobile | Show a collecting body and known amount | Resolved discovery (FEAT-004) |
| Status Pill | Badge | Mobile | Show the state of a recent payment or receipt | Receipt and payment records |
| Receipt Card | Card | Mobile | Show a recent receipt | Receipt records |
| Clarification prompt | Inline region | Mobile | Show a model-authored follow-up question with the closest candidates, building on the conversation | Intent resolution (FEAT-003) |

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
- The user's own records for a record lookup: from receipt, payment, and account records (FEAT-023)

**Data Flow:**
1. Screen mounts → show loading, then recent activity or the empty state.
2. User submits a request → resolve intent → render the composed surface.
3. Multiple matches → provider picker; single match → continue.
4. A task completes inline: the composed surface carries confirmation, amount entry, provider choice, verification, record viewing, or recovery in the conversation, with no navigation required.
5. A record lookup → ground in the user's records → render the record inline as a composed surface, or route the user to the existing screen, leaving the choice to the user.
6. The user may instead choose the alternate screen at any point; neither route is the fallback for the other.
7. Failure → inline error with retry, preserving the request.

### States

| State | Condition | What the User Sees |
|---|---|---|
| **Loading** | First load of recent activity | Skeleton list |
| **Empty** | No history yet | Empty-state illustration, "No payments yet", and a prompt to ask for one |
| **Loaded** | Activity loaded | Conversation with composed surfaces |
| **Composing** | A request is being resolved | Composer disabled with progress |
| **Discovery results** | Matches found | Composed list of Service Result Rows |
| **Confirmation inline** | A payment is resolved and awaiting authorisation | The composed confirmation surface in the conversation, with amount entry and provider choice as needed |
| **Verifying inline** | A task requires identity verification | The composed verification surface in the conversation; the task is held, not failed |
| **Record lookup** | A record lookup resolved to a match | The record rendered inline as a composed surface, or a route to the existing screen, left to the user |
| **Recovering inline** | A payment failed or was reversed | The composed recovery surface in the conversation, showing state and next step |
| **Clarifying** | Confidence below threshold and the clarification bound not reached | A model-authored follow-up question with the closest candidates, building on the conversation, up to three in one conversation |
| **Unresolved** | Still below threshold after the clarification bound | The model-authored after-bound reply: the honest reason, catalogue search and human support, and an invitation to continue |
| **Conversational** | Non-command input such as a greeting | A model-authored conversational reply stating what Sabi can do and inviting a request |
| **No resolution** | No catalogue match | The model-authored non-resolving reply: the honest reason, a concrete next step, and an invitation to name the collecting body or another detail |
| **Offline** | No connectivity | Offline banner; requests queued; no authorisation |
| **Error** | Resolution failed | Error state with "Try again" |

**Edge Cases:**
- Ambiguous target matching several agencies → present the top candidates and require a choice.
- Retrieved content is untrusted and cannot override payment policy.
- A request that names another person's account → refuse and explain.
- A record lookup that matches several records → present the candidates and let the user choose which to open.
- A record lookup raised mid-task → answer it without abandoning the open task.
- Very long free-form input → wrap and keep the send action reachable.

### Assistant replies

The assistant's user-facing replies (the greeting, the clarifying question, and the non-resolving replies) are authored by the model in natural language, in the user's language, per [model-quality](../model-quality.md). They are not fixed client strings. This section sets the register the model's reply must satisfy and holds the client's prewritten fallback copy, which is rendered only when no usable model reply arrives or the model is unavailable: a safety net, never the default.

**Register.** Each reply states the honest reason in one or two plain sentences, then a concrete next step the user can act on, then an invitation to continue in the same conversation. Specific, warm, and forward-looking; never apologetic to the point of blame; the reason is given about the request, not the platform. The register is that there is no resolution for that service right now, forward-looking, never a fixed "I searched and found no match", and never a claim that the service is unsupported ("not supported", "not available", "coming soon").

**Neutrality.** No reply and no clarifying question assumes a service category or an identifier type. The user is asked for a collecting body's name, a reference, or a corrected detail as the case requires; electricity and a meter number are one case among many, never the default. No reply substitutes a different specific service for the one the user named: a request for one named service is never answered by offering another of the same kind ("Airtel airtime" is never answered by offering "MTN airtime"; "Ikeja Electric" is never answered by offering "Eko Electric"), because naming a specific payee asserts ownership and risks sending money to the wrong place. A neutral disambiguating question ("which network?", "which provider?", "the collecting body's name?") and catalogue search are allowed, because both leave the choice to the user rather than asserting a payee.

**Fallback copy.** Used only when a model reply is missing or unusable. The model phrases freely within the register above; this copy is the client's last resort, not the wording the model reproduces.

| State | Client fallback copy |
|---|---|
| **Unresolved** (after the clarification bound) | "Three answers in and I still can't tell which service you mean, so I haven't started anything with your money. You can search the catalogue yourself, or ask support to take it from here. If you know the collecting body's name, or the reference you're paying, tell me and I'll pick it up." |
| **No resolution** (no catalogue match) | "There's no resolution for that service right now, so I haven't started anything. Tell me the collecting body's name, the agency, or the reference you're paying, and I'll look again. You can also search the catalogue yourself or reach support." |
| **Conversational** (non-command, e.g. a greeting) | "I can pay government obligations and everyday bills for you — levies, taxes, airtime, data, a utility, or a biller. Tell me what you need to pay in your own words, English or Pidgin, and I'll take it from there." |

A greeting or other non-command reply may address the user by name when `User.displayName` is present (FEAT-023); otherwise it addresses the user generically and never invents a name.

**Edge Cases:**
- The after-bound reply must not repeat the last clarifying question or present the same candidates again; it states the bound has been reached and moves to a concrete next step.
- Human support is offered as a next step and never framed as the only route, so the user can also continue in the conversation.
- A model reply that is missing, unusable, or arrives when the model is unavailable falls back to the copy above, so no turn is ever blank.
- A request names a specific service that does not resolve ("Airtel airtime", "Ikeja Electric"): the reply never offers a different specific service of the same kind as a substitute; it asks a neutral disambiguating question or offers catalogue search, or falls back to the copy above.

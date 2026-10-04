# Screen 6: Receipts

**Route:** `/receipts`
**Platform:** Mobile
**Presentation:** Page
**Purpose:** Let the user find and open any stored receipt by reference, agency, or date. This screen is an alternate surface: a record lookup in the conversation can render a receipt inline or route here, and neither route is the fallback for the other.
**Visual Focus:** The search field and the list of Receipt Cards beneath it.
**Action Hierarchy:** Primary: open a receipt; Secondary: search; Tertiary: export.
**Entry Points:** The Receipts tab of the navigation shell; [Home](03-home.md), which may route here from a record lookup in the conversation. The user chooses the route.
**Exit Points:** [Receipt Detail](07-receipt-detail.md).

### Layout

**Mobile layout:**
- AppBar with title "Receipts".
- A search TextField group pinned below the AppBar.
- Body: a List of Receipt Cards, newest first, grouped by month.
- BottomNavigationBar with 3 tabs: Home, Receipts, Account.

### Components

| Component | Type | Platform | Purpose | Data Source |
|---|---|---|---|---|
| Search field | TextField | Mobile | Filter by reference, agency, or date | User input |
| Receipt Card | Card | Mobile | Show reference, payee, amount, date, kind | Receipt records |
| Status Pill | Badge | Mobile | Show awaiting, verified, or unverified | Receipt record |

### Data Requirements

**Data Needed:**
- Receipts for the account, paginated — from receipt records
- Search results for a query — from receipt search

**Data Flow:**
1. Screen mounts → show loading, then the list or the empty state.
2. A search query → filter and render matching Receipt Cards.
3. Open a card → Receipt Detail.
4. Failure → error state with retry.

### States

| State | Condition | What the User Sees |
|---|---|---|
| **Loading** | First load | Skeleton list |
| **Empty** | No receipts yet | Empty-state illustration, "No receipts yet", and a prompt to make a first payment |
| **Loaded** | Receipts fetched | Grouped list of Receipt Cards |
| **Searching** | Query in progress | Results replace the list |
| **No results** | Search matched nothing | Clear message with the query and a way to clear it |
| **Error** | Fetch failed | Error state with "Try again" |
| **Offline** | No connectivity | Offline banner; cached receipts shown if available |

**Edge Cases:**
- Large history → paginate; do not block the first screenful.
- A receipt awaiting its official identifier → show the awaiting pill rather than hiding it.
- Receipt storage failure → warn the user not to discard the transaction reference.

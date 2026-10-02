# Sabi: branding

## Brand Identity

**Product Name:** Sabi
**Tagline:** Say it. Pay it. Prove it.
**Brand Personality:** Dependable, plain-spoken, human, local, calm
**Tone of Voice:** Direct and reassuring, in plain Nigerian English and Pidgin. Talks like a capable person, not a brochure: it states the amount, the outcome, and the proof. Never salesy, never gamified, never bureaucratic.

## Design Character

**Audience:** Nigerian adults and small businesses on low-end to mid-range Android or feature phones who owe recurring obligations and everyday bills ([user-personas](user-personas.md)).

**Feels like:** Dependable, clear, human, grounded, calm.

**Must not feel like:** A premium or exclusive bank; a cold government portal; a betting or gamified app; a WhatsApp clone. The audience is mass-market and affordability-constrained, and the subject is real money, so the design reads as trustworthy and uncluttered rather than premium, clinical, or playful.

**Motion character:** Calm and purposeful; a payment confirmation lands with weight rather than celebration.

**Research basis:**
- [Paystack visual identity](https://paystack.com/assets/website/downloads/paystack-visual-identity.pdf) — borrowed the "trusting easy" split: soft, familiar forms for ease, colour and clarity for trust, carried with a clear-not-complex tone.
- [Fintech trust palettes](https://uicolors.org/trends/fintech-trust) — borrowed the discipline of a strict semantic colour system where blue and green carry meaning and colour never states a status on its own.
- [N26 design system](https://opendesign.cc/en/sites/n26) — borrowed the single dominant accent on a warm off-white base, a friendly-but-professional register, and one primary action per surface.

The space this direction claims: incumbents (OPay, PalmPay, Moniepoint) compete on speed, rewards, and cashback ([PalmPay](https://www.palmpay.com/ng), [OPay](https://www.opayweb.com/about-us)). Sabi competes on correctness and proof, so its character leads with trust and clarity rather than excitement.

## Brand Mark & Logo System

**Mark Type:** combination (pictorial symbol with a wordmark lockup)
**Concept:** A speech bubble whose lower tail tears like the edge of a receipt. The mark carries the product's whole arc in one form: a conversation that produces written proof.
**Intended Reading:** A conversation that ends in a receipt.
**Must Not Read As:** WhatsApp's green chat bubble; a generic verified tick; the wordmark geometry of OPay, PalmPay, or Paystack; a shopping bag or a price tag; a torn envelope or a map pin.
**Symbolism:** The rounded bubble is speech and conversation; the torn zigzag edge is the printed receipt; the tear repeats as the brand's recurring graphic device.

### Logo Usage
**Backgrounds:** Must survive the warm off-white surface, the deep green brand colour, and photography. On dark surfaces the mark uses its light form.
**Graphic Device:** On busy or photographic backgrounds the symbol sits inside a rounded-square (squircle) container, so the torn edge never competes with surrounding detail.
**Minimum Size:** Digital: symbol 24px, wordmark lockup 96px wide. Print: symbol 8mm, lockup 30mm wide.
**Exclusion Zone:** Clear space on all sides equals the corner radius of the bubble.
**Dos and Don'ts:** Do not rotate, recolour, stretch, outline, or add drop shadows or glows; do not set the bare symbol on photography without the container; do not detach the torn edge from the bubble; do not re-letter the wordmark in a different face.

**Iconography:** UI icons only, distinct from the mark: 24px grid, 2px stroke, round caps and joins, outlined style, 2px corner radius. A custom set drawn to sit with the mark's rounded geometry; a named third-party set may be used only if redrawn to this grid.

## Design System

### Color Palette

Primary colour is a deep verified-green, chosen over the category-default blue to carry the "settled and receipted" idea while avoiding the brighter greens some incumbents use. The page surface is a warm off-white. Every token carries a light and a dark value in one row; there are no `-dark` sibling tokens.

#### Primary Colors — a 9-shade scale
| Token | Light | Dark | Role |
|---|---|---|---|
| `primary-900` | `#0A3829` | `#C6ECDE` | Darkest — text on primary tints |
| `primary-700` | `#0E5940` | `#8AD0B6` | Pressed backgrounds, high-emphasis text |
| `primary-500` | `#12805C` | `#3FA981` | **Base** — buttons, links, active states |
| `primary-300` | `#8FCDB2` | `#236E54` | Selected backgrounds, focus rings |
| `primary-100` | `#E3F2EC` | `#14382B` | Lightest — tinted backgrounds (alerts, selected rows) |

Generate all nine shades (100–900); the five above are the anchors. Hover and pressed states use adjacent shades (`primary-600`/`primary-700`); on touch surfaces the pressed state is the hover shade.

#### Neutral Colors — a 9-shade warm grey scale
| Token | Light | Dark | Role |
|---|---|---|---|
| `grey-900` | `#1B1C1A` | `#F4F5F1` | Darkest — primary text |
| `grey-600` | `#5B5F54` | `#A8AC9E` | Secondary text |
| `grey-300` | `#C6C9BE` | `#3F4238` | Borders, dividers |
| `grey-100` | `#EEEFE8` | `#202219` | Secondary surfaces, input backgrounds |
| `grey-50` | `#FAF8F4` | `#17181A` | Page background |

Greys are tinted warm toward the brand's off-white rather than left neutral. Structural roles keep their names and inherit the source shade's light and dark values as one token: `surface` = `grey-50`, `surface-secondary` = `grey-100`, `text-primary` = `grey-900`, `text-secondary` = `grey-600`, `border` = `grey-300`. `on-primary` is a role token for text and icons on a primary fill: `#FFFFFF` light, `#0A2E22` dark.

#### Accent & Semantic Colors — 5-shade scales
Generate a 5-shade scale (100/300/500/700/900) for each state and the accent. The 500 shade is the element colour, the 100 shade tints its background, the 900 shade is its text. `success` is a brighter green than `primary` so brand and state never read as the same signal.

| Token | Light | Dark | Role |
|---|---|---|---|
| `success-500` | `#16A34A` | `#4ADE80` | Settled payments, confirmations |
| `warning-500` | `#D97706` | `#FBBF24` | Warnings, pending and caution states |
| `error-500` | `#DC2626` | `#F87171` | Failed payments, errors, destructive actions |
| `info-500` | `#2563EB` | `#60A5FA` | Informational alerts and tips |
| `accent-500` | `#D9643A` | `#F0A088` | Highlighting new features, graph lines |

### Typography

| Role | Font | Fallback |
|---|---|---|
| Display / Headings | `Inter` | `system-ui, sans-serif` |
| Body | `Inter` | `system-ui, sans-serif` |
| Monospace | `JetBrains Mono` | `monospace` |

Monospace is included because references, meter numbers, and electricity tokens are read and copied character by character. `Inter` is self-hosted and subset to Latin; if it fails to load, the platform sans is used.

| Token | Size | Weight | Line Height | Usage |
|---|---|---|---|---|
| `display` | 40px / 2.5rem | 700 | 1.2 | Splash, hero headings |
| `h1` | 32px / 2rem | 700 | 1.3 | Page titles |
| `h2` | 24px / 1.5rem | 600 | 1.35 | Section headings |
| `h3` | 20px / 1.25rem | 600 | 1.4 | Card titles, subsections |
| `body-large` | 18px / 1.125rem | 400 | 1.5 | Lead paragraphs, featured text |
| `body` | 16px / 1rem | 400 | 1.5 | Body text, form labels |
| `body-small` | 14px / 0.875rem | 400 | 1.5 | Captions, helper text, timestamps |
| `caption` | 12px / 0.75rem | 500 | 1.4 | Labels, badges, overlines |

### Spacing

| Token | Value | Usage |
|---|---|---|
| `space-xs` | 4px | Tightest spacing — icon-to-label, inline badges |
| `space-sm` | 8px | Compact — between related items, input inner padding |
| `space-md` | 16px | Default — card padding, list item gaps, section gaps |
| `space-lg` | 24px | Generous — between sections, dialog padding |
| `space-xl` | 32px | Breathing room — page margins, hero padding |
| `space-2xl` | 48px | Major separation — between unrelated sections |
| `space-3xl` | 64px | Layout-level — above and below the fold |

### Sizing

| Token | Value | Usage |
|---|---|---|
| `radius-sm` | 4px | Checkboxes, tags, small inputs |
| `radius-md` | 8px | Buttons, inputs, cards |
| `radius-lg` | 12px | Modals, sheets, large cards |
| `radius-full` | 9999px | Pills, avatars, badges |
| `button-height` | 44px | Minimum touch target for buttons |
| `input-height` | 48px | Text inputs, selects |
| `icon-sm` | 16px | Inline icons, status indicators |
| `icon-md` | 24px | Navigation icons, button icons |
| `icon-lg` | 32px | Feature icons, empty state illustrations |

### Shadows

| Token | Light | Dark | Usage |
|---|---|---|---|
| `shadow-none` | `none` | `none` | Flat surfaces |
| `shadow-sm` | `0 1px 2px rgba(0,0,0,0.05)` | `0 1px 2px rgba(0,0,0,0.4)` | Subtle lift — cards on the base surface |
| `shadow-md` | `0 4px 6px rgba(0,0,0,0.07)` | `0 4px 8px rgba(0,0,0,0.5)` | Elevated — dropdowns, active cards |
| `shadow-lg` | `0 10px 25px rgba(0,0,0,0.1)` | `0 10px 30px rgba(0,0,0,0.65)` | Modals, drawers, sheets |
| `shadow-xl` | `0 24px 50px rgba(0,0,0,0.15)` | `0 24px 60px rgba(0,0,0,0.7)` | Full-screen overlays |

On dark surfaces elevation is carried by the shadow's dark value plus a subtle `border` where a shadow alone is not readable.

### Component Tokens

Buttons follow an action pyramid: one true primary action per surface, a few secondary, tertiary as links. A destructive action stays secondary until its confirmation step, where it becomes primary.

| Variant | Background | Text | Border | Radius | Height | Padding |
|---|---|---|---|---|---|---|
| Primary | `primary-500` | `on-primary` | none | `radius-md` | `button-height` | 16px 24px |
| Secondary | `primary-100` | `primary-700` | none | `radius-md` | `button-height` | 16px 24px |
| Outline | transparent | `primary-500` | 1.5px `primary-500` | `radius-md` | `button-height` | 16px 24px |
| Ghost | transparent | `text-secondary` | none | `radius-md` | `button-height` | 16px 24px |
| Destructive | `error-500` | `#FFFFFF` | none | `radius-md` | `button-height` | 16px 24px |

| Input state | Border | Background | Text |
|---|---|---|---|
| Default | `border` | `surface-secondary` | `text-primary` |
| Focus | `primary-500` | `surface` | `text-primary` |
| Error | `error-500` | `surface` | `text-primary` |
| Disabled | `border` | `surface-secondary` | `text-secondary` opacity 0.5 |

### Responsive & Device Targets

The visual interface is the mobile application. Send and receive channels (USSD/SMS, WhatsApp) are text surfaces owned by their providers and are not styled by this system.

| Device class | Width range | Layout posture |
|---|---|---|
| Mobile | < 600px | Single column, bottom navigation |
| Large mobile / tablet | 600–1024px | Adaptive one-to-two column, bottom navigation retained |

**Input modalities:** Touch primary; on-screen keyboard for text entry. No pointer, hover, or desktop targets.
**Breakpoints:** 600px and 1024px only.

### Internationalization & Localization

| Aspect | Decision |
|---|---|
| Launch locales | `en-NG` (Nigerian English) and `pcm` (Nigerian Pidgin) |
| Later locales | `ha-NG`, `yo-NG`, `ig-NG` (FEAT-016) |
| Direction | LTR only |
| Text expansion | Allow ~30–40% growth over English; Pidgin often runs longer than English for the same meaning |
| Formats | Dates, numbers, currency (NGN), and names via locale APIs — never hardcoded |
| Content | UI strings externalized; no concatenated sentences |
| Fallback | Fall back to English and show the English string rather than a blank |
| Script coverage | All launch and later locales use Latin script; the font must cover Yoruba and Igbo diacritics (ẹ, ọ, ṣ, ụ, ị) |

## Accessibility

**Target:** WCAG 2.2 Level AA

- **Contrast ratios:** All text meets WCAG AA minimum (4.5:1 normal, 3:1 large). Dark-mode values are checked independently of light mode.
- **Preferred accessible colour:** Dark text on a light tinted surface over white text on a dark colour; for secondary text on a coloured panel, rotate the hue toward a brighter colour rather than pushing toward white.
- **Touch targets:** Interactive elements are a minimum 44×44px.
- **Focus indicators:** Visible focus ring on all interactive elements (2px `primary-500` outline, 2px offset).
- **Colour independence:** Colour is never the only signal. Payment states pair `success`/`warning`/`error` with an icon and a text label, since colour-blind users must still read whether a payment settled.
- **Screen reader support:** All images have alt text, all inputs have labels, and dynamic content (payment status, receipt arrival) uses live regions.
- **Reduced motion:** Respect the OS reduce-motion setting. State animations degrade to a static result, and no meaning depends on motion.

## Asset Style Guide

### Brand Mark Direction
| Aspect | Direction |
|---|---|
| **Mark type** | Combination — pictorial speech-bubble symbol with a wordmark lockup |
| **Sharp ↔ Round** | Rounded bubble with one sharp accent — the torn receipt edge |
| **Solid ↔ Line** | Solid bubble; the torn edge is a sharp cut, not a drawn stroke |
| **Symmetry ↔ Asymmetry** | Near-symmetric bubble, deliberately asymmetric torn tail |
| **Flat ↔ Dimensional** | Flat, single-plane; no gradients or bevels |
| **Construction feel** | Grid-based geometric bubble with a hand-torn edge |
| **Graphic device** | Rounded-square container for busy or photographic backgrounds |
| **Required mark forms** | Master, one-colour, knockout, app icon, favicon, transparent; lockups horizontal, stacked, and icon-only |

### Iconography Direction
| Aspect | Direction |
|---|---|
| **Grid & stroke** | 24px grid, 2px stroke, round caps and joins |
| **Style** | Outlined, single weight |
| **Corner radius** | 2px |
| **Set** | Custom set drawn to the mark's rounded geometry; a third-party set only if redrawn to this grid |
| **States** | Default `text-secondary`; active `primary-500`; disabled `text-secondary` at reduced opacity |

### Illustration Direction
| Aspect | Direction |
|---|---|
| **Style** | Flat vector with geometric forms that echo the mark's rounded bubble and torn edge |
| **Line weight** | Hierarchy scaled to the artwork, not one fixed px: bold outline for the dominant form, medium for the main shapes, fine for detail; scales up with the canvas |
| **Color palette** | Duotone in `primary-500` and `grey-600` on a transparent ground, with `accent-500` as a single highlight |
| **Usage** | Empty states (no receipts, no discovery results) and the offline state only; never decorative |

### Motion Direction
State animation, not video. Applied to the three states that carry weight: a payment that succeeds, one that is pending, and one that fails.

| Aspect | Direction |
|---|---|
| **Trigger** | A payment state change: success, pending, or failure |
| **Pacing** | Calm and purposeful — success settles in about 400–600ms; pending loops gently; error appears without bounce or overshoot |
| **Loop character** | Success and error are one-shot; pending loops seamlessly until the state resolves |
| **Grade** | Brand colours only; no gradients, no particles, no confetti |
| **Audio** | None |
| **Format** | Lightweight vector or runtime animation; no video and no heavy animation bundles, to protect the cold-start and memory budgets for a 1 to 2 GB device (NFR-001) |
| **Reduced motion** | Degrades to the static result; the outcome is never conveyed by motion alone |

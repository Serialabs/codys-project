# Feature Inventory – KNO Post‑Purchase Surveys (MVP)

Features marked **V2** are out of scope for the initial build and included only for reference.

---

## Survey Creation

| Feature | Description | MVP / V2 |
|---|---|---|
| Template‑based creation | Start from 30+ pre‑built templates or blank. Add, remove, and reorder questions. | **MVP** |
| Basic question types | Radio, checkbox, short text, long text. | **MVP** |
| Advanced question types | Dropdown, slider, NPS, date picker, ranking, email/phone capture, image/video upload. | V2 |
| Follow‑up logic | Conditional branching on the follow‑up question. Select parent question and optionally trigger only on specific answer choices. Can also end the survey early or jump to the confirmation screen. | **MVP** |
| "Other" option toggle | Adds a text input to multiple‑choice questions. | V2 |
| Multiple surveys per store | Starter plan: 1 survey. Analyst+: multiple. | V2 |
| Survey styling | Global theme settings, per‑survey overrides, light/dark theme, colour picker, navigation text, custom CSS. | V2 |
| Promo code action | Generates single‑use Shopify discount codes. Displays on survey completion with auto‑apply button. | **MVP** (single‑use only; multi‑use and fallback in V2) |
| Other actions (review, referral, social, VIP) | Conditionally shown post‑survey CTAs. | V2 |

---

## Targeting & Timing

| Feature | Description | MVP / V2 |
|---|---|---|
| Audience builder | Define rules using customer/order/survey data. MVP supports lifetime order count and product ID. | **MVP** |
| Random survey selection | When multiple surveys are eligible, randomly pick one and prevent repeat views. | V2 |
| Display priority logic | Mutually exclusive, sequential, or round‑robin survey prioritisation. | V2 |
| Thank You page display | Survey appears immediately after checkout via Checkout UI extension. | **MVP** |
| Order Status page display | Survey shown when customers return to check order status. | V2 |
| Link and QR delivery | Unique survey URL and QR code; optional forced email capture; pre‑fill email via query string. | V2 |

---

## Placement Surfaces

| Surface | MVP / V2 |
|---|---|
| Shopify checkout extension (Thank You page) | **MVP** |
| Order Status page | V2 |
| Standalone link / QR code | V2 |
| Email embed | V2 |

---

## Data Capture & Linkage

| Feature | Description | MVP / V2 |
|---|---|---|
| Order linkage | Retrieve order ID via `OrderConfirmationApi` and associate with each response. | **MVP** |
| Customer linkage | Capture customer ID if logged in. Email capture for link surveys. | In‑checkout customer ID: **MVP**; link survey email capture: V2 |
| Survey metadata | Store survey ID, timestamp, question answers per response. | **MVP** |
| UTM / source tracking | Audience rules using UTM parameters. | V2 |

---

## Reporting & Analytics

| Feature | Description | MVP / V2 |
|---|---|---|
| Survey stats dashboard | Views, partial completions, completions, answer distribution per question. | **MVP** |
| CSV export | Export responses with order ID, customer ID, and answers. | **MVP** |
| Order Attribution report | Revenue and AOV per answer choice; "Estimate 100%" projection toggle. | V2 |
| Benchmarking & automated insights | Demographic/psychographic aggregates benchmarked across brands. | V2 |
| Advanced filters & segmentation | Filter by audience, product, location, etc. | V2 (MVP: date filter only) |

---

## Integrations

| Integration | MVP / V2 |
|---|---|
| Klaviyo — push survey status and question responses as custom properties | **MVP** |
| Shopify Flow — trigger workflows on survey completion | V2 |
| Triple Whale, Peel Insights, Google Analytics, TikTok | V2 |
| ReCharge subscription sync | V2 |
| Custom webhooks / API | V2 |

---

## Admin & Management

| Feature | MVP / V2 |
|---|---|
| Basic validation (required fields, plan limits) | **MVP** |
| Team roles and permissions | V2 |
| Plan management via Shopify Billing | V2 |
| Duplicate suppression (don't show same survey twice) | V2 |
| Multi‑language support | V2 |

---

## Known Constraints

- **Checkout UI extension limitations**: No custom CSS/HTML injection, no video questions, no slider questions, no dynamic confirmation screens.
- **Plan caps**: Starter — 1 survey, 3 questions, basic question types only.
- **Response bias**: Opt‑in surveys won't reach 100% of customers; the Order Attribution "Estimate 100%" feature compensates but relies on assumptions.
- **Privacy**: Must comply with GDPR/CCPA. Anonymous survey mode available for link surveys.

# Product Requirements Document – Post‑Purchase Survey App

## Problem Statement

DTC merchants on Shopify struggle to understand which marketing channels and messages drive sales. Cookie-based and last‑click attribution misses the full customer journey. Merchants need a lightweight way to ask customers immediately after purchase how they heard about the brand and what motivated the purchase — and to connect those answers to order data and automate follow‑up incentives. Without this, they rely on guesswork, leading to inefficient marketing spend.

## Target Users

**DTC brand owner ("Taylor")** — Runs a small lifestyle brand. Manages marketing, product, and fulfilment. Wants to know which influencer collaborations and ad campaigns drive sales to allocate a limited budget wisely.

**E‑commerce marketer ("Jordan")** — Works at a mid‑sized Shopify store. Responsible for growth and retention. Needs to measure impact per channel and trigger personalised Klaviyo flows based on customer feedback.

**Agency analyst ("Sam")** — Advises multiple Shopify clients. Wants a standardised zero‑party data capture method, benchmarking, and easy CSV exports. Prefers a turnkey solution.

## Jobs‑to‑be‑Done

- "When I launch campaigns, I want to ask customers how they found us and why they bought, so I can verify attribution assumptions."
- "When I need to increase repeat purchases, I want to offer a discount code immediately after the survey."
- "When analysing performance, I want to link survey responses to orders and revenue."
- "When integrating with my ESP, I want answers to sync automatically without manual exports."

---

## Functional Requirements

### Survey Builder
1. Merchants can create a survey using a template or from scratch.
2. Surveys support radio (single choice), checkbox (multi‑select), and short/long text inputs.
3. Follow‑up logic is configurable per question: show conditional follow‑ups, define exit points, or end the survey early.
4. Optional promo code action: generates one‑time discount codes and displays them on completion.
5. Builder enforces plan limits on question count with inline validation and upgrade prompts.

### Placement and Delivery
1. Survey renders as a Checkout UI extension on the Shopify Thank You page.
2. Extension fetches the order ID via `OrderConfirmationApi` to link responses.
3. MVP supports one active survey. Multi‑survey rotation and link/QR delivery are V2.

### Targeting
1. Merchants define a simple audience: new customers (lifetime order count = 1) vs. returning (count > 1), with optional product ID filter.
2. Survey only displays to customers matching the audience; others see nothing.

### Data Capture
1. Per response, store: shop ID, survey ID, order ID, customer ID (if available), timestamp, and answers to each question.
2. Associate responses with orders via order ID for later revenue analysis.
3. Store any generated promo code for redemption auditing.

### Reporting & Integrations
1. Dashboard shows survey views, partial completions, completed submissions, and per‑question answer distribution.
2. CSV export of responses with order and customer identifiers.
3. Klaviyo integration: per‑question toggle to sync answers as custom properties; push survey status events (view, partial, completed).
4. V2: Order Attribution report (revenue/AOV per answer), Shopify Flow triggers.

### Admin & Platform
1. Admin UI uses Shopify App Bridge and Polaris components.
2. Authentication via Shopify OAuth with offline access.
3. Plan management via Shopify Billing API.
4. Checkout extension deployed via Checkout UI Extension API; must respect its limitations (no HTML/CSS injection, no video or slider questions).
5. Backend complies with Shopify rate limits; webhooks verified via HMAC.

---

## Non‑Functional Requirements

- **Performance** — Survey widget loads within 500ms after Thank You page renders; answer submissions complete in under 200ms.
- **Reliability** — 99.9% uptime; API failures handled with exponential backoff retry.
- **Security & Compliance** — HTTPS for all communications, sensitive data encrypted at rest, GDPR/CCPA compliance (data deletion, anonymity options).
- **Scalability** — Support up to 10k survey responses per day; DB and queues scale horizontally.
- **Accessibility** — WCAG 2.1 AA; keyboard navigation and screen reader labels required.

---

## Success Metrics

- **Survey completion rate** — % of customers who complete after seeing it. Target ≥ 50%.
- **Response attribution coverage** — % of orders linked to at least one survey answer. Target ≥ 30%.
- **Time to first insight** — From installation to first actionable insight. Goal: within 1 day.
- **Promo code redemption rate** — % of generated codes that are redeemed.
- **Integration usage** — Merchants enabling Klaviyo and events successfully pushed.

---

## Constraints

- Shopify checkout extensibility: cannot inject HTML/CSS, use video questions, or use slider questions inside checkout.
- Starter plan: 1 survey, 3 questions, basic question types only.
- GDPR/CCPA compliance required; no sensitive personal data collection without consent.
- MVP to be built by a solo developer within two weeks.

---

## Out of Scope (V2)

- Advanced question types (dropdown, slider, NPS, date picker, ranking, image/video)
- Multi‑survey rotation and display priority logic
- Order Status page and link/QR delivery
- Benchmarks and automated insights
- Shopify Flow and third‑party analytics integrations (Triple Whale, Peel, etc.)
- Multi‑user roles and team management

---

## Risks & Mitigations

| Risk | Mitigation |
|---|---|
| Low response rate | Keep survey short (1–3 questions), position at top of page, include promo incentive. |
| Performance impact on checkout | Load survey asynchronously after page renders; minimise script size. |
| Klaviyo dependency | Robust integration for MVP; document that only new responses sync (historical data requires manual import). |
| Plan limit confusion | Show remaining question/survey count in the builder; prompt upgrade on limit hit. |
| Privacy concerns | Email capture is optional; include anonymity mode and a privacy notice. |
| Shopify checkout API changes | Monitor Shopify developer updates; use feature flags to disable unsupported components. |

---

## MVP Acceptance Criteria

- ✅ Merchant can install the app and access a dashboard.
- ✅ Merchant can create one survey with up to 3 questions (radio, checkbox, text).
- ✅ Merchant can define a basic audience (new vs. returning customers) and assign it to the survey.
- ✅ Merchant can add a promo code action that generates a single‑use discount code shown after completion.
- ✅ Merchant can add the survey block to the Thank You page via Checkout editor.
- ✅ Customers matching the audience see the survey, answer questions, and receive a promo code.
- ✅ Responses saved with order ID and customer ID (if available).
- ✅ Merchant sees summary metrics and can export CSV.
- ✅ Merchant can enable Klaviyo integration and push responses and status as custom properties.
- ✅ System meets performance and security requirements.

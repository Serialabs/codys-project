# Build Plan – Post‑Purchase Survey App

10‑day schedule for a solo developer. Adjust based on pace and complexity.

---

## Day 1 — Project Setup & Planning
- Scaffold Shopify Remix app using Shopify CLI.
- Init Git repo; configure ESLint/Prettier; install Polaris and Prisma.
- Create a project board with tasks from this plan.
- Review Checkout UI extension docs and set up a development store.

## Day 2 — Database & Models
- Define the Prisma schema: shops, surveys, questions, choices, audiences, responses, promo codes.
- Run `prisma migrate dev` to create the local DB.
- Write seed scripts for sample surveys and questions to speed up UI dev.

## Day 3 — OAuth & Install Flow
- Implement Shopify OAuth via `@shopify/shopify-api`.
- Store access tokens in `shops` table; handle app install/uninstall webhooks.
- Test installation on the dev store; verify redirect to dashboard.

## Day 4 — Admin UI Scaffolding
- Build dashboard layout with Polaris.
- Add navigation: Surveys, Audiences, Analytics, Integrations.
- Implement survey list view and a **Create survey** button.

## Day 5 — Survey Builder
- Build survey creation/editing page: title input, Add Question modal (radio/checkbox/text), reorderable question list.
- Implement follow‑up logic UI on follow‑up questions (select parent question and trigger answer).
- Implement promo code action config (discount value, expiry).
- Client‑side validation for required fields and plan limits.
- Save surveys to DB via API routes.

## Day 6 — Audience Builder & Targeting
- Create Audiences page: form for attribute (lifetime order count, product ID), operator, and value.
- Save audience rules to DB; associate with surveys.
- Allow selecting one audience in survey settings.
- Basic preview logic to estimate matching customers.

## Day 7 — Checkout UI Extension
- Create survey widget as a Checkout UI extension with React.
- Fetch active survey and audience rules from backend on Thank You page load.
- Retrieve order ID via `OrderConfirmationApi`.
- Render questions, handle navigation, collect answers.
- Generate a client‑side response ID (UUID) and send events (`survey_impression`, `survey_started`, `question_answered`, `survey_submitted`) to backend.

## Day 8 — Backend Endpoints & Discount Codes
- Implement endpoints: get active survey (by shop and audience), save response and answers, generate promo code.
- Secure endpoints with HMAC verification.
- Integrate with Shopify Discounts API to create single‑use codes.
- Queue code generation jobs via BullMQ with basic retry logic.

## Day 9 — Analytics Dashboard & CSV Export
- Build analytics page: total views, starts, completions; bar chart per question's answer distribution.
- API route to aggregate events and responses from DB.
- **CSV export** button that streams responses (order ID, customer ID, answers).
- Pagination or streaming for large datasets.

## Day 10 — Klaviyo Integration & Polish
- Integrations settings page: Klaviyo API key input, per‑question sync toggle.
- Worker that listens for new responses and pushes selected properties to Klaviyo; handles success/failure events.
- Unit tests (survey creation, audience matching, discount code generation) with Jest.
- End‑to‑end test for the checkout extension with Playwright.
- Manual QA on desktop and mobile.
- Deploy to production.

---

## Testing Plan

- **Unit tests** — Data model functions, audience matching logic, discount code generation (Jest).
- **Integration tests** — API endpoints for survey fetch and response submit (Supertest).
- **End‑to‑end tests** — Simulate a customer placing an order and completing a survey; verify DB records (Playwright).
- **Manual tests** — Cover guest vs. logged‑in, new vs. returning customer scenarios.
- **Performance tests** — Concurrent survey submissions via k6 to validate API and queue throughput.

---

## Definition of Done

- All automated tests pass; manual QA complete.
- Merchant can install the app, create and publish a survey (up to 3 questions) with an optional promo code action.
- Survey appears on the Thank You page for matching customers; responses stored in DB.
- Analytics dashboard shows views, starts, completions, and answer distribution; CSV export works.
- Klaviyo integration syncs responses and status events.
- App deployed to production with secure environment variables and webhooks registered.
- All performance, security, and accessibility requirements met.
- V2 features and open questions documented for future iterations.

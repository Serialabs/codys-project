# System Design – Post‑Purchase Survey App

---

## Components

### 1. Shopify Admin App (Merchant Dashboard)
- Built with Shopify's Remix app template and **Polaris** UI components.
- Interfaces for survey creation, audience management, analytics, and integrations.
- Communicates with the backend via authenticated REST or GraphQL calls.
- Uses **App Bridge** for Shopify admin embedding and OAuth.

### 2. Checkout UI Extension (Survey Widget)
- Lightweight script running on the Thank You (Order Confirmation) page.
- Calls `OrderConfirmationApi` to obtain the order ID.
- Fetches survey config and audience rules from the backend.
- Renders questions, captures answers, and sends events to the backend.
- Displays the promo code action and auto‑applies the discount via URL redirect.
- Must comply with UI extension limitations: no custom HTML/CSS, no video or slider questions.

### 3. Backend API
- Stateless Node.js server (Express or Remix API routes).
- CRUD for surveys, questions, audiences, and actions.
- Endpoints for the checkout extension to: fetch the active survey by audience, and record responses.
- Rate limiting per store and per IP.
- Promo code generation via the Shopify Discounts API.
- Asynchronous integration jobs (Klaviyo sync) dispatched via queue.

### 4. Database
- **PostgreSQL** with Prisma ORM.
- Stores shops, surveys, questions, audiences, responses, and promo codes.
- Connection pooling; data isolated per shop.
- Soft deletion for surveys to retain historical response data.

### 5. Queue & Worker
- **BullMQ** on Redis for asynchronous tasks:
  - Unique discount code generation
  - Klaviyo event pushing
  - Analytics aggregation
- Workers run separately from the API.
- Exponential backoff retry on failure.

### 6. Analytics / Event Pipeline
- Checkout extension batches client events and sends them to the backend.
- Backend logs events to an `events` table and optionally forwards to PostHog or Segment.
- Nightly jobs compute aggregates (views, completion rate, distribution) into an `analytics_summary` table for the reporting dashboard.
- V2: Order Attribution joins responses with Shopify order data via order ID.

### 7. Integrations
- **Klaviyo** — Response recorded → job queued → event and selected answers pushed to Klaviyo.
- **Shopify Flow (V2)** — App registers a Flow trigger and emits events on survey completion.
- **Other analytics tools (V2)** — Webhooks or exports to Triple Whale, Peel, etc.

---

## Data Flow

```
Customer checkout → Thank You page → Survey extension
  ├─ Fetches survey & audience rules from backend
  ├─ Retrieves order ID via OrderConfirmationApi
  ├─ Renders questions & collects answers
  └─ Sends events: impression, answer_selected, survey_submitted
                          │
                          ▼
                    Backend API
                          │
          ┌───────────────┴────────────────┐
          │                                │
      Save to DB                     Enqueue jobs
          │                                │
    Responses table            Promo code generation
                                 Klaviyo sync
                                 Analytics aggregation
```

---

## Security Model

- **Authentication** — Shopify OAuth (offline tokens) for admin calls. Checkout extension authenticates via signed requests containing shop domain and signature.
- **Authorisation** — Each API request is scoped to a shop; row‑level isolation prevents cross‑shop data access.
- **Webhook verification** — All incoming webhooks validated using HMAC signatures.
- **Data encryption** — Sensitive fields (emails, promo codes) encrypted at rest.
- **Permissions** — Extension requests `read_order` only; no access to payment data.

---

## Rate Limiting & Retry

- Per‑shop request limits (e.g., 10 req/sec) to stay within Shopify's API limits. Exceeding returns HTTP 429 with a `retry-after` header.
- Queue uses exponential backoff (5s → 30s → 2 min) with a max retry count for discount code creation and Klaviyo calls. Failures logged and surfaced in the admin dashboard.
- Idempotent keys used for promo code generation and response submission to prevent duplicates on page refresh.

---

## Deployment

- Backend and worker on Fly.io, Railway, or Render. Multiple regions for latency.
- Secrets (Shopify API keys, DB URL, Klaviyo key) managed via the hosting platform's secrets manager.
- CI/CD via GitHub Actions: lint → test → deploy on merge to main.
- Error rates and performance monitored via logs and metrics dashboards.

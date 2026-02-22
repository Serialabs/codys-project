# Tech Stack – Post‑Purchase Survey App

---

## Application Framework

- **Backend & Frontend** — Shopify's **Remix**‑based app template with TypeScript. Provides SSR, built‑in routing, and App Bridge integration. Next.js is a viable alternative.
- **Admin UI** — **Polaris** React components for a native Shopify admin look and feel. Handles accessibility and responsive design.
- **State management** — React context or Zustand. Avoid heavy frameworks.
- **Checkout extension** — Built with the **Checkout UI Extensions API** and React (`@shopify/checkout-ui-extensions-react`). Packaged in `/extensions` and registered in `extension.config.toml`.

---

## Data Layer

- **Database** — **PostgreSQL** hosted on Supabase, Railway, or Fly.io.
- **ORM** — **Prisma** for schema modelling, migrations, and type safety.
- **Redis** — Managed via Upstash or Fly.io. Backs **BullMQ** queues for async tasks (promo code generation, Klaviyo sync).
- **Caching** — Redis or in‑memory cache for frequently accessed survey configs.

---

## Hosting & Deployment

- **Fly.io or Railway** — Simple Node.js deployment with built‑in Postgres and Redis. Fly.io preferred for global edge regions.
- **CI/CD** — GitHub Actions: lint → test → deploy on merge to main.
- **Environment variables:**

| Variable | Purpose |
|---|---|
| `SHOPIFY_API_KEY` / `SHOPIFY_API_SECRET` | OAuth credentials |
| `SCOPES` | Required Shopify scopes (`read_orders`, `read_customers`, `write_discounts`) |
| `DATABASE_URL` | Postgres connection string |
| `REDIS_URL` | Redis connection |
| `KLAVIYO_API_KEY` | Klaviyo integration |
| `APP_URL` | Public URL for Shopify callbacks and webhooks |
| `JWT_SECRET` | Internal API auth (if used) |

---

## Queue & Background Processing

- **BullMQ** — Redis‑backed job queue with retries and rate limiting.
- Workers run separately from the main API server.
- Alternative: Google Cloud Tasks or AWS SQS if already on those platforms.

---

## Integrations & SDKs

| Tool | Purpose |
|---|---|
| `@shopify/shopify-api` | OAuth, REST/GraphQL calls, webhook registration |
| `@shopify/ui-extensions-react` | Checkout extension components and hooks |
| `@klaviyo/node` (or plain HTTP) | Push events and properties to Klaviyo |
| PostHog (Node SDK) or Segment | Server‑side event capture |
| Jest + React Testing Library | Unit tests |
| Playwright | End‑to‑end tests on the checkout extension and admin UI |

---

## Local Development Workflow

1. `npm install`
2. `shopify app dev` — starts a dev server and connects a test store.
3. Set env vars in `.env.development`.
4. `prisma migrate dev` — create the DB schema (local Postgres via Docker).
5. `npm run dev` — starts API and extension; Shopify CLI proxies via ngrok.
6. Place a test order on the dev store to verify the survey appears.
7. `npm test` — run automated tests.
8. `shopify app deploy` — deploy to production.

---

## Deployment Checklist

- [ ] Set environment variables in hosting platform
- [ ] Provision Postgres and Redis instances
- [ ] Run database migrations
- [ ] Register webhooks (app/uninstall, billing, discount code updates)
- [ ] Deploy admin app and checkout extension; update extension config with production URL
- [ ] Verify OAuth callback and app installation
- [ ] Test survey on Thank You page in a development store
- [ ] Enable domain whitelisting for the extension
- [ ] Monitor logs and set up alerts for integration errors

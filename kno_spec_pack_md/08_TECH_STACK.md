# Recommended Tech Stack – Post‑Purchase Survey App

This section recommends technologies and tooling to implement the survey app efficiently while aligning with Shopify’s ecosystem.  The stack is chosen for developer productivity, scalability and cost‑effectiveness.

## Application framework

* **Backend & frontend**: Use Shopify’s **Remix**‑based app template (official in 2025) with **TypeScript**. Remix provides server‑side rendering, built‑in routing and easy integration with App Bridge.  If Remix is not an option, **Next.js** is an alternative.  
* **Admin UI**: Adopt **Polaris** React components for consistent Shopify admin look and feel. Polaris handles accessibility and responsive design out of the box.  
* **State management**: Use React context or a lightweight library like Zustand; avoid heavy frameworks.  
* **Checkout extension**: Build the survey widget using **Checkout UI Extensions API** and React. It must be packaged in the app’s `/extensions` folder and registered in `extension.config.toml`.  Use the provided `@shopify/checkout-ui-extensions-react` library for components and state hooks.

## Data layer

* **Database**: **PostgreSQL** hosted on **Supabase**, **Railway** or **Fly.io**. Postgres offers strong consistency and support for JSON fields when needed.  
* **ORM**: Use **Prisma** to model entities (shops, surveys, questions, audiences, responses). Prisma’s migration and type safety accelerate development and align with the data model defined earlier.  
* **Redis**: Deploy **Redis** (managed via Upstash or Fly.io) to back **BullMQ** queues for asynchronous tasks (promo code generation, Klaviyo sync).  
* **Caching**: Leverage Redis or in‑memory caching for frequently accessed survey configurations to reduce database reads.

## Hosting & deployment

* **Fly.io or Railway**: Both platforms offer simple deployment of Node.js apps with built‑in Postgres and Redis add‑ons. Choose Fly.io for global edge regions and built‑in load balancing.  
* **GitHub Actions**: Set up CI/CD to lint, test and deploy the app on pushes to the main branch.  
* **Environment management**: Use `.env` files locally and environment variables in hosting platform. Key variables include:  
  - `SHOPIFY_API_KEY` and `SHOPIFY_API_SECRET` – OAuth credentials.  
  - `SCOPES` – Required scopes (e.g., `read_orders`, `read_customers`, `write_discounts`).  
  - `DATABASE_URL` – Postgres connection string.  
  - `REDIS_URL` – Redis connection.  
  - `KLAVIYO_API_KEY` – For integration.  
  - `APP_URL` – Public URL for Shopify callbacks and webhook endpoints.  
  - `JWT_SECRET` – For internal API authentication (if used).

## Queue and background processing

* **BullMQ**: A lightweight Node.js library that uses Redis to manage job queues with retries and rate limiting. Jobs such as generating promo codes, sending events to Klaviyo and aggregating analytics run in workers separate from the main API server.  
* **Alternative**: Use cloud queues like Google Cloud Tasks or AWS SQS if hosted on those platforms, but BullMQ keeps infrastructure simple.

## Integrations & SDKs

* **Shopify API**: Use the official `@shopify/shopify-api` package for OAuth, REST/GraphQL calls and webhook registration.  
* **Checkout UI extensions**: Use `@shopify/ui-extensions-react` for building the extension and connect it to the backend via fetch calls.  
* **Klaviyo API**: Use `@klaviyo/node` or simple HTTP requests to push events and properties; wrap calls in a worker with error handling【959380932338212†L109-L178】.  
* **Analytics**: Integrate **PostHog** via the Node SDK to capture events server‑side; or use **Segment** to forward events to multiple destinations.  
* **Testing**: Use **Jest** and **React Testing Library** for unit tests; **Playwright** for end‑to‑end tests on the checkout extension and admin UI.

## Local development workflow

1. Clone the repository and run `npm install`.  
2. Use the Shopify CLI (`shopify app dev`) to start a development server and generate a test store for local testing.  
3. Set environment variables in `.env.development`.  
4. Run `prisma migrate dev` to create the database schema in a local Postgres instance (e.g., using Docker).  
5. Launch the API and extension with `npm run dev`. The Shopify CLI proxies requests to the local extension via ngrok.  
6. Use a development store to place test orders and verify the survey appears.  
7. Run `npm test` to execute automated tests.  
8. Use `shopify app deploy` to deploy to production when ready.

## Deployment checklist

* [ ] Set up environment variables in the hosting platform (API keys, DB URL, Klaviyo key).  
* [ ] Provision Postgres and Redis instances.  
* [ ] Run database migrations.  
* [ ] Register webhooks for app/uninstall, billing and discount code updates.  
* [ ] Deploy the admin app and checkout extension; update extension config with the production app URL.  
* [ ] Verify OAuth callback and check that the app installs correctly.  
* [ ] Test survey display on the Thank You page in production using a development store.  
* [ ] Enable domain whitelisting for the extension.  
* [ ] Monitor logs and metrics for errors; set up alerts on failed integrations.
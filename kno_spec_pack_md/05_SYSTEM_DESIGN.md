# System Design – Post‑Purchase Survey App

This section outlines the high‑level architecture for building a Shopify post‑purchase survey app.  The architecture is designed for scalability, security and compliance with Shopify’s extensibility model.

## Components

1. **Shopify Admin App (Merchant Dashboard)**  
   * Built using Shopify’s official app template (Remix or Next.js) with **Polaris** UI components.  
   * Provides interfaces for survey creation, audience management, analytics and integrations.  
   * Communicates with the backend via authenticated REST or GraphQL calls.  
   * Uses **App Bridge** to embed within Shopify admin and handle OAuth.

2. **Checkout UI Extension (Survey Widget)**  
   * A lightweight script packaged as a **Checkout UI extension** that runs on the Thank You (Order confirmation) page.  
   * Calls Shopify’s `OrderConfirmationApi` to obtain the order ID【956241798895964†L160-L169】.  
   * Fetches the survey configuration and audience rules from the backend.  
   * Renders questions, captures answers and sends events back to the backend.  
   * Displays the promo code action and automatically applies the discount via URL if configured【295004260503847†L160-L177】.  
   * Must comply with UI extension limitations (no custom HTML/CSS, no video/slider)【487850571088937†L315-L333】.

3. **Backend API**  
   * Stateless Node.js server or serverless functions (e.g., with Express/Remix API routes).  
   * Handles CRUD operations for surveys, questions, audiences and actions.  
   * Exposes endpoints for the checkout extension to fetch active survey by audience and record responses.  
   * Implements rate limiting (per store and per IP) to comply with Shopify’s API limits.  
   * Provides endpoints to generate promo codes via the Shopify Discounts API.  
   * Processes integrations (e.g., pushes to Klaviyo) asynchronously via a queue.

4. **Database**  
   * Use **PostgreSQL** with an ORM (Prisma) to store shops, users, surveys, questions, audiences, responses and promo codes.  
   * Key tables and indexes are defined in the data model (see next file).  
   * Employ connection pooling and isolation per shop to ensure data separation.  
   * Implement soft deletion for surveys to retain historical data.

5. **Queue & Worker**  
   * Use a job queue (e.g., **BullMQ** on Redis) to handle asynchronous tasks such as:  
     * Generating unique discount codes.  
     * Pushing events to Klaviyo (or other ESPs).  
     * Aggregating analytics and computing daily metrics.  
   * Workers run separately from the API to avoid blocking requests.  
   * Configure retry logic with exponential backoff in case of temporary failures.

6. **Analytics/Event Pipeline**  
   * Capture client events (impression, answer selection, submission) in the checkout extension and send them to the backend.  
   * On the backend, publish events to a stream (e.g., Kafka, PostHog or simple database table) for downstream analytics.  
   * Computed metrics (views, completion rate, distribution) can be served from a pre‑aggregated table or materialised view.  
   * For Order Attribution (V2), join responses with Shopify order data via order ID and compute revenue metrics.【954105040073377†L83-L100】

7. **Integrations**  
   * **Klaviyo integration:** When a survey response is recorded, a job on the queue pushes the event and selected answers to Klaviyo via their API【959380932338212†L109-L178】.  
   * **Shopify Flow (V2):** The app registers a Flow trigger and emits events when a response meets certain criteria【15588629035657†L36-L63】.  
   * **Other analytics/BI tools (V2):** Provide webhooks or exports to Triple Whale, Peel, etc.【978605946208732†L121-L123】.

## Data flow

```text
 Customer checkout -> Thank You page -> Survey extension
    |- calls backend to fetch survey & audience rules
    |- collects order ID via OrderConfirmationApi
    |- renders questions & collects answers
    |- sends events: impression, answer_selected, survey_submitted
                            |
                            v
                      Backend API
                            |
           +--------------+--------------+
           |                             |
       Save to DB                    Enqueue jobs
           |                             |
     Responses table             Promo code generation
                                  Klaviyo sync
                                  Analytics aggregation
```

## Security model

* **Authentication:** Use Shopify OAuth (offline tokens) for authenticated admin calls.  The checkout extension authenticates via signed requests containing the shop domain and signature.  
* **Authorization:** Each API request identifies the shop and ensures isolation of data; row‑level security prevents cross‑shop access.  
* **Webhook verification:** All incoming webhooks (e.g., app uninstall, billing events) are validated using HMAC signatures.  
* **Data encryption:** Sensitive fields (emails, promo codes) are encrypted at rest.  
* **Permissions:** The checkout extension requests only the `read_order` permission to access order IDs and does not access payment data.

## Rate limiting & retry strategy

* **API rate limiting:** Implement per‑shop request limits (e.g., 10 requests/sec) to prevent abuse and to stay within Shopify’s call limits.  Exceeding the limit returns HTTP 429 with a retry‑after header.  
* **Retry strategy:** For tasks like discount code creation or Klaviyo calls, the queue uses exponential backoff (e.g., 5s, 30s, 2 min) and a maximum retry count.  Failures are logged and surfaced in the admin dashboard.  
* **Idempotency:** Use idempotent keys for promo code generation and response submission to avoid duplicates if the customer refreshes the page.

## Deployment considerations

* Deploy the backend and worker on a cloud hosting provider (Fly.io, Railway or Render).  Use multiple regions close to the majority of customers.  
* Store environment secrets (Shopify API keys, database URL, Klaviyo keys) securely with the hosting platform’s secrets manager.  
* Use continuous deployment with automated tests to ensure stability.  
* Monitor performance and error rates via logs and metrics dashboards.
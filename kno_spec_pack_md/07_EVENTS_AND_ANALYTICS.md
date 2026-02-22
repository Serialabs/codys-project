# Events and Analytics

To understand user behaviour and power reporting, the app must emit both **client‑side** and **server‑side** events.  Each event has a defined payload and is captured in a central analytics system (e.g., PostHog, Segment, or a custom event table).  The table below describes the core events and their attributes.

## Client events (checkout extension)

| Event name | Trigger | Payload fields | Purpose |
| --- | --- | --- | --- |
| `survey_impression` | The survey widget is rendered on the Thank You page. | `shop_id`, `survey_id`, `order_id`, `customer_id?`, `timestamp` | Count how many customers were eligible to see the survey and calculate view vs completion rates. |
| `survey_started` | Customer interacts with the survey (e.g., first answer selected or started typing). | `shop_id`, `survey_id`, `response_id` (generated client‑side UUID), `order_id`, `customer_id?`, `timestamp` | Distinguish impressions from active respondents. |
| `question_answered` | Customer answers a question (single or multi choice) or inputs text. | `response_id`, `question_id`, `answer_value(s)`, `timestamp`, `position` | Enables granular analysis of drop‑offs and answer distribution. |
| `survey_submitted` | Customer completes the last question and reaches the confirmation screen. | `response_id`, `survey_id`, `completed_at` | Records completion and triggers further processing (promo code, integrations). |
| `promo_code_clicked` | Customer clicks the “Apply discount now” button on the promo action. | `response_id`, `promo_code_id`, `timestamp` | Track redemption intent and success/failure. |

## Server events (backend and integrations)

| Event name | Trigger | Payload fields | Purpose |
| --- | --- | --- | --- |
| `response_saved` | The backend receives a completed response and writes it to the database. | `response_id`, `survey_id`, `shop_id`, `order_id`, `customer_id?`, `completed_at` | Confirmation that data persistence succeeded. |
| `response_linked_to_order` | Response is matched to a Shopify order (initially via order ID; later could include revenue data). | `response_id`, `order_id`, `total_price`, `currency`, `timestamp` | Foundation for Order Attribution reports. |
| `promo_code_generated` | A discount code is created via Shopify API for a response. | `promo_code_id`, `shop_id`, `survey_id`, `code`, `expires_at` | Auditing and rate limiting of code generation. |
| `promo_code_redeemed` | Shopify triggers a webhook when the code is used (requires additional webhook subscription). | `promo_code_id`, `order_id`, `redeemed_at` | Measure effectiveness of incentives. |
| `klaviyo_sync_succeeded` | The job worker successfully pushes the response and answers to Klaviyo. | `response_id`, `survey_id`, `klaviyo_profile_id`, `timestamp` | Monitor integration reliability【959380932338212†L109-L178】. |
| `klaviyo_sync_failed` | The job worker fails to push data to Klaviyo after retries. | `response_id`, `error_message`, `timestamp` | Surface integration errors to the merchant. |
| `survey_created` | Merchant creates a new survey in the admin dashboard. | `survey_id`, `shop_id`, `created_at` | Admin analytics for usage trends. |
| `survey_published` | Survey status changes from draft to published. | `survey_id`, `shop_id`, `published_at` | Track go‑live events. |
| `audience_modified` | Merchant updates an audience rule. | `audience_id`, `survey_id`, `shop_id`, `changes`, `timestamp` | Audit and debug targeting issues. |

## Event collection & processing

* **Instrumentation:** Use a thin client inside the checkout extension to batch events and send them via fetch/XHR to the backend. The backend logs events into a `events` table and optionally forwards them to PostHog or Segment.  
* **Privacy:** Do not capture personally identifiable information beyond what is necessary (order ID, customer ID). If email address is captured via link surveys, treat it as sensitive data.  
* **Sampling:** For high‑traffic stores, implement sampling or rate limiting on `question_answered` events to reduce volume.  
* **Aggregation:** Nightly jobs compute aggregates (views, started, submitted) per survey and question, storing them in a `analytics_summary` table. These summaries feed the reporting dashboard.  
* **Alerting:** When `klaviyo_sync_failed` or other errors exceed a threshold, send alerts to the merchant via email or admin dashboard notification.

## Recommended tooling

* **PostHog or Segment** – Provide an event pipeline with real‑time capture, user segmentation and dashboards.  
* **Supabase or Data Studio** – For internal analytics, summarise responses and generate ad‑hoc queries.  
* **Slack notifications (V2)** – Send notifications when a new survey is created, published or when integration errors occur.
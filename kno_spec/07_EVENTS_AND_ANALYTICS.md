# Events & Analytics

---

## Client Events (Checkout Extension)

| Event | Trigger | Key Payload Fields | Purpose |
|---|---|---|---|
| `survey_impression` | Survey widget renders on Thank You page | `shop_id`, `survey_id`, `order_id`, `customer_id?`, `timestamp` | Count eligible views; calculate view‑to‑completion rate |
| `survey_started` | Customer interacts with the survey (first answer or keystroke) | `shop_id`, `survey_id`, `response_id`, `order_id`, `customer_id?`, `timestamp` | Distinguish impressions from active respondents |
| `question_answered` | Customer answers a question | `response_id`, `question_id`, `answer_value(s)`, `timestamp`, `position` | Analyse drop‑offs and answer distribution |
| `survey_submitted` | Customer completes last question | `response_id`, `survey_id`, `completed_at` | Record completion; trigger promo code and integrations |
| `promo_code_clicked` | Customer clicks "Apply discount now" | `response_id`, `promo_code_id`, `timestamp` | Track redemption intent |

---

## Server Events (Backend & Integrations)

| Event | Trigger | Key Payload Fields | Purpose |
|---|---|---|---|
| `response_saved` | Backend writes a completed response to DB | `response_id`, `survey_id`, `shop_id`, `order_id`, `customer_id?`, `completed_at` | Confirm persistence |
| `response_linked_to_order` | Response matched to a Shopify order | `response_id`, `order_id`, `total_price`, `currency`, `timestamp` | Foundation for Order Attribution (V2) |
| `promo_code_generated` | Discount code created via Shopify API | `promo_code_id`, `shop_id`, `survey_id`, `code`, `expires_at` | Auditing and rate‑limit tracking |
| `promo_code_redeemed` | Shopify webhook fires when code is used | `promo_code_id`, `order_id`, `redeemed_at` | Measure incentive effectiveness |
| `klaviyo_sync_succeeded` | Worker successfully pushes response to Klaviyo | `response_id`, `survey_id`, `klaviyo_profile_id`, `timestamp` | Monitor integration reliability |
| `klaviyo_sync_failed` | Worker fails to push after retries | `response_id`, `error_message`, `timestamp` | Surface errors to merchant |
| `survey_created` | Merchant creates a survey | `survey_id`, `shop_id`, `created_at` | Usage analytics |
| `survey_published` | Survey status changes to published | `survey_id`, `shop_id`, `published_at` | Track go‑live events |
| `audience_modified` | Merchant updates an audience rule | `audience_id`, `survey_id`, `shop_id`, `changes`, `timestamp` | Audit targeting changes |

---

## Event Collection & Processing

- **Instrumentation** — Checkout extension batches events and sends via fetch to the backend. Backend logs to an `events` table and optionally forwards to PostHog or Segment.
- **Privacy** — Only capture order ID and customer ID. Email addresses (from link surveys) treated as sensitive.
- **Sampling** — For high‑traffic stores, `question_answered` events may be sampled to reduce volume.
- **Aggregation** — Nightly jobs compute views, started, and submitted counts per survey and question into an `analytics_summary` table. This powers the reporting dashboard.
- **Alerting** — When `klaviyo_sync_failed` errors exceed a threshold, alert the merchant via the admin dashboard or email.

---

## Recommended Tooling

- **PostHog or Segment** — Real‑time event pipeline with user segmentation and dashboards.
- **Supabase or Data Studio** — Internal analytics and ad‑hoc queries.
- **Slack notifications (V2)** — Notify on survey publish or integration errors.

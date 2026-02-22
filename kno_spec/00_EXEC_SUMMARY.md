# KNO Post‑Purchase Surveys – Executive Summary

## What KNO Does

KNO Post‑Purchase Surveys is a Shopify app that embeds multi‑question surveys on the order confirmation (Thank You) and order status pages to collect zero‑party data immediately after checkout. It's designed for DTC brands and agencies who want to understand how customers discovered their brand, what motivated the purchase, and how long they considered buying.

## Core Capabilities

**Survey Builder** — Template-driven setup with 30+ full survey templates and pre‑built questions. Supports radio, checkbox, text, scale, dropdown, and other question types depending on plan. Merchants can build from scratch or start from a template.

**Logic & Follow‑ups** — Questions support conditional logic so follow‑up questions or Actions only display based on earlier answers. Logic is configured on the follow‑up question, enabling multiple branching paths and exit points.

**Audience Targeting** — Merchants create Audiences using order and customer attributes (e.g., first‑time vs. returning buyers, product purchased, location, language, prior survey completion). When multiple surveys are live, display logic randomly picks a matching survey and supports mutually exclusive, sequential, or round‑robin experiences.

**Delivery** — Surveys are delivered via a Shopify Checkout UI extension on the Thank You/Order Status pages, or via unique URLs and QR codes. Link surveys support forced email capture to tie responses to customer profiles.

**Reporting & Attribution** — Survey stats, core reports, and an Order Attribution report link responses to order count, revenue, and AOV. The app estimates total orders/revenue by applying response rates to the full order cohort. Reports are filterable by survey, question, audience, and date range.

**Integrations** — Native integrations with Klaviyo, Shopify Flow, ReCharge, Triple Whale, Peel Insights, Google Analytics, and TikTok. Survey responses and status can be pushed into Klaviyo as custom properties, and Shopify Flow triggers can tag orders or customers.

**Post‑Survey Actions** — Promo codes (auto‑generated one‑time or multi‑use), review prompts, referral invites, and social follow CTAs. Actions can be conditionally displayed based on answers.

**Plans** — Starter, Analyst, Pro. Starter supports radio/checkbox/text questions and one survey. Analyst adds dropdown, slider, NPS, date picker, ranking, and email/phone capture. Pro unlocks image/video upload and unlimited audiences.

## Key Differentiators

1. **Embedded in Shopify checkout** — High response rates from surveys served directly on the Thank You/Order Status pages via Checkout UI extension.
2. **60+ audience targeting attributes** — Lifetime order count, location, product IDs, discount codes, and previous survey responses.
3. **Action engine** — Auto‑generated promo codes, review prompts, and referral invites within the survey widget.
4. **Order‑level attribution** — Ties answers to revenue and can project results as if 100% of customers responded.
5. **Deep automation hooks** — Klaviyo and Shopify Flow integrations push responses into customer profiles and trigger workflows without manual exports.

## MVP Scope

1. **Survey builder** with radio, checkbox, and text questions. Up to 3 questions per survey. Follow‑up logic and basic design customisation.
2. **Shopify Checkout UI extension** on the Thank You page. Order ID retrieved via `OrderConfirmationApi`.
3. **Audience targeting** for new vs. returning customers and product ID.
4. **Reporting dashboard** — views, submission rate, response distribution, CSV export, date filter.
5. **Klaviyo integration** — push selected question answers and survey status (view, partial, completed) as custom properties.
6. **Promo code action** — generate one‑time discount codes via Shopify Discounts API.

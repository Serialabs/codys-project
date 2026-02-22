# Product Requirements Document (PRD) – Post‑Purchase Survey App

## Problem statement

Direct‑to‑consumer merchants on Shopify often struggle to understand which marketing channels and messages drive sales. Existing analytics tools rely on cookies or last‑click attribution that doesn’t capture the full customer journey.  Merchants need a lightweight way to ask customers **immediately after purchase** how they heard about the brand and what motivated the purchase, then connect those answers to order data and automate follow‑up incentives.  Without an integrated solution, merchants rely on manual surveys or guesswork, leading to inefficient marketing spend and missed opportunities.

## Target users (personas)

1. **DTC brand owner (“Taylor”)**  
   *Runs a small lifestyle brand on Shopify.* Taylor manages marketing, product and fulfilment.  She wants to know which influencer collaborations and ad campaigns are driving sales so she can allocate her limited budget wisely.

2. **E‑commerce marketer (“Jordan”)**  
   *Works at a mid‑sized Shopify store.* Jordan is responsible for growth and retention.  They need to measure the impact of each channel (social, email, referral) and to trigger personalised Klaviyo flows based on customer feedback.

3. **Agency analyst (“Sam”)**  
   *Advises multiple Shopify clients.* Sam wants a standardised way to capture zero‑party data across clients, benchmark performance and quickly export results for slide decks.  Sam may have technical skills but prefers a turnkey solution.

## Jobs‑to‑be‑done

* “When I’ve launched marketing campaigns, I want to ask customers how they found us and why they bought, so I can verify our attribution assumptions.”  
* “When I need to increase repeat purchases, I want to offer a discount code or social follow CTA immediately after the survey, so I can incentivise engagement.”  
* “When analysing performance, I want to link survey responses to orders and revenue, so I can calculate ROI per channel.”  
* “When integrating with my CRM/ESP, I want survey answers to sync automatically, so I can personalise email flows without manual exports.”

## Functional requirements

### Survey builder
1. Merchants can create a survey using a template or start from scratch.  
2. Surveys support basic question types: radio (single choice), checkbox (multi‑select) and short/long text inputs【739290907688265†L35-L58】.  
3. Merchants can configure follow‑up logic on questions to show conditional follow‑ups and define exit points【945476463987832†L69-L116】.  
4. The survey can include an optional **promo code action** that generates one‑time discount codes and displays them on completion【295004260503847†L77-L104】.  
5. The builder enforces plan limits on number of questions and surveys, providing inline validation and upgrade prompts【978605946208732†L200-L214】.

### Placement and delivery
1. The survey must render as a **Checkout UI extension** on the Shopify **Thank You page**.  
2. The extension fetches the **order ID** via `OrderConfirmationApi` to link responses【956241798895964†L160-L169】.  
3. Optionally, the extension can also be placed on the Order Status page (V2).  
4. The system must support one active survey in the MVP; multi‑survey rotation and link/QR delivery are deferred.

### Targeting
1. Merchants can define a simple audience: new customers (lifetime order count = 1) vs returning (order count > 1) and optionally filter by product ID【607622501565733†L65-L119】.  
2. The survey displays only to customers matching the audience; others see nothing.  
3. In future versions, merchants can create complex audiences using additional attributes and priorities【132019224187076†L62-L120】.

### Data capture
1. For each survey response, store: shop ID, survey ID, order ID, customer ID (if available), timestamp and the answers to each question.  
2. Associate responses with the order using the order ID to enable later revenue analysis.  
3. Provide an optional field to capture the customer’s email address if delivered via link (V2).  
4. Store any promo code generated for auditing redemption.

### Reporting & integrations
1. Merchants can view a dashboard with counts of survey views, partial completions and completed submissions and distribution of answers per question【954105040073377†L83-L100】.  
2. Provide a CSV export of responses with order and customer identifiers.  
3. Integrate with **Klaviyo**: allow merchants to toggle per question whether to sync the answer as a custom property and push survey status events (view, partial, completed)【959380932338212†L109-L178】.  
4. In later versions, support an Order Attribution report linking revenue and AOV to answers【954105040073377†L164-L181】, plus triggers for Shopify Flow【15588629035657†L36-L63】.

### Admin and platform requirements
1. The app must follow Shopify’s App Bridge guidelines and use Polaris components for a native admin experience.  
2. Authentication uses Shopify OAuth with offline access for API calls.  
3. Use Shopify Billing API to manage plan upgrades/downgrades.  
4. The checkout extension must be deployed via the **Checkout UI Extension** API and abide by its limitations (no custom HTML/CSS injection, no video/slider questions【487850571088937†L315-L333】).  
5. The backend must comply with Shopify’s rate limits and verify webhooks using HMAC.

## Non‑functional requirements

* **Performance:** Survey widget should load within 500 ms after the Thank You page renders; answer submissions should complete in under 200 ms.  
* **Reliability:** The system should have 99.9 % uptime and gracefully handle API failures (retry with exponential backoff).  
* **Security & compliance:** Use HTTPS for all communications, encrypt sensitive data at rest, respect GDPR/CCPA by offering data deletion and anonymity options【985696967857397†L102-L125】.  
* **Scalability:** Support up to 10k survey responses per day for large stores; DB and queues should scale horizontally.  
* **Accessibility:** UI components must meet WCAG 2.1 AA standards; keyboard navigation and screen reader labels are required.  
* **Internationalisation (V2):** Future support for multiple languages using Shopify’s translation APIs【978605946208732†L102-L123】.

## Success metrics

* **Survey completion rate** – percentage of customers who complete the survey after seeing it (target ≥ 50 % for post‑purchase placement).  
* **Response attribution coverage** – proportion of orders linked to at least one survey answer (target ≥ 30 % for the MVP).  
* **Time to first insight** – time from installation to first actionable insight (goal: within 1 day).  
* **Promo code redemption rate** – percentage of generated discount codes that are redeemed (measure engagement of the Action).  
* **Integration usage** – number of merchants enabling Klaviyo integration and events successfully pushed.

## Constraints

* **Shopify checkout extensibility limitations**: cannot inject arbitrary HTML/CSS, video or slider questions inside the checkout【487850571088937†L315-L333】.  
* **Plan limits:** Starter plan supports only one survey and a fixed number of questions.  
* **Data privacy laws:** Must respect GDPR/CCPA; avoid collecting sensitive personal data, and provide opt‑out or deletion mechanisms.  
* **Cross‑browser behaviour:** Must work in the modern browsers supported by Shopify checkout.  
* **Time and resource constraints:** MVP to be built by a solo developer within two weeks.

## Out of scope

* Advanced question types (dropdown, slider, NPS, date picker, ranking, image/video) – deferred to V2【739290907688265†L63-L115】.  
* Multi‑survey rotation, mutually exclusive and round‑robin logic – not required for first version【132019224187076†L62-L120】.  
* Order Status page and link/QR delivery – to be added after core Thank You page extension.  
* Benchmarks and automated insights – reserved for higher plans【624157833394015†L40-L87】.  
* Shopify Flow triggers and other integrations (Triple Whale, Peel)【15588629035657†L36-L63】.  
* Multi‑user roles and team management – implement later.

## Risks and mitigations

| Risk | Mitigation |
| --- | --- |
| **Low response rate** – Customers may ignore the survey on the Thank You page. | Keep the survey short (1–3 questions), position it at the top of the page, and provide an incentive (promo code).  Monitor response rate and adjust question count accordingly. |
| **Performance impact on checkout** – The survey widget could delay the Thank You page. | Load the survey asynchronously after the page has rendered; use caching and minimal script size.  Ensure extension execution time stays within Shopify guidelines. |
| **Dependency on Klaviyo** – Merchants may expect full CRM integration. | Provide a robust Klaviyo integration for MVP; document limitations (only pushes new responses, historical data must be imported manually【959380932338212†L186-L191】). |
| **Plan limit confusion** – Merchants might exceed question limits without realising. | Display clear indicators of remaining questions and surveys in the builder; prompt to upgrade when limits are hit. |
| **Privacy concerns** – Collecting email via link surveys may raise privacy issues. | Make email capture optional, provide anonymity mode, and include a privacy notice explaining how data is used【985696967857397†L102-L125】. |
| **Shopify checkout API changes** – Shopify’s extensibility API is evolving. | Monitor Shopify developer updates, design the extension with feature flags to disable unsupported components (e.g., slider questions). |

## Acceptance criteria for MVP

* ✅ Merchant can install the app via Shopify App Store and access a dashboard.  
* ✅ Merchant can create one survey with up to three questions using radio, checkbox or text inputs.  
* ✅ Merchant can define a basic audience (new vs returning customers) and assign it to the survey.  
* ✅ Merchant can add a promo code action that generates a single‑use discount code and displays it after survey completion.  
* ✅ Merchant can add the survey block to the Thank You page via the Checkout editor and select the survey to display【734100394826036†L190-L214】.  
* ✅ Customers who meet the audience criteria see the survey on the Thank You page, answer questions and receive a promo code.  
* ✅ Survey responses are saved with order ID and customer ID (if available).  
* ✅ Merchant sees summary metrics (views, completes, response distribution) and can export a CSV.  
* ✅ Merchant can enable Klaviyo integration and push survey responses and status as custom properties.  
* ✅ System meets performance and security requirements.
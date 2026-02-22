# Feature Inventory – KNO Post‑Purchase Surveys

The following table enumerates the capabilities of KNO Post‑Purchase Surveys grouped by functional area.  Each feature is labelled as part of the **MVP** or deferred to **V2** based on the initial build scope defined in the executive summary.

## Survey creation

| Feature | Description (paraphrased) | Source | MVP or V2 |
| --- | --- | --- | --- |
| **Template‑based survey creation** | Merchants can start from 30+ pre‑built templates or build from scratch; the builder lets them add, remove and reorder questions. | App listing emphasises 5‑minute setup with templates【990055361029885†L117-L124】【367261123567251†L120-L139】. | **MVP** – start with a few starter templates to accelerate onboarding. |
| **Question types – basic** | Radio buttons, checkboxes, text input and text area are available to all plans. These cover single‑choice, multi‑choice and free‑text responses. | Question types doc lists radio, checkbox, text input and textarea for the Starter plan【739290907688265†L35-L58】. | **MVP** – include radio, checkbox and short/long text. |
| **Question types – advanced** | Dropdowns, sliders, Net Promoter Score/scale, date pickers, rankings, email/phone capture, image or video uploads. | Analyst plan adds dropdown, slider, NPS, date picker, ranking, email and phone【739290907688265†L63-L115】; Pro plan adds image/video upload【739290907688265†L110-L115】. | **V2** – advanced inputs require extra components and validation. |
| **Logic and branching** | Questions can have follow‑up logic. Logic is configured on the follow‑up question; merchants can show follow‑ups based on specific answer choices, end the survey early, or skip to the confirmation screen. | Logic guide explains how follow‑up logic is set and that detailed choice allows branching by answer【945476463987832†L69-L116】. | **MVP** – simple logic (show follow‑up if certain answer selected). |
| **Other options toggle** | Merchants can add an “Other” option to multiple‑choice questions that reveals a text input. | Survey creation doc mentions toggling the “Other” field on a multiple‑choice question【451442093169152†L53-L93】. | **V2** – useful but not essential for MVP. |
| **Multiple surveys per store** | Plans allow multiple surveys to be created and run concurrently; each plan caps the number of live surveys. | Pricing table lists plan limits: Starter includes one survey, Analyst allows multiple【978605946208732†L200-L214】. | **V2** – initial release could limit merchants to a single survey. |
| **Survey styling** | Global theme settings and per‑survey styling; light/dark theme, primary colour picker, navigation text edits and custom CSS injection. | Styling doc shows theme options and custom CSS editor【197526823399142†L70-L106】【197526823399142†L117-L151】. | **V2** – MVP uses default theme with minimal customisation. |
| **Post‑survey Actions** | After the last question, a survey can show Actions such as promo codes, review requests, referral invites, social follow buttons or VIP sign‑up; actions can be conditionally displayed via logic. | Actions doc lists available actions and emphasises logic to show them based on answers【672473035596646†L46-L62】【672473035596646†L129-L149】. | **MVP** – include promo code action only; other actions in V2. |
| **Promo code generator** | Generates single‑use or multi‑use discount codes in Shopify, includes fallback codes and automatically applies discount when customer clicks the button. | Promo code action doc describes generating one‑time codes, fallback codes and auto‑apply behaviour【295004260503847†L77-L104】【295004260503847†L160-L177】. | **MVP** – implement single‑use codes; multi‑use and fallback features for V2. |

## Targeting and timing

| Feature | Description | Source | MVP or V2 |
| --- | --- | --- | --- |
| **Audience builder** | Merchants define audiences using customer, order or survey data (e.g., lifetime order count, city, product variant ID, discount code, prior survey completion). Operators include equals, not equals, greater/less than and contains. | Audiences doc lists attribute types and operators【607622501565733†L65-L119】. | **MVP** – support simple audiences based on new vs returning customers and product IDs. |
| **Random survey selection** | When multiple surveys are live, the script on the thank‑you page checks which surveys apply and randomly selects one for the customer. | Targeting doc states that when multiple surveys apply, KNO randomly picks one and prevents the same customer seeing a survey again【311270689736979†L87-L109】. | **V2** – not required with single‑survey MVP. |
| **Display priority logic** | Merchants can designate mutually exclusive surveys, sequential follow‑ups or round‑robin rotation to prioritise which survey a customer sees. | Survey display priority guide explains mutually exclusive, previous survey completion (sequencing) and round‑robin logic【132019224187076†L62-L120】. | **V2** – advanced prioritisation comes after multiple surveys. |
| **Timing on Thank You/Order Status page** | Surveys appear on the order confirmation page immediately after checkout and can also show on the Order Status page when customers check their order later. | Extensibility setup doc shows how to add a KNO block to the Thank You and Order Status pages【487850571088937†L167-L200】. | **MVP** – display on Thank You page; Order Status can be added later. |
| **Link and QR delivery** | Each survey has a unique URL that can be shared via email, QR code or social media. Merchants can force email capture or pre‑fill the email parameter to attribute responses to shoppers. | Delivery doc notes unique survey URLs, email capture and QR codes【985696967857397†L32-L93】. | **V2** – focus on in‑checkout surveys first; add link sharing later. |

## Placement surfaces

| Surface | Description | Source | MVP or V2 |
| --- | --- | --- | --- |
| **Shopify checkout extension (Thank You page)** | Primary placement: embed survey as an app block on the order confirmation (Thank You) page using the Shopify checkout UI extension. | Extensibility guide describes adding a KNO block in Checkout Editor【734100394826036†L190-L214】. | **MVP** – core channel for capturing high response rates. |
| **Order Status page** | Customers returning to check their order can see the survey on the Order Status page. | Extensibility guide notes that the block can also be added to the Order Status page【487850571088937†L167-L200】. | **V2** – optional after initial release. |
| **Standalone link/QR** | Surveys accessible via unique URL or QR code; can be delivered in email campaigns, packaging inserts or receipts. | Delivery doc covers unique survey links and QR codes【985696967857397†L32-L93】. | **V2** – not required for initial checkout focus. |
| **Embedded in emails** | Some competitors embed surveys directly in email. KNO uses links rather than inline embed, but this could be extended. | Not directly documented; noted for comparison with other tools. | **V2** – optional if needed. |

## Data capture and linkage

| Feature | Description | Source | MVP or V2 |
| --- | --- | --- | --- |
| **Order linkage** | KNO retrieves the order ID via Shopify’s `OrderConfirmationApi` and `OrderStatusApi` when the survey loads, then associates the response with the correct order. | Shopify dev doc states that order ID is available via `OrderConfirmationApi` on the thank‑you page and `OrderStatusApi` on the order status page【956241798895964†L160-L169】. | **MVP** – store order ID for each response. |
| **Customer linkage** | If the customer is logged in, the survey captures the customer ID; for guest checkouts or link surveys, merchants can request email input or pre‑fill via query string. | Delivery doc explains forcing email capture and pre‑filling the email parameter to tie responses to shoppers【985696967857397†L102-L125】. | **V2** – implement email capture on link surveys; in‑checkout surveys automatically get customer ID. |
| **UTM/source tracking** | Audiences and templates support tracking source/medium parameters (UTM tags) to understand marketing channel; the built‑in attribution survey uses this data in follow‑ups. | Audiences can use source/medium and UTM fields【607622501565733†L65-L119】; attribution template asks where customer first heard about the brand【364980791551066†L90-L109】. | **V2** – optional; MVP uses manual answer selection. |
| **Survey metadata** | Each response stores survey ID, date/time, question answers, and optional tags (audience, version, distribution channel). | Implicit from reporting features; order attribution report aggregates responses by date and survey【954105040073377†L83-L100】. | **MVP** – record essential metadata; tags later. |

## Reporting and analytics

| Feature | Description | Source | MVP or V2 |
| --- | --- | --- | --- |
| **Survey stats dashboard** | Shows views, partial completions, completions and question‑level response distribution per survey. | Reporting doc mentions survey stats and response counts【954105040073377†L83-L100】. | **MVP** – include view/submission counts and simple charts. |
| **Order Attribution report** | Aggregates survey responses with order data to show number of orders, revenue and average order value per answer choice; can scale to estimate full cohort via the “Estimate 100 %” toggle. | Order attribution doc explains methodology and the Estimate 100 % option【954105040073377†L164-L181】. | **V2** – advanced reporting requiring revenue data. |
| **Benchmarking & automated insights** | Higher tiers provide aggregated demographic and purchase motivator insights and benchmark comparison across brands. | Marketing copy describes automated insights with demographic/psychographic questions and benchmarks【624157833394015†L40-L87】【624157833394015†L116-L130】. | **V2** – outside initial scope. |
| **CSV export** | Ability to export survey responses, including answers and order metadata, as CSV for external analysis or upload to other tools. | Implied by integration docs and analytics; merchants must manually upload historical data to Klaviyo【959380932338212†L186-L191】. | **MVP** – include simple CSV export. |
| **Filters and segments** | Report filters by survey, question, audience and date range; ability to segment responses by audience criteria. | Order attribution report allows filtering by date range and survey【954105040073377†L83-L100】. | **V2** – simple date filter in MVP, full segmentation later. |

## Integrations

| Integration | Description | Source | MVP or V2 |
| --- | --- | --- | --- |
| **Klaviyo integration** | Pushes survey status (view, partial, completed) and selected question responses into Klaviyo as custom properties. Merchants can toggle which questions sync. | Klaviyo doc outlines connection and toggling responses【959380932338212†L109-L178】. | **MVP** – implement; part of initial plan. |
| **Shopify Flow** | Survey completion or specific answers can trigger Flow workflows; flows receive data like question/response IDs, survey metadata, customer lifetime orders/spent and order total/currency. | Flow integration doc shows data passed to Flow and enabling triggers per question【15588629035657†L36-L63】【15588629035657†L271-L314】. | **V2** – optional for more advanced automation. |
| **Analytics/BI tools (Triple Whale, Peel, GA, TikTok)** | KNO lists many analytics and attribution tools among partners; integration details are minimal. | App listing mentions integrations with Triple Whale, Peel Insights, Google Analytics, TikTok and others【978605946208732†L121-L123】. | **V2** – focus on Klaviyo initially; add connectors later. |
| **Recharge subscription sync** | KNO integrates with subscription platforms to capture data for subscription orders. | App listing lists Recharge integration【978605946208732†L129-L139】. | **V2** – beyond initial scope. |
| **Custom webhooks/API** | Some merchants may need to send responses to proprietary systems; no public docs were found. | Not directly documented. | **V2** – implement after core features. |

## Admin & management

| Feature | Description | Source | MVP or V2 |
| --- | --- | --- | --- |
| **Team roles and permissions** | Ability to invite team members and assign roles (admin vs viewer). | Not found in public docs; assumption that at least basic team sharing exists. | **V2** – defer until multi‑user support is required. |
| **Plan management** | Merchants can upgrade/downgrade plans, which unlock additional questions, audiences and integrations. | App listing shows tier differences【978605946208732†L200-L249】. | **V2** – rely on Shopify Billing for plan upgrades. |
| **Error handling & validation** | Builder warns when question limit is exceeded, invalid logic is configured, or actions are missing required fields; survey can’t be published until fixed. | Assumption based on typical builder UX; not explicitly documented. | **MVP** – basic validations (e.g., require question text, limit number of questions). |
| **Duplicate suppression** | Customers will not see the same survey again and are only shown one survey even if multiple apply. | Targeting doc states customers won’t see same survey twice and random selection ensures only one survey is served【311270689736979†L87-L109】. | **V2** – optional once multiple surveys exist. |
| **Support for multiple languages** | Surveys can be translated via Langify and other translation apps; KNO itself offers some translations (English, French, Spanish, Italian). | App listing mentions support for multiple languages and compatibility with translation apps【978605946208732†L102-L123】. | **V2** – outside initial release; rely on static text in English. |

## Limits and constraints

| Constraint | Description | Source |
| --- | --- | --- |
| **UI extension limitations** | Under Shopify checkout extensibility, KNO cannot support custom CSS styling of surveys, video questions, slider questions, dynamic confirmation screens and custom HTML actions. | Extensibility doc lists features that are not currently supported in the checkout extension【487850571088937†L315-L333】. |
| **Survey question and audience caps** | Plans cap the number of surveys (1–unlimited), questions per survey, audiences, integrations and actions. Starter plan supports only basic question types. | Pricing table shows plan limits and features【978605946208732†L200-L249】【881960173322908†L36-L118】. |
| **Response bias** | With opt‑in surveys, response rates may be below 100 %; the Order Attribution report’s estimation option scales results to 100 % but relies on assumptions. | Order attribution report explains the “Estimate 100 %” toggle【954105040073377†L164-L181】. |
| **Data privacy** | Surveys collect zero‑party data; merchants must respect privacy laws (GDPR, CCPA). KNO offers anonymity options for link surveys and the ability to disable email capture. | Delivery doc allows anonymous surveys by not forcing email capture【985696967857397†L102-L125】. |

This inventory informs the later PRD and technical design by outlining which capabilities form the minimum viable product and which are planned for subsequent releases.
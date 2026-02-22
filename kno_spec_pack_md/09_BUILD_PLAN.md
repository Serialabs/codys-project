# Build Plan – Post‑Purchase Survey App

The following plan outlines a 10‑day schedule for a solo developer to build the MVP defined in the PRD.  Adjust timelines based on personal pace and complexity.

## Timeline and milestones

### **Day 1: Project setup & planning**
* Scaffold the Shopify Remix app using the Shopify CLI.  
* Initialise Git repository, configure ESLint/Prettier, install Polaris and Prisma.  
* Create a **project board** with tasks corresponding to this plan.  
* Review Shopify Checkout UI extension docs and set up a development store.

### **Day 2: Database & models**
* Define the Prisma schema based on the data model (shops, surveys, questions, choices, audiences, responses, promo codes).  
* Run `prisma migrate dev` to create the local database.  
* Implement seed scripts for sample surveys and questions to speed up UI development.

### **Day 3: OAuth & install flow**
* Implement Shopify OAuth using `@shopify/shopify-api`.  
* Store access tokens in the `shops` table and handle app install/uninstall webhooks.  
* Test app installation on the dev store; ensure proper redirection to the app dashboard.

### **Day 4: Admin UI scaffolding**
* Build the basic admin dashboard layout using Polaris.  
* Add navigation: **Surveys**, **Audiences**, **Analytics**, **Integrations**.  
* Implement listing of existing surveys and a **Create survey** button.

### **Day 5: Survey builder**
* Build the survey creation/editing page: input for title, Add Question modal with radio/checkbox/text inputs, reorderable list.  
* Implement follow‑up logic UI on the follow‑up question (dropdown to select parent question and answer).  
* Implement promo code action configuration (discount value, expiry).  
* Add client‑side validation for required fields and plan limits.  
* Save surveys to the database via API routes.

### **Day 6: Audience builder & targeting**
* Create the Audiences page: form to define simple attributes (lifetime order count equals 1, product ID in list).  
* Save audience rules to the database and associate them with surveys.  
* In the survey settings, allow selecting one audience.  
* Build preview logic to estimate how many customers meet the criteria (using sample data or order API stub).

### **Day 7: Checkout UI extension**
* Create the survey widget as a Checkout UI extension with React.  
* Fetch the active survey and audience rules from the backend when the Thank You page loads.  
* Use `OrderConfirmationApi` to retrieve the order ID【956241798895964†L160-L169】.  
* Render questions, handle navigation and collect answers.  
* Generate a unique response ID client‑side and send `survey_impression`, `survey_started`, `question_answered`, and `survey_submitted` events to the backend.

### **Day 8: Backend endpoints & discount codes**
* Implement REST endpoints: get active survey (by shop and audience), save response and answers, generate promo code.  
* Secure endpoints with HMAC verification.  
* Integrate with Shopify Discounts API to create single‑use codes for the promo action【295004260503847†L77-L104】.  
* Queue code generation jobs using BullMQ; implement basic retry logic.

### **Day 9: Analytics dashboard & CSV export**
* Build the analytics page: display total views, starts and completions; show a bar chart for each question’s answer distribution.  
* Implement API route to aggregate events and responses from the database.  
* Add **CSV export** button that streams a CSV of responses (order ID, customer ID, answers).  
* Ensure pagination or streaming for large data sets.

### **Day 10: Klaviyo integration & polish**
* Create settings page for integrations; allow merchants to enter their Klaviyo API key and toggle which questions to sync【959380932338212†L109-L178】.  
* Implement a worker that listens for new responses and pushes selected properties to Klaviyo; handle success and failure events.  
* Write unit tests for core logic (survey creation, audience matching, discount code generation) and end‑to‑end test for the checkout extension using Playwright.  
* Perform manual QA on desktop and mobile; confirm that the survey displays correctly in the checkout and that responses are recorded.  
* Prepare documentation and deployment scripts; deploy to production.

## Testing plan

* **Unit tests** – Test data model functions, audience matching logic and discount code generation with Jest.  
* **Integration tests** – Use supertest to test API endpoints (survey fetch, response submit).  
* **End‑to‑end tests** – Use Playwright to simulate a customer placing an order on a development store and completing a survey; verify events and database records.  
* **Manual tests** – Place orders in different scenarios (guest vs logged in, new vs returning) and verify the survey appears appropriately.  
* **Performance tests** – Use load testing (e.g., k6) to simulate concurrent survey submissions and ensure the API and queue can handle expected load.

## Definition of Done

* The app passes all automated tests and manual QA.  
* A merchant can install the app, create and publish a survey with up to three questions and an optional promo code action.  
* The survey shows on the Thank You page for customers meeting the audience criteria, collects responses and stores them in the database.  
* The merchant can view analytics (views, starts, completes, distribution) and export responses as CSV.  
* The merchant can enter a Klaviyo key, toggle questions and see responses synced to Klaviyo.  
* The application is deployed to production with secure environment variables and webhooks registered.  
* All features meet performance, security and accessibility requirements.  
* Open questions and deferred features are documented for future iterations.
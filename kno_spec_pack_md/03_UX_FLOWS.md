# UX Flows – KNO Post‑Purchase Surveys

This document describes the key user experiences as step‑by‑step flows.  Each flow lists entry conditions, UI states/screens, primary happy paths, error states and notable edge cases.

## 1. Merchant Flow: Install → Configure → Publish → Interpret Results

### Entry conditions
* Merchant has a Shopify store with checkout extensibility enabled.  
* They have decided to install a post‑purchase survey to learn why customers purchased.

### Steps
1. **App installation**  
   * Merchant visits the Shopify App Store and clicks **Add app** for “KNO Post‑Purchase Surveys.”  
   * After OAuth authorization, the app is listed in the store’s Apps.  The merchant is redirected to the KNO admin dashboard.
2. **Onboarding wizard**  
   * Dashboard presents a welcome screen summarising benefits and showing plan options (Starter, Analyst, Pro).  A “Start free trial” button leads to the builder.  
   * Merchant selects a plan (free trial or paid), which triggers Shopify Billing.  
   * The wizard prompts the merchant to **create their first survey** using a template or blank.  Basic templates include attribution (“How did you hear about us?”) and customer feedback (“Why did you buy?”).  
   * The wizard also instructs the merchant to open the Shopify **Checkout editor** and add the KNO app block to the Thank You page【734100394826036†L190-L214】.  A side panel provides step‑by‑step instructions with screenshots.
3. **Survey builder**  
   * The survey builder is a form‑like interface with a title, description and a list of questions.  
   * **Add Question** button opens a modal to choose a question type (radio, checkbox, text input).  Merchant enters the prompt and answer choices.  
   * Merchant can enable follow‑up logic: on the follow‑up question there is a toggle to “Show if previous answer is X.”  They select the parent question and answer value【945476463987832†L69-L116】.  
   * **Add Action** can be used at the end of the survey to insert a **promo code action**; merchant chooses discount amount, sets expiration and whether to auto‑apply the code【295004260503847†L77-L104】.  
   * Merchant configures simple design settings (title text, primary button colour) or uses the default theme.  
   * Validation runs on save: required fields must be filled, maximum number of questions enforced per plan.  Errors display inline with the relevant field.  
   * Merchant clicks **Save** and then **Publish** to make the survey live.
4. **Audience creation**  
   * Merchant navigates to **Audiences** and clicks **Create audience**.  They choose an attribute (e.g., lifetime order count) and operator (e.g., equals 1) to target new customers【607622501565733†L65-L119】.  
   * They assign the audience to the survey in the survey settings.  
   * They can preview the audience size (approximate) based on historical orders.  
5. **Activating the Checkout extension**  
   * Merchant opens Shopify’s **Checkout editor**.  
   * On the Thank You page, they click **Add app block** → **KNO Post‑Purchase Survey** and position it at the top of the page.  A configuration form appears to select which survey to display【734100394826036†L190-L214】.  
   * Merchant saves and publishes the checkout.  
6. **Interpreting results**  
   * In the KNO dashboard, the **Analytics** tab shows summary metrics: total views, completed responses, response rate and distribution charts for each question.  
   * Merchant can export a CSV or toggle the Klaviyo integration to sync responses.  They can filter by date range and survey.  
   * If they have the **Order Attribution report** (V2), they navigate to a dedicated page with tables of orders, revenue and average order value per answer choice【954105040073377†L83-L100】.  
   * The merchant uses insights to adjust marketing budgets or product development.

### Error states
* **Missing app block** – If the merchant publishes a survey but forgets to add the app block in the checkout editor, customers will never see it.  The dashboard should display a warning banner.  
* **Plan limits exceeded** – When the merchant adds more questions than their plan allows, the builder disables publish and shows a tooltip referencing the limit【978605946208732†L200-L214】.  
* **Invalid logic** – If a follow‑up question references a parent question that has been deleted, the builder surfaces an error requiring re‑selection.

### Edge cases
* **Mobile vs desktop** – The checkout extension must be responsive. On mobile, the survey collapses into a card that expands when tapped.  
* **Multiple surveys** – In later versions, if the merchant has multiple surveys with overlapping audiences, only one will show; the system uses random or sequential priority logic【132019224187076†L62-L120】.  
* **Guest checkout** – If the order is placed by a guest, the survey can still link to the order ID but cannot link to customer ID; merchants may ask for email in the survey or rely on the Klaviyo pre‑fill parameter.【985696967857397†L102-L125】

## 2. Customer Flow: Complete purchase → See survey → Submit → Post‑submit

### Entry conditions
* Customer has just completed checkout on a Shopify store using the KNO survey app.  
* The store’s checkout extension is configured to show a survey for the audience matching this customer.

### Steps
1. **Thank You page load**  
   * After payment, the customer is directed to the order confirmation page.  The top section displays a **survey card** titled “We’d love your feedback!” (or merchant‑customised text).  The first question appears with multiple‑choice options or text input.  
   * The survey component requests the order ID via Shopify’s `OrderConfirmationApi` and associates the session with the order【956241798895964†L160-L169】.  
2. **Answer questions**  
   * Customer selects an option or types an answer.  For multiple‑choice questions, the “Next” button activates once an answer is selected.  
   * If conditional logic applies, the next question may change based on their answer (e.g., if they select “Influencer,” a follow‑up asks which influencer【364980791551066†L90-L109】).  
   * Customer can skip optional questions or exit early; mandatory questions block progression.  
3. **Completion and Action**  
   * Upon answering the last question, a **confirmation screen** thanks them for their input.  If a promo code action is attached, it shows the discount code and a button “Apply discount now.” Clicking opens a new tab with the discount applied to the store【295004260503847†L77-L104】【295004260503847†L160-L177】.  
   * If no action is attached, the survey simply displays a thank‑you message and closes.  
4. **Background events**  
   * The survey sends `survey_submitted` and `survey_completed` events to the backend.  If the customer is logged in, their customer ID is saved; if not, only the order ID is stored.  
   * If Klaviyo integration is enabled, the answers are pushed as custom properties【959380932338212†L109-L178】.  

### Error states
* **Network failure** – If the survey fails to load due to network issues, a message asks the customer to refresh the page.  
* **Validation error** – If the customer attempts to proceed without selecting a required option, the form highlights the unanswered field.  
* **Promo code failure** – If the discount code cannot be generated or applied, the action displays an error and offers a fallback code or instructs the customer to copy and paste【295004260503847†L160-L177】.

### Edge cases
* **Returning customer** – If the customer placed multiple orders, the audience logic may exclude them from the survey (e.g., only new customers).  
* **Multiple surveys** – When more than one survey matches, the system randomly selects one; the customer does not know the others exist【311270689736979†L87-L109】.  
* **Link surveys** – If the survey is delivered via email or QR code, the first page may ask for the customer’s email to link the response【985696967857397†L102-L125】.  The UI collects email before showing questions and stores it with the response.

## 3. Reporting Flow: Filter → Segment → Attribution report → Export/Share

### Entry conditions
* Merchant has received responses to at least one survey.  
* They want to understand the distribution of answers and link responses to orders and revenue.

### Steps
1. **Access analytics**  
   * From the KNO admin dashboard, merchant clicks the **Analytics** tab.  They see a list of surveys with view, response and completion counts.  
   * Clicking a survey opens a detailed report with question‑level charts (bar charts for multiple choice, word clouds for text answers).  
2. **Filter and segment**  
   * Merchant can select a **date range**, choose the survey and optionally filter by audience or product.  Basic filters are available in the MVP.  
   * In V2, additional segmentation (first‑time vs returning, product category, location) can be selected from a sidebar.  
3. **Interpret results**  
   * For each question, the report shows the percentage and count of each answer.  For text answers, the merchant can view a list or export as CSV.  
   * If using the Order Attribution report, an additional table lists answer choices with associated **order count, revenue and average order value**; the merchant can toggle **Estimate 100 %** to scale the numbers based on response rate【954105040073377†L164-L181】.  
4. **Export and share**  
   * Merchant can download a **CSV export** of responses including order ID, customer ID/email and answers.  
   * In V2, merchants can share a live report link with team members or embed charts in other dashboards.  
5. **Integration hooks**  
   * If Klaviyo integration is enabled, the report provides a link to view contacts in Klaviyo with the new properties.  
   * For Shopify Flow (V2), the merchant configures workflows to trigger emails or Slack messages when a specific answer is recorded【15588629035657†L36-L63】.

### Error states
* **No data** – If there are no responses yet, the dashboard displays a placeholder encouraging the merchant to wait or check the placement on the Thank You page.  
* **Filter mismatch** – Selecting a date range with no data yields an empty chart; the system displays “No results for this range.”  
* **Permission error** – If the merchant’s role doesn’t allow viewing analytics (for multi‑user accounts), the dashboard denies access.

### Edge cases
* **Large datasets** – When exporting large numbers of responses, the system paginates or compresses the file.  
* **Deleted surveys** – If a survey is deleted, historical data is retained in reports but new responses stop.  A warning indicates that deleted surveys cannot be restored.  
* **Plan downgrade** – If the merchant downgrades to a plan that only supports one survey, additional surveys are paused and hidden from analytics until they upgrade again.
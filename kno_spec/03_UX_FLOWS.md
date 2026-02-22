# UX Flows – KNO Post‑Purchase Surveys

---

## 1. Merchant Flow: Install → Configure → Publish → Interpret Results

### Entry conditions
- Merchant has a Shopify store with checkout extensibility enabled.
- They want to capture post‑purchase feedback.

### Steps

**1. App installation**
Merchant installs via the Shopify App Store. After OAuth, they land on the KNO dashboard.

**2. Onboarding wizard**
- Welcome screen with plan selection (Starter, Analyst, Pro) and a free trial option.
- Wizard prompts merchant to create their first survey (template or blank). Basic templates include "How did you hear about us?" and "Why did you buy?"
- Wizard instructs merchant to open the Shopify Checkout editor and add the KNO app block to the Thank You page.

**3. Survey builder**
- Form‑based interface: title, description, list of questions.
- **Add Question** opens a modal to choose type (radio, checkbox, text). Merchant enters the prompt and answer choices.
- Follow‑up logic is configured on the follow‑up question: toggle "Show if previous answer is X," select the parent question and answer value.
- **Add Action** appends a promo code action: set discount amount, expiry, and whether to auto‑apply.
- Basic design settings (title text, primary button colour) or use the default theme.
- Validation on save: required fields enforced, question limit per plan enforced. Inline errors shown.
- **Save → Publish** to make the survey live.

**4. Audience creation**
- Navigate to **Audiences → Create audience**.
- Choose attribute (e.g., lifetime order count), operator (e.g., equals 1), and value.
- Assign the audience to the survey in survey settings.

**5. Activating the Checkout extension**
- Open Shopify's Checkout editor.
- On the Thank You page, click **Add app block → KNO Post‑Purchase Survey**. Position it at the top.
- Select which survey to display in the configuration panel.
- Save and publish the checkout.

**6. Interpreting results**
- In the **Analytics** tab: total views, completions, response rate, and per‑question distribution charts.
- Export CSV or use the Klaviyo toggle to sync responses.
- Filter by date range and survey.

### Error states
- **Missing app block** — Survey is published but the block isn't added to the checkout. Dashboard shows a warning banner.
- **Plan limits exceeded** — Adding more questions than the plan allows disables Publish and shows an upgrade tooltip.
- **Invalid logic** — A follow‑up references a deleted parent question; builder surfaces an error requiring re‑selection.

### Edge cases
- **Mobile** — The checkout extension must be responsive. Survey collapses into an expandable card on mobile.
- **Guest checkout** — Response links to order ID but not customer ID. Merchants can add an email question or rely on Klaviyo pre‑fill.

---

## 2. Customer Flow: Complete Purchase → Survey → Submit → Post‑Submit

### Entry conditions
- Customer has just completed checkout.
- The store's checkout extension is configured with a survey matching this customer's audience.

### Steps

**1. Thank You page load**
- A survey card appears at the top of the order confirmation page.
- The extension retrieves the order ID via `OrderConfirmationApi` and associates the session with the order.

**2. Answer questions**
- Customer selects an option or types an answer. "Next" activates after an answer is provided.
- Conditional logic may change the next question based on their answer.
- Optional questions can be skipped; required questions block progression.

**3. Completion and Action**
- Confirmation screen thanks them for their input.
- If a promo code action is attached: discount code is shown with an "Apply discount now" button that opens a new tab with the discount applied.
- If no action: simple thank‑you message and close.

**4. Background events**
- `survey_submitted` and `survey_completed` events sent to the backend.
- Customer ID saved if logged in; otherwise only order ID is stored.
- If Klaviyo integration is enabled, answers are pushed as custom properties.

### Error states
- **Network failure** — Survey fails to load; message asks customer to refresh.
- **Validation error** — Required question unanswered; field is highlighted.
- **Promo code failure** — Code can't be generated; fallback code displayed or customer instructed to copy/paste.

### Edge cases
- **Returning customer** — Audience logic may exclude them (e.g., new customers only).
- **Link surveys** — First page asks for email to link the response before showing questions.

---

## 3. Reporting Flow: Filter → Review → Export

### Entry conditions
- Merchant has at least one survey with responses.

### Steps

**1. Access analytics**
- From the dashboard, click **Analytics**.
- See list of surveys with view, response, and completion counts.
- Click a survey to open detailed report: bar charts for multiple choice, text list for open responses.

**2. Filter**
- Select a date range and survey. MVP supports date filter only; full segmentation is V2.

**3. Interpret results**
- Per question: percentage and count of each answer. Text answers viewable as a list.

**4. Export**
- Download a CSV with order ID, customer ID/email, and answers.

**5. Integration hooks**
- If Klaviyo is enabled, a link navigates to Klaviyo contacts with the synced properties.

### Error states
- **No data** — Placeholder encouraging merchant to check Thank You page placement.
- **Filter mismatch** — No results for selected range; "No results for this range" message shown.

### Edge cases
- **Large datasets** — Export paginates or compresses the file.
- **Deleted surveys** — Historical data retained; new responses stop. Warning shown that deletion is irreversible.
- **Plan downgrade** — Extra surveys paused and hidden from analytics until upgrade.

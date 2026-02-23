# KNO Post-Purchase Survey App — Claude Code Implementation Spec

## How to use this file

Each **Phase** is a self-contained Claude Code prompt. Complete one phase fully before starting the next. Do not skip ahead. At the end of each phase, verify the acceptance criteria before moving on.

**Stack:** Remix (TypeScript) · Polaris · Prisma · PostgreSQL · Redis · BullMQ · Shopify Checkout UI Extensions · Klaviyo API

---

## Phase 1 — Project Scaffold & Database Schema

### Prompt for Claude Code

Set up the base project and full database schema. Nothing else.

**Scaffold the app:**
- Use `shopify app create` with the Remix template and TypeScript.
- Install dependencies: `@shopify/polaris`, `prisma`, `@prisma/client`, `bullmq`, `ioredis`.
- Configure ESLint and Prettier.
- Create a `.env.example` with all required variables:
  - `SHOPIFY_API_KEY`, `SHOPIFY_API_SECRET`, `SCOPES`, `APP_URL`
  - `DATABASE_URL`, `REDIS_URL`
  - `KLAVIYO_API_KEY`, `JWT_SECRET`

**Define the full Prisma schema at `prisma/schema.prisma`:**

```prisma
model Shop {
  id             String    @id @default(uuid())
  shopDomain     String    @unique
  accessToken    String
  plan           Plan      @default(starter)
  installedAt    DateTime  @default(now())
  uninstalledAt  DateTime?
  surveys        Survey[]
  responses      Response[]
  promoCodes     PromoCode[]
  integrations   Integration[]
}

enum Plan { starter analyst pro }

model Survey {
  id            String     @id @default(uuid())
  shopId        String
  shop          Shop       @relation(fields: [shopId], references: [id])
  title         String
  status        SurveyStatus @default(draft)
  createdAt     DateTime   @default(now())
  updatedAt     DateTime   @updatedAt
  promoActionId String?
  questions     Question[]
  audiences     Audience[]
  responses     Response[]
  promoCodes    PromoCode[]
}

enum SurveyStatus { draft published archived }

model Question {
  id                String        @id @default(uuid())
  surveyId          String
  survey            Survey        @relation(fields: [surveyId], references: [id])
  type              QuestionType
  prompt            String
  isRequired        Boolean       @default(true)
  position          Int
  followUpParentId  String?
  followUpValue     String?
  syncToKlaviyo     Boolean       @default(false)
  choices           Choice[]
  responseItems     ResponseItem[]
}

enum QuestionType { radio checkbox text_short text_long }

model Choice {
  id         String   @id @default(uuid())
  questionId String
  question   Question @relation(fields: [questionId], references: [id])
  label      String
  value      String
  position   Int
}

model Audience {
  id        String         @id @default(uuid())
  surveyId  String
  survey    Survey         @relation(fields: [surveyId], references: [id])
  attribute AudienceAttr
  operator  AudienceOp
  value     String
  createdAt DateTime       @default(now())
  responses Response[]
}

enum AudienceAttr { lifetime_orders product_id }
enum AudienceOp   { eq gt lt in }

model Response {
  id           String         @id @default(uuid())
  surveyId     String
  survey       Survey         @relation(fields: [surveyId], references: [id])
  shopId       String
  shop         Shop           @relation(fields: [shopId], references: [id])
  orderId      String
  customerId   String?
  audienceId   String?
  audience     Audience?      @relation(fields: [audienceId], references: [id])
  completedAt  DateTime?
  promoCodeId  String?
  promoCode    PromoCode?     @relation(fields: [promoCodeId], references: [id])
  items        ResponseItem[]
}

model ResponseItem {
  id          String   @id @default(uuid())
  responseId  String
  response    Response @relation(fields: [responseId], references: [id])
  questionId  String
  question    Question @relation(fields: [questionId], references: [id])
  choiceValue String?
  textAnswer  String?
}

model PromoCode {
  id         String    @id @default(uuid())
  shopId     String
  shop       Shop      @relation(fields: [shopId], references: [id])
  surveyId   String
  survey     Survey    @relation(fields: [surveyId], references: [id])
  code       String    @unique
  createdAt  DateTime  @default(now())
  expiresAt  DateTime?
  isRedeemed Boolean   @default(false)
  responses  Response[]
}

model Integration {
  id         String          @id @default(uuid())
  shopId     String
  shop       Shop            @relation(fields: [shopId], references: [id])
  type       IntegrationType
  configJson Json
  enabled    Boolean         @default(false)
}

enum IntegrationType { klaviyo flow webhook }
```

**Run migrations:** `prisma migrate dev --name init`

**Write a seed file** at `prisma/seed.ts` that creates one shop, one published survey with 3 questions and 2 audiences (new customer, returning customer).

### Acceptance Criteria
- [ ] `shopify app dev` starts without errors
- [ ] `prisma migrate dev` runs cleanly
- [ ] `prisma db seed` populates the DB with sample data
- [ ] `.env.example` documents every required variable

---

## Phase 2 — Shopify OAuth & Install Flow

### Prompt for Claude Code

Implement Shopify OAuth so merchants can install the app and land on a dashboard. No UI beyond a basic shell yet.

**What to build:**
- OAuth install and callback routes using `@shopify/shopify-api`. On successful OAuth, upsert the shop record in the DB with the access token.
- App uninstall webhook handler at `POST /webhooks/app/uninstalled`. On receipt, set `Shop.uninstalledAt = now()` and verify the HMAC signature before processing.
- A root dashboard route (`/app`) that:
  - Wraps content in the Polaris `AppProvider` and Shopify `AppBridgeProvider`.
  - Shows a top-level `Page` with a `Layout` containing four nav items: **Surveys**, **Audiences**, **Analytics**, **Integrations**.
  - Renders a placeholder `Card` for each section.
- Session storage using the DB (store session tokens in the `Shop` table, not in-memory).

**Do not build yet:** Survey builder UI, audience UI, analytics, or any integrations.

### Acceptance Criteria
- [ ] Clicking "Install" from a development store completes OAuth and lands on `/app`
- [ ] Uninstall webhook sets `uninstalledAt` on the shop record
- [ ] HMAC verification rejects requests with invalid signatures (test with a bad signature)
- [ ] Navigation renders all four sections without errors

---

## Phase 3 — Survey Builder (DB + API + UI)

### Prompt for Claude Code

Build the complete survey builder: creation, editing, and publishing. This is end-to-end — DB reads/writes, API routes, and the full UI.

**API routes to implement:**
- `GET /api/surveys` — list all surveys for the current shop (id, title, status, question count, created date)
- `POST /api/surveys` — create a new survey (title, status: draft)
- `GET /api/surveys/:id` — fetch a survey with all questions, choices, and audience
- `PUT /api/surveys/:id` — update title, status, questions, choices, follow-up logic, and promo action config
- `DELETE /api/surveys/:id` — soft-delete (set status to archived)

All routes must be scoped to the authenticated shop. Return 403 if the survey belongs to a different shop.

**UI screens to implement:**

*Survey list page (`/app/surveys`):*
- `ResourceList` of surveys showing title, status badge, question count, and created date.
- **Create survey** button opens a modal to enter a title and pick a starter template or blank. Templates: "How did you hear about us?" (3 pre-filled radio questions) and "Why did you buy?" (2 pre-filled radio questions + 1 text question). Creating from a template pre-populates the questions.
- Each row links to the survey edit page.

*Survey edit page (`/app/surveys/:id`):*
- Editable title at the top.
- Ordered list of questions. Each question card shows: prompt text, question type badge, required toggle, position controls (up/down), and a delete button.
- **Add question** button opens a modal:
  - Question type selector: Radio, Checkbox, Short Text, Long Text.
  - Prompt text input.
  - Required toggle.
  - For Radio and Checkbox: repeating choice input rows with add/remove. Each choice has a label and an auto-slugified value.
  - For Radio/Checkbox: a **Follow-up logic** section. Toggle "Only show if previous answer is..." then select the parent question (dropdown of existing questions) and the specific answer value that triggers it.
- **Promo code action** card below questions:
  - Toggle to enable/disable.
  - When enabled: discount percentage input, expiry days input, auto-apply toggle.
- **Validation before save:**
  - All questions must have a non-empty prompt.
  - Radio/Checkbox questions must have at least 2 choices.
  - Starter plan enforces max 3 questions — show an inline error and disable Save if exceeded.
  - Follow-up logic must reference a valid parent question.
- **Save** button (saves as draft). **Publish** button (sets status to `published`, only enabled if validation passes and at least 1 question exists).
- Status badge in the header updates live.

**Do not build yet:** Audience assignment inside the survey (that comes in Phase 4). Show a placeholder "Audience: Not set" card on the edit page.

### Acceptance Criteria
- [ ] Can create a blank survey and a survey from each template
- [ ] Can add, reorder, and delete questions of all 4 types
- [ ] Radio/Checkbox questions enforce minimum 2 choices
- [ ] Follow-up logic saves and re-loads correctly on page refresh
- [ ] Promo code action config saves and re-loads correctly
- [ ] Starter plan limit of 3 questions blocks publishing
- [ ] Publish sets status to `published` and the badge updates
- [ ] Soft-delete removes survey from the list but data persists in DB

---

## Phase 4 — Audience Builder (DB + API + UI)

### Prompt for Claude Code

Build the audience builder and connect audiences to surveys.

**API routes to implement:**
- `GET /api/audiences` — list audiences for the shop
- `POST /api/audiences` — create an audience rule
- `PUT /api/audiences/:id` — update
- `DELETE /api/audiences/:id` — delete

**UI screens to implement:**

*Audiences list page (`/app/audiences`):*
- `ResourceList` showing audience name (auto-generated from rule: e.g., "New customers"), attribute, operator, value.
- **Create audience** button.

*Create/Edit audience form (inline or modal):*
- **Attribute** selector: "Lifetime orders" or "Product ID".
- **Operator** selector (options change based on attribute):
  - Lifetime orders: "equals", "greater than", "less than"
  - Product ID: "is one of"
- **Value** input (text; for Product ID allow comma-separated IDs).
- Auto-generated display name shown as a preview (e.g., "Lifetime orders = 1" → "New customers").

*Connect audience to survey:*
- On the survey edit page (Phase 3), replace the "Audience: Not set" placeholder with a `Select` dropdown populated from the shop's audiences.
- Selecting an audience saves `audience_id` on the survey.
- Show a plain-language summary: e.g., "This survey will show to customers whose lifetime orders = 1."

**Audience matching logic (server-side utility function):**

Write a `matchesAudience(order: OrderData, audience: Audience): boolean` function used by the checkout extension in Phase 5. Implement:
- `lifetime_orders eq N` — customer's lifetime order count equals N
- `lifetime_orders gt N` — greater than N
- `lifetime_orders lt N` — less than N
- `product_id in [...]` — any line item variant ID is in the list

### Acceptance Criteria
- [ ] Can create audiences for new customers (lifetime_orders eq 1) and returning (lifetime_orders gt 1)
- [ ] Can create a Product ID audience with multiple comma-separated IDs
- [ ] Survey edit page shows the audience selector and saves the link
- [ ] `matchesAudience()` unit tests pass for all four operator types

---

## Phase 5 — Checkout UI Extension (Survey Widget)

### Prompt for Claude Code

Build the Shopify Checkout UI extension that renders the survey on the Thank You page.

**Extension setup:**
- Create the extension at `/extensions/post-purchase-survey/` using `shopify app generate extension` with type `checkout_ui_extension`.
- Register it on the `purchase.thank-you.block.render` extension point.
- Configure `extension.config.toml` with the correct capabilities.

**Extension behaviour:**

*On mount:*
1. Call `OrderConfirmationApi` to get the order ID, customer ID (if available), and line item variant IDs.
2. POST to `/api/extension/survey-config` with `{ shopDomain, orderId, customerId, variantIds, lifetimeOrderCount }`. The backend returns either the active survey config (questions, choices, logic) or `null` if no survey matches.
3. If `null`, render nothing (empty fragment).

*Rendering:*
- Render a `BlockStack` inside a `Card` with a heading ("Quick question for you 👋" or merchant-customised text).
- Show one question at a time. For Radio/Checkbox: render `ChoiceList`. For text: render `TextField`.
- "Next" button is disabled until an answer is provided for required questions.
- Apply follow-up logic: after each answer, determine the next question to show (skip questions whose parent answer condition isn't met).
- Progress indicator (e.g., "Question 2 of 3").

*On survey completion:*
- POST all answers to `/api/extension/responses` with `{ surveyId, orderId, customerId, audienceId, answers: [{ questionId, choiceValue?, textAnswer? }] }`.
- The backend returns `{ promoCode?: string, promoUrl?: string }`.
- If a promo code exists, show a `Banner` with the code and a `Button` ("Apply discount") that opens `promoUrl` in a new tab.
- If no promo code, show a simple thank-you `Text` block.

*Events to emit (fire-and-forget POST to `/api/extension/events`):*
- `survey_impression` on mount (if a survey is returned)
- `survey_started` on first answer
- `question_answered` on each answer
- `survey_submitted` on completion

**Backend endpoints for the extension:**
- `POST /api/extension/survey-config` — authenticate via shop domain + HMAC. Run `matchesAudience()` against the request data. Return survey config or `null`.
- `POST /api/extension/responses` — save the `Response` and `ResponseItem` rows. If the survey has a promo action, enqueue a `generatePromoCode` job (Phase 6) and return a pending state. If code is already generated synchronously (simple path for MVP), return it immediately.
- `POST /api/extension/events` — write event rows to an `events` table (add this table to the schema: `id, shopId, surveyId, responseId?, eventName, payload JSON, createdAt`).

### Acceptance Criteria
- [ ] Extension renders on the Thank You page only when the customer matches the audience
- [ ] Extension renders nothing for non-matching customers
- [ ] All 4 question types render correctly
- [ ] Follow-up logic skips the correct questions
- [ ] Answers POST correctly and are saved to `response_items`
- [ ] `survey_impression`, `survey_started`, `question_answered`, `survey_submitted` events are recorded
- [ ] Promo code banner renders and the apply button opens the correct URL

---

## Phase 6 — Promo Code Generation (Queue + Shopify Discounts API)

### Prompt for Claude Code

Implement asynchronous promo code generation using BullMQ and the Shopify Discounts API.

**Queue setup:**
- Create a Redis connection using `ioredis` with the `REDIS_URL` env variable.
- Create a BullMQ `Queue` named `promoCodeQueue` and a corresponding `Worker`.
- The worker file runs as a separate process (`workers/promoCodeWorker.ts`). It must be started alongside the main app server.

**Job: `generatePromoCode`**

Job payload: `{ shopId, surveyId, responseId, discountPercent, expiryDays }`

Worker steps:
1. Load the shop's `accessToken` from DB.
2. Call the Shopify Admin GraphQL API mutation `discountCodeBasicCreate` to create a single-use percentage discount:
   - No minimum purchase requirement.
   - Cannot be combined with other discounts.
   - `usageLimit: 1`, `appliesOncePerCustomer: true`.
   - Expiry = `now + expiryDays` (if set).
3. On success: save the code to `promo_codes`, link it to the response (`Response.promoCodeId`), and emit a `promo_code_generated` event.
4. On failure: retry with exponential backoff (5s, 30s, 2 min). After max retries, log the failure and emit a `promo_code_generation_failed` event. Surface this in the admin dashboard (a simple flag on the response record is sufficient for now).
5. Use an idempotency key (responseId) to prevent duplicate code generation if the job is retried.

**Surfacing the code to the customer:**
- After the worker saves the code, update the response record.
- The checkout extension's `/api/extension/responses` endpoint should poll or use a short delay + re-fetch to return the code. For MVP: generate synchronously in the request handler if the queue is too slow, and fall back to the queue for high-load scenarios.

**Promo URL format:** `https://{shopDomain}?discount={code}` — Shopify auto-applies the discount when the customer visits this URL.

### Acceptance Criteria
- [ ] A generated code appears in the Shopify admin under Discounts
- [ ] The code is single-use and has no minimum purchase requirement
- [ ] The code is saved to `promo_codes` and linked to the response
- [ ] Retries fire on failure (test by temporarily revoking the API token)
- [ ] Duplicate jobs for the same `responseId` do not create duplicate codes
- [ ] The promo URL correctly applies the discount when opened in a browser

---

## Phase 7 — Analytics Dashboard (DB + Loader + UI)

> **This is the most important phase. Build it thoroughly.**

### Prompt for Claude Code

Build the full analytics dashboard. The UI file already exists — your job is to build the data layer that feeds it correctly.

**Place the provided UI file at:** `app/routes/app.analytics.tsx`

The file is complete. Do not modify the UI components, types, or layout. Your only job is to replace the mock data in the `loader` function with real DB queries, and to build the CSV export endpoint.

---

### Step 7a — Understand the data contract

The loader returns a `LoaderData` object. Every field must come from real DB queries. Here is what each field means:

```ts
type LoaderData = {
  features: {
    enableTrends: boolean;      // Always false for MVP
    enableAttribution: boolean; // Always false for MVP
  };
  timeRange: { start: string; end: string }; // YYYY-MM-DD from URL params, default last 30 days
  surveys: SurveyOption[];      // All published surveys for this shop: { id, title }
  audiences: AudienceOption[];  // All audiences for this shop: { id, name }
  selected: {
    surveyId: string;           // From URL param, default to most recently created published survey
    audienceId: string | "all"; // From URL param, default "all"
  };
  summary: SummaryMetrics;      // Aggregate counts — see definition below
  questions: QuestionReport[];  // Per-question breakdown — see definition below
};
```

**`SummaryMetrics` — how to compute each field:**

| Field | DB query |
|---|---|
| `totalViews` | Count of `events` rows where `eventName = 'survey_impression'`, scoped to `surveyId` + date range |
| `totalResponses` | Count of `events` rows where `eventName = 'survey_started'`, scoped to `surveyId` + date range |
| `totalSubmissions` | Count of `Response` rows where `completedAt` is within the date range and `surveyId` matches |
| `responseRate` | `totalResponses / totalViews` (return 0 if totalViews = 0) |
| `completionRate` | `totalSubmissions / totalResponses` (return 0 if totalResponses = 0) |

If `audienceId` is not `"all"`, additionally filter all counts by `Response.audienceId = audienceId`. For events that don't carry `audienceId`, join through the `responses` table.

**`QuestionReport[]` — how to compute each question:**

For each question in the selected survey (ordered by `position`):

```ts
type QuestionReport = {
  id: string;         // question.id
  prompt: string;     // question.prompt
  typeLabel: string;  // "Radio" | "Checkbox" | "Short text" | "Long text"
  metrics: MetricsRow[];
  trends?: TrendPoint[];      // Leave undefined for MVP (enableTrends = false)
  attribution?: AttributionRow[]; // Leave undefined for MVP (enableAttribution = false)
};
```

**`MetricsRow[]` — how to compute:**

For `radio` and `checkbox` questions:
- Group `response_items` by `choiceValue` where `questionId` matches and the parent response's `completedAt` is within the date range.
- For each `choiceValue`, join to `choices` to get the `label`.
- `count` = number of `response_items` with that `choiceValue`.
- `percent` = `count / totalAnsweredForThisQuestion` as a float 0–1.
- Sort by `count` descending.
- `totalAnsweredForThisQuestion` = count of distinct `responseId` values in `response_items` for this question within the date range.

For `text_short` and `text_long` questions:
- Return the 20 most recent `textAnswer` values as `MetricsRow[]` where `label = textAnswer`, `count = 1`, `percent = 0`.
- These are rendered as a plain list in the UI (the `MetricsPanel` component handles this gracefully).

---

### Step 7b — Implement the loader

Replace the mock data in the existing `loader` function in `app/routes/app.analytics.tsx` with real Prisma queries. The function signature and return type stay identical.

```ts
export async function loader({ request }: LoaderFunctionArgs) {
  // 1. Authenticate the shop (use Shopify session from App Bridge)
  // 2. Parse URL params: start, end, surveyId, audienceId
  // 3. Default start = today - 30 days, end = today (YYYY-MM-DD strings)
  // 4. Load surveys (published only) for the shop
  // 5. Default surveyId = most recently created published survey
  // 6. Load audiences for the shop
  // 7. Compute summary metrics via Prisma queries (see Step 7a)
  // 8. Compute QuestionReport[] for the selected survey (see Step 7a)
  // 9. Return the LoaderData object
}
```

**Query performance requirements:**
- Do not run N queries for N questions. Use a single grouped query on `response_items` for all questions in the survey, then distribute results in-memory.
- Index required (add to schema migration if not already present): `response_items.questionId`, `responses.surveyId + completedAt`, `events.surveyId + eventName + createdAt`.

---

### Step 7c — CSV export endpoint

Create a new route at `app/routes/app.exports.responses[.csv].tsx`.

**Accepts:** same query params as the loader — `surveyId`, `audienceId`, `start`, `end`.

**Returns:** a streaming CSV response with `Content-Type: text/csv` and `Content-Disposition: attachment; filename="responses.csv"`.

**Columns:**
```
response_id, order_id, customer_id, completed_at, [one column per question, using question prompt as header]
```

For radio/checkbox questions: the `choiceValue` (or the choice `label` — use label for readability).
For text questions: the `textAnswer`.
If a question was not answered for a given response, leave the cell empty.

**Implementation note:** Stream the CSV row by row using a `ReadableStream` or Remix's `Response` with a generator. Do not load all responses into memory at once — use Prisma cursor-based pagination with batches of 500 rows.

---

### Step 7d — Empty and loading states

The UI file uses `SkeletonBodyText` from Polaris. Wire this up correctly:

- Use Remix's `useNavigation` hook in the route. When `navigation.state === "loading"`, render `SkeletonBodyText` in place of the summary cards and question panels. The filter bar should remain interactive.
- Empty state for no surveys: render a Polaris `EmptyState` with a "Create your first survey" action linking to `/app/surveys/new`.
- Empty state for no responses in date range: render the summary cards with all zeros and a `Banner` with `tone="info"` saying "No responses in this period. Try widening the date range."

---

### Acceptance Criteria
- [ ] `totalViews`, `totalResponses`, and `totalSubmissions` show correct counts against real data in the seeded DB
- [ ] `responseRate` and `completionRate` are calculated correctly (no division-by-zero crash)
- [ ] Changing the survey selector reloads the page with the new `surveyId` param and updates all data
- [ ] Changing the audience filter correctly filters summary counts and per-question metrics
- [ ] Date range picker applies `start` and `end` params and updates all data
- [ ] Per-question `MetricsRow[]` for radio/checkbox questions are sorted by count descending and percentages sum to ~100%
- [ ] Per-question `MetricsRow[]` for text questions show the 20 most recent responses
- [ ] Trends and Attribution tabs render the "not enabled" placeholder (feature flags are `false`)
- [ ] CSV export downloads a valid file with correct headers matching question prompts
- [ ] CSV export streams without loading all rows into memory (verify with 1000+ seeded responses)
- [ ] Loading skeleton renders while Remix navigation is in-flight
- [ ] Empty state renders correctly when no surveys exist
- [ ] Empty state banner renders when date range returns zero responses
- [ ] No N+1 queries — confirm with Prisma query logging enabled during test

---

## Phase 8 — Klaviyo Integration (Queue + UI)

### Prompt for Claude Code

Build the Klaviyo integration: settings UI, per-question sync toggles, and a background worker that pushes data.

**Settings UI at `/app/integrations`:**
- `Card` for Klaviyo with:
  - API key input (password field, value masked after save).
  - List name input (optional — the Klaviyo list to add the customer to).
  - **Save** button. On save, validate the key by calling `GET https://a.klaviyo.com/api/lists/` with the key. Show a success or error banner.
  - Connected/Disconnected status badge.
- Below the connection settings: a list of the shop's published survey questions. Each question has a toggle "Sync to Klaviyo." Toggling updates `Question.syncToKlaviyo` in the DB.

**BullMQ job: `klaviyoSync`**

Job payload: `{ responseId, shopId }`

Worker steps:
1. Load the response, its items, and the questions where `syncToKlaviyo = true`.
2. Load the shop's Klaviyo API key from the `integrations` table.
3. Identify the customer: use `customerId` if available, otherwise skip (can't push without an identifier).
4. Build the Klaviyo profile update payload. Map each synced question to a custom property. Property name = question prompt (truncated to 60 chars, spaces replaced with underscores). Value = the answer.
5. Also push survey status as a property: `kno_survey_status = "completed"`, `kno_survey_title = <survey title>`, `kno_survey_completed_at = <ISO timestamp>`.
6. Call `PATCH https://a.klaviyo.com/api/profiles/{klaviyo_profile_id}` to update the profile.
7. On success: emit `klaviyo_sync_succeeded` event.
8. On failure after max retries: emit `klaviyo_sync_failed` event and surface it in the integrations page as a warning badge ("X sync failures in the last 24h").

**Trigger:** Enqueue a `klaviyoSync` job inside the `/api/extension/responses` handler, after the response is saved, if Klaviyo is enabled for the shop.

### Acceptance Criteria
- [ ] Entering an invalid Klaviyo API key shows an error banner
- [ ] Entering a valid key shows a "Connected" status badge
- [ ] Question sync toggles save correctly to the DB
- [ ] After a customer completes a survey, their Klaviyo profile is updated with the correct custom properties within ~30 seconds
- [ ] `kno_survey_status`, `kno_survey_title`, and `kno_survey_completed_at` appear on the Klaviyo profile
- [ ] Failed syncs are surfaced as a warning in the integrations UI
- [ ] Disabling the integration stops new sync jobs from being enqueued

---

## Phase 9 — Polish, Validation & Deployment

### Prompt for Claude Code

Harden the app, write tests, and deploy to production.

**Validations and edge cases:**
- If the merchant publishes a survey but has not added the KNO block to the Shopify Checkout editor, show a persistent warning banner on the dashboard: "Your survey is published but may not be visible. Make sure you've added the KNO block to your Thank You page in the Checkout editor."
- Handle guest checkouts: if `customerId` is null, store the response with only `orderId`. Skip Klaviyo sync (log that sync was skipped due to no customer ID).
- Handle the extension failing to load (network error): catch the error in the extension, render nothing, and do not emit events.
- Rate-limit `/api/extension/*` routes to 20 requests/sec per shop using an in-memory or Redis counter.

**Tests to write:**

Unit tests (Jest):
- `matchesAudience()` — all 4 operator types, edge cases (empty product ID list, lifetime_orders = 0)
- Survey validation — question limits, required fields, follow-up logic referencing deleted questions
- Promo code idempotency — calling `generatePromoCode` twice with the same `responseId` creates only one code
- Analytics aggregation — given a set of raw events, assert the correct summary counts

Integration tests (Supertest):
- `POST /api/extension/responses` — saves response items, enqueues jobs, returns correct shape
- `GET /api/analytics/overview` — returns correct counts for a seeded DB
- `GET /api/analytics/export` — returns valid CSV with correct columns and row count

End-to-end test (Playwright):
- Install the app on a development store, create a survey with 2 questions and a promo code action, assign a "new customer" audience, add the KNO block to the Thank You page, place a test order as a new customer, complete the survey, assert the promo code banner appears, assert the response is saved in the DB.

**Deployment:**
- Deploy to Fly.io or Railway. Set all env variables in the platform secrets manager.
- Run `prisma migrate deploy` (not dev) in production.
- Register webhooks: `app/uninstalled`, `orders/paid` (for future promo code redemption tracking).
- Deploy the checkout extension via `shopify app deploy`.
- Verify the extension appears in the Checkout editor block list.
- Run the Playwright e2e test against production.

### Acceptance Criteria
- [ ] Warning banner appears when no checkout block is detected
- [ ] Guest checkout responses save without crashing
- [ ] All unit tests pass
- [ ] All integration tests pass
- [ ] Playwright e2e test passes against production
- [ ] App is live at the production URL with all env variables set
- [ ] Extension is visible in the Shopify Checkout editor
- [ ] Klaviyo sync works end-to-end in production

---

## Appendix — Things to Verify Before Starting Phase 7

These were flagged as open questions in the spec. Check them before building analytics:

1. **Does KNO store partial survey responses?** Decide whether `Response` rows are created on `survey_started` or only on `survey_submitted`. If on `started`, you need a `status` field on `Response` (`started | completed`). This affects funnel accuracy.

2. **What counts as an "impression"?** If the extension mounts but the customer never interacts, does that count? Decide before building the aggregation job — it changes the `impressions` denominator.

3. **Promo code rate limits.** Before building Phase 6 at scale, verify the Shopify Discounts API rate limit (typically 2 discount code batch operations/sec). Add appropriate throttling in the worker if needed.

4. **Analytics period comparison.** The stat cards show trend vs. the previous period. Decide the comparison logic before building: previous N days (where N = selected range length) is the simplest and recommended approach.

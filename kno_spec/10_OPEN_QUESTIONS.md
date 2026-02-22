# Open Questions & Verification Steps

Items to clarify before finalising the MVP or planning V2.

| Question | Why it matters | How to verify |
|---|---|---|
| **How are Automated Insights and benchmarking calculated?** | Marketing copy mentions demographic/psychographic analysis and brand benchmarks, but no docs explain the methodology or whether third‑party data is used. | Contact KNO support or request a demo. Confirm what data is collected and whether user consent is required. |
| **Are there built‑in team roles and permissions?** | Multi‑user support may be needed for larger merchants; not mentioned in public docs. | Check the KNO admin UI after installation or contact support. |
| **Does KNO store partial survey responses?** | Affects analytics design and whether `survey_started` events need to persist incomplete state. | Install on a dev store, close the Thank You page mid‑survey, and check whether a partial response appears in the dashboard. |
| **Are there rate limits on promo code generation?** | Shopify's Discounts API may cap codes per minute, affecting the promo action design. | Review Shopify Discounts API docs and run load tests. |
| **How does KNO handle translations and localisation?** | App listing mentions multi‑language support but the mechanism (manual vs. automatic) is unclear. | Explore the builder for a locale selector or Shopify translation API integration. If absent, localisation likely requires duplicating surveys per language. |
| **What data points are sent to Triple Whale, Peel, and other analytics integrations?** | Needed to design compatible connectors in V2. | Inspect integration settings in the app or contact support for field mappings. |
| **Are there character limits on question prompts?** | Long prompts may not display well in the checkout extension. | Test with long text in a dev store; check for truncation or overflow. |
| **How are multiple languages handled in the checkout extension?** | The extension may or may not inherit the store's locale automatically. | Test the app on a store set to a non‑English language and observe behaviour. |

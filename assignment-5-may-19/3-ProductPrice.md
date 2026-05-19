`PRODUCT_PRICE`

---
### IDENTITY & LOOKUP

* **PRODUCT_ID** [core]
  The product being priced — always the primary join key
* **PRODUCT_PRICE_TYPE_ID** [core]
  What kind of price: LIST_PRICE, DEFAULT_PRICE, PROMO_PRICE etc. — determines which row wins
* **PRODUCT_PRICE_PURPOSE_ID** [core]
  PURCHASE vs COMPONENT — filters price to the right use case
* **CURRENCY_UOM_ID** [core]
  Multi-currency setups need this to pick the right row (USD, EUR...)
* **PRODUCT_STORE_GROUP_ID** [situational]
  Scopes price to a store group — important for multi-store, can be null for global

### VALIDITY WINDOW

* **FROM_DATE** [core]
  Price effective from — always filter WHERE FROM_DATE <= now
* **THRU_DATE** [core]
  Price expires at — filter WHERE THRU_DATE IS NULL OR THRU_DATE > now. Null = still active (your data)

### THE ACTUAL PRICE

* **PRICE** [core]
  Base price — the main value. Your rows all show 52
* **PRICE_WITHOUT_TAX** [situational]
  Explicit ex-tax price — useful when tax-inclusive pricing is configured
* **PRICE_WITH_TAX** [situational]
  Inclusive price — use for display in tax-inclusive storefronts
* **TAX_AMOUNT / TAX_PERCENTAGE** [situational]
  Stored tax breakdowns — populated when tax is baked into the price row

### ADVANCED / RARELY USED

* **TERM_UOM_ID** [situational]
  Pricing unit (per month, per year) — relevant for subscriptions, null for one-off
* **CUSTOM_PRICE_CALC_SERVICE** [situational]
  Points to a custom Groovy/service for dynamic pricing — usually null
* **TAX_AUTH_PARTY_ID / TAX_AUTH_GEO_ID / TAX_IN_PRICE**
  Tax authority config — only matters if OFBiz is doing tax calculation itself

### AUDIT TRAIL

* **CREATED_DATE / CREATED_BY_USER_LOGIN** [audit]
  Who created it and when
* **LAST_MODIFIED_DATE / LAST_MODIFIED_BY_USER_LOGIN** [audit]
  Last manual edit
* **LAST_UPDATED_STAMP / LAST_UPDATED_TX_STAMP / CREATED_STAMP / CREATED_TX_STAMP** [audit]
  OFBiz internal — transaction-level timestamps, not business logic

---
## Key observations:
- `THRU_DATE` is null on all rows → these are all currently active prices. Your query should always handle `THRU_DATE IS NULL OR THRU_DATE > NOW()`.
- All rows have `PRICE = 52` but the tax fields (`PRICE_WITHOUT_TAX`, `TAX_AMOUNT`, etc.) are empty → tax calculation is being handled elsewhere (likely at checkout), not stored in this table.
- The **composite key** for picking the right price row is: `PRODUCT_ID + PRICE_TYPE + PURPOSE + CURRENCY + STORE_GROUP + active date window`. Missing any of these in a query can return duplicate or wrong rows.

**Most common mistake** — querying just by `PRODUCT_ID` without filtering on `PRODUCT_PRICE_TYPE_ID = 'LIST_PRICE'` and the date window. You'll get multiple rows and won't know which one to use.

| Field Name | Type | Description |
|---|---|---|
| SHOP_ID | PK | Which Shopify store. Part of the composite key — always pair with PRODUCT_ID when querying. |
| PRODUCT_ID | PK | Internal OFBiz product ID (virtual or variant). The other half of the composite key. Duplicates in your data (10525, 100065) = same PRODUCT_ID mapped to two Shopify IDs. Re-import bug. |
| SHOPIFY_PRODUCT_ID | external | Shopify's parent product GID. Used to identify the virtual product on Shopify side during inbound dedup check. |
| SHOPIFY_INVENTORY_ITEM_ID | external | The ID inventory jobs push stock to via Shopify API. Null on rows 1–9 in your data — outbound sync silently skips these. Must check ?has_content in FTL, not ?exists — empty string passes exists check. |
| LAST_UPDATED_STAMP | audit | When this mapping row was last touched — business-level timestamp written by the service layer. |
| LAST_UPDATED_TX_STAMP | audit | OFBiz transaction-level timestamp. NaT on unmapped rows means the sync transaction never committed for them. |
| CREATED_STAMP | audit | When the mapping was first created. NaT = row was inserted without a completed sync (placeholder state). |
| CREATED_TX_STAMP | audit | Transaction timestamp at creation. Pair with CREATED_STAMP to confirm the write actually committed. |

------------------------------


Sure, quick run through:The table is intentionally lean — only 8 fields. The core payload is just 4: two PKs that say *which product in which store*, and two external IDs that say *where on Shopify*. Everything else is audit.

The most operationally dangerous field is `SHOPIFY_INVENTORY_ITEM_ID` — when it's null or empty, the product is invisible to all outbound sync jobs. That's the first thing to check when a product's inventory isn't updating on Shopify.

Ready to move to virtual + variant?
**product** in HotWax/OFBiz at the code level:
---

## A product is two separate `Product` rows, not one

There is no single "product" record. Every sellable item is split into:

**Virtual product** — the style/template parent
- `isVirtual = "Y"`, `isVariant = "N"`
- Has no physical inventory of its own
- Represents the concept: "Eco Hoodie"
- Holds `SELECTABLE_FEATURE` entries (the dropdowns: Size, Color)

**Variant product** — the actual SKU child
- `isVirtual = "N"`, `isVariant = "Y"`
- This is what gets ordered, reserved, shipped
- Holds `STANDARD_FEATURE` entries (the fixed specs: Red, Small)
- Has its own `ProductPrice`, `GoodIdentification`, `InventoryItem` linkage

The two are connected via a `ProductAssoc` row with `productAssocTypeId = "PRODUCT_VARIANT"`.

---

## The `Product` entity's key fields

| Field | What it means |
|---|---|
| `productId` | Internal primary key (e.g. `ECO_HOODIE_RED_S`) |
| `productTypeId` | What kind of product: `FINISHED_GOOD`, `MARKETING_PKG_PICK`, `DIGITAL_GOOD`, etc. |
| `internalName` | The SKU string — used as the uniqueness check during sync |
| `isVirtual` | `Y` = style parent, no inventory |
| `isVariant` | `Y` = physical SKU child |

---

## What hangs off a product

A product row alone means almost nothing. The full picture requires:

- **`ProductAssoc`** — links variant to virtual parent (`PRODUCT_VARIANT`), or component to bundle (`PRODUCT_COMPONENT`)
- **`GoodIdentification`** — external identifiers: `SKU`, `UPCA` (barcode), `SHOPIFY_PROD_ID`
- **`ProductFeatureAppl`** — which features apply to it (`SELECTABLE_FEATURE` on virtual, `STANDARD_FEATURE` on variant)
- **`ProductPrice`** — pricing with `fromDate`/`thruDate` for temporal history
- **`ProductCategoryMember`** — which catalog categories it belongs to
- **`ShopifyShopProduct`** — maps the internal `productId` to Shopify's `shopifyProductId`, `shopifyVariantId`, `shopifyInventoryItemId`
- **`InventoryItem`** — only on variants, never on the virtual parent

---

## Product types that behave differently

| `productTypeId` | Behavior |
|---|---|
| `FINISHED_GOOD` | Normal physical SKU |
| `MARKETING_PKG_PICK` | Bundle/kit — no direct inventory, explodes into `PRODUCT_COMPONENT` children at fulfillment |
| `DIGITAL_GOOD` | No physical inventory |
| `GIFT_CARD` | Special handling |
| `SERVICE` | No inventory |

The `MARKETING_PKG_PICK` type is set during mapping when the Shopify payload has `requiresComponents = true`. Its ATP is calculated as the minimum of its components' ATP divided by their quantity ratios — it never holds stock itself.

---

## The identity of a product during sync

When a product arrives from Shopify, the system doesn't use `productId` to find it. It uses `internalName` (the SKU string) as the match key, falling back to `GoodIdentification` lookups. `productId` is internal only. This is why `GoodIdentification` and `ShopifyShopProduct` are the two most critical tables when debugging missing or duplicate products — the product row can exist perfectly but be invisible to sync if those mapping rows are missing.
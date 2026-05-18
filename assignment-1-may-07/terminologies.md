# `productTypeId` values 

## `FINISHED_GOOD`

This is a **regular, standalone, sellable product** — a single physical item that exists on its own.

**Example:**
A Red Hoodie in Size S → it's one item, it ships as one item, inventory is tracked directly on it.

In the context of the document, a **variant** like `TEST-HOODIE-RED-S` is a `FINISHED_GOOD` because:
- It has its own SKU
- It has its own inventory count
- When ordered, exactly that one item gets picked and shipped

---

## `MARKETING_PKG_PICK`

This is a **bundle or kit** — a product that is sold as one unit on the storefront but is **physically made up of multiple component items** underneath.

The "PICK" part means the warehouse literally **picks individual components** to fulfill the order, not a pre-packed box.

**Example:**
A "3-Pack of Socks" listed as one product on Shopify, but in the warehouse it means picking 3 individual sock units.

So when an order comes in for a `MARKETING_PKG_PICK`:
- The system doesn't reserve inventory on the bundle itself
- It **explodes** it into its components via `PRODUCT_COMPONENT` associations
- Then reserves inventory on each individual component separately

---

## How the Code Decides Which One

From the mapping service `map#Product`, the decision is made here:

```
if payload.requiresComponents == true
    → productTypeId = MARKETING_PKG_PICK
else
    → productTypeId = FINISHED_GOOD
```

So Shopify tells the system via the `requiresComponents` flag whether a product is a simple item or a bundle that needs component explosion during fulfillment.

---

## Key Difference in a Table

| | `FINISHED_GOOD` | `MARKETING_PKG_PICK` |
|---|---|---|
| What it is | Single sellable item | Bundle / Kit |
| Inventory tracked on | The product itself | Its components |
| On order | Reserve directly | Explode into components first |
| Shopify flag | `requiresComponents = false` | `requiresComponents = true` |
| Example | Red Hoodie S | 3-Pack Socks |
---
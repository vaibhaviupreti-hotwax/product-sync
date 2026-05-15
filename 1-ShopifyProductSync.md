# Shopify Product Import + Sync (BASICS ONLY)
[Basics, no technical stuff]
- “Product import” and “product sync” are just tools to help you move and manage all those products without going crazy.

## 1. What is “product import” in plain language?
“Bringing a bunch of products into your store all at once, instead of typing them in one by one.”

### Product ImportReal-World AnalogyScenario: 
You open a physical shop.
Supplier: Gives you a massive spreadsheet of inventory.
Manual approach: Writing each item on physical index cards.
Import approach: Handing the list to an assistant for automated entry.
System role: The software acts as that automated assistant.

### Why Use Product Import?
Speed: Bulk uploads save hours compared to manual creation.
Consistency: Reuses organized supplier data without manual typos.
Bulk updates: Modifies prices, descriptions, and stock instantly.
Platform migration: Moves catalog data seamlessly between different e-commerce systems.

---
## 2. Product Sync## Plain Language Definition

* Core concept: Keeping information matching and up-to-date between multiple locations.
* The distinction: Import is a one-time move; sync is a continuous conveyor belt of updates.

-  Real-World Analogy

* The setup: You have a warehouse tracking system and an online storefront.
* The trigger: The warehouse sells 3 items, leaving 7 in stock.
* The action: Sync forces the storefront to update immediately to 7.
* The goal: Ensures both systems stay in absolute agreement.

------------------------------
To help complete your notes, tell me if you want to proceed with:

* The additional info you mentioned next
* A comparison table of Import vs. Sync
* Common triggers that start a sync process

---

## 3. Why does product sync exist? What’s its purpose?
At a basic, human level, product sync exists to solve one huge problem:

“I don’t want to manually update everything, everywhere, every time something changes.”

Key purposes of product sync
Avoiding mistakes and mismatches
Without sync, you might have:

Shopify saying “In stock: 20”
Your warehouse saying “In stock: 5”
Your marketplace (like Amazon) saying “In stock: 0”
This:

Confuses customers
Can cause overselling (selling things you don’t actually have)
Makes you look unreliable
Sync’s job is to keep all those numbers in line with each other.

Saving time
Without sync, you’d have to:

Log into Shopify → update product
Log into your warehouse system → update product
Log into marketplaces (e.g. Amazon, eBay) → update product
Log into POS (in-person store system) → update product
Every time:

Price changes
Stock changes
A product description changes
Sync automates this. You change it in one place, and sync spreads that change everywhere else.

Centralizing your truth
Most businesses want one “main” place that is the source of truth for product information. For example:

“Shopify is our main system; everything else copies from there.”
or
“The warehouse system is the main source; Shopify reads from it.”
Product sync is the bridge that takes the information from that main source and copies it to other systems, so they all match.

Handling multiple sales channels
Modern stores don’t just sell in one place. You might sell:

On your online store
In a physical store
On marketplaces (Amazon, eBay)
On social media (Instagram Shopping, Facebook Shop)
All these channels might show the same product. Without sync:

Updating all of them becomes a nightmare.
With sync:

You can change a price once and it flows out to all those places.

## 4. Basic difference: import vs sync
Let’s boil it down as simply as possible:

Product import
Mostly a one-time or occasional bulk action
You take a file or list of products and bring them into your store
Think: “Add or update lots of products at once”
Product sync
Usually a continuous or repeated action
Automatically keeps products matching between two or more systems
Think: “Keep everything up to date, everywhere”
A store might:

Use import to get all products into Shopify for the first time
Then use sync to:
Keep stock levels correct
Keep prices consistent
Keep descriptions and images aligned

## 5. Deep dive into the basics (still non-technical)
Let’s go a little deeper into the idea level—how to “think” about these things, without any technical details.

### 5.1. What kind of information is being imported or synced?
When you hear “product data,” think of it as a profile for each item you sell. That profile usually includes:

Name – what you call it (e.g. “Blue Cotton T-Shirt”)
Description – how you explain it to the customer
Price – how much it costs
SKU / Code – a unique code that identifies it (like a license plate for each product)
Stock quantity – how many you have right now
Variants – different sizes, colors, etc. for the same basic product
Images – photos of the product
Category / collections – where it appears in your store (“Men’s Shirts”, “New Arrivals”)
Import loads or updates these profiles in bulk.
Sync makes sure these profiles match across systems.

### 5.2. One-way vs two-way thinking
When you think about sync, it helps to think in terms of direction:

One-way:

“My warehouse tells Shopify what the stock is”
So the warehouse is the boss, and Shopify just follows.
Two-way:

“If stock changes in Shopify or in the warehouse, both sides get updated”
They’re both allowed to “speak”, and sync keeps them aligned.
You don’t need the technical details, but understanding the idea of direction helps you reason about who is in charge of product information.

### 5.3. When do you use import vs sync in your mind?
Think in terms of situations:

Situation 1: Launching your store
You get a big product spreadsheet from your supplier.
You want your store filled quickly.
Tool in your head: “This is a product import moment.”
Situation 2: Changing prices on a big sale
You’re running a Black Friday sale.
You drop prices on 300 products.
Doing it manually one by one is too slow.
Tool in your head: “Bulk update via import.”
Situation 3: Selling across multiple channels
You sell on:

Shopify online store
A physical boutique
Amazon marketplace
You don’t want to log into each one to keep stock updated.

Tool in your head: “This is a product sync moment.”

Situation 4: Working with a supplier or dropshipper
Your supplier can send you regular updates:
Which products are available
New products
Products that are discontinued
Instead of manually keeping up:
Tool in your head: “Either regular imports or an ongoing sync.”
### 5.4. Risks if you don’t understand these basics
Again, no technical talk—just practical consequences.

Overselling
If stock doesn’t sync and your store thinks you have 10 when you only have 2, you’ll sell products you don’t actually have, which leads to:

Refunds
Angry customers
Bad reviews
Underselling
If one system says “0 left” while the warehouse has 50, you’ll stop selling products you could still make money from.

Inconsistent prices
One channel says $20, another says $25:

Customers lose trust
You have to honor the lower price or deal with complaints
Time drain
Without import or sync:

Every new product = lots of manual work
Every update = repeated tasks across systems
The more you grow, the worse it gets
Product import and sync exist to avoid all of this.

## 6. How to think about these concepts if you’re not technical
Here are some non-technical mental models that might help.

### 6.1. Think in terms of “lists” and “mirrors”
Import = “Take this list and load it into my shop.”
Sync = “Make sure my shop is a mirror of that list (or that those two lists are mirrors of each other).”
If the list updates regularly and you want the shop to reflect it, you’re thinking about sync.

### 6.2. Think in terms of “one source of truth”
Ask yourself:

“Where do I want the real product information to live?”

Possibilities:

Your online store
Your warehouse system
Your supplier’s catalog
Once you pick that main source, then:

Import is how you initially copy that truth into Shopify
Sync is how you keep that truth updated in Shopify and other places

### 6.3. Think in terms of “how often does this change?”
If it’s a big one-time setup → import mindset
If it’s changing daily or constantly (stock, prices, etc.) → sync mindset

## 7. Recap in ultra-simple language
Product import

You hand your store a big list of products.
It creates or updates them all at once.
You use it when you want to load or change many products together.
Product sync

You have product information in more than one place (Shopify, warehouse, marketplaces).
You want them all to show the same, up-to-date info without manually editing each one.
You use it to keep things matching and current everywhere.
If you tell me a bit more about your situation—like:

Are you starting a new store or improving an existing one?
Do you sell in multiple places (online + physical, or multiple platforms)?
Do you have a supplier or warehouse that tracks stock?
— I can walk through what product import and product sync would look like specifically for your setup, still without going into technical details.
---
# Shopify Product Sync (ADVANCED)
Product sync between Shopify and an OMS
— what’s involved, which APIs and patterns exist, and the practical considerations. 
Products (REST): https://shopify.dev/docs/api/admin-rest/latest/resources/product
Products (GraphQL): https://shopify.dev/docs/api/admin-graphql/latest/objects/Product
Variants (GraphQL): https://shopify.dev/docs/api/admin-graphql/latest/objects/ProductVariant
Inventory (REST & GraphQL):
https://shopify.dev/docs/api/admin-rest/latest/resources/inventoryitem
https://shopify.dev/docs/api/admin-rest/latest/resources/inventorylevel
https://shopify.dev/docs/api/admin-graphql/latest/objects/InventoryItem
https://shopify.dev/docs/api/admin-graphql/latest/objects/InventoryLevel
Bulk operations: https://shopify.dev/docs/api/usage/bulk-operations

## 1. What “product sync” usually means
When you say “product sync” between Shopify and an OMS, you’re normally dealing with:

Product catalog sync

Product basics: title, description, productType (e.g. MARKETING_PKG_AUTO), vendor, tags.
Options (Size, Color, etc.).
Variants: SKU, price, barcode, weight, tax/ship flags.
Images.
Possibly metafields.
Inventory sync

Available quantities per variant per location.
Whether inventory is managed by Shopify or externally.
Price sync

Base price per variant.
Sometimes compare‑at price, cost.
State sync

Product status (ACTIVE/DRAFT/ARCHIVED).
Whether a variant is available to sell.
The OMS usually:

Pulls this data from Shopify to build its own product master and SKUs.
Optionally pushes changes back (prices, inventory, etc.), depending on who is the “source of truth.”

## 2. Shopify data model you’re syncing
### 2.1 Product
Key object: Product (GraphQL) / Product (REST)

Typical important fields:

id (GID) / numeric ID
title
descriptionHtml / bodyHtml
handle
status (ACTIVE, DRAFT, ARCHIVED)
productType (free-form string; custom like MARKETING_PKG_AUTO is allowed)
vendor
tags
options (e.g. Size, Color)
images

### 2.2 ProductVariant
Key object: ProductVariant (GraphQL)

Important fields:

id / numeric variant ID
sku (critical for OMS)
barcode
title (composed from options)
price, compareAtPrice
weight, weightUnit
taxable
requiresShipping
inventoryItem (link to inventory data)
selectedOptions (e.g. {name: "Size", value: "M"})
### 2.3 Inventory
Inventory is separate from products/variants:

An SKU/variant has an associated InventoryItem.
Availability at each location is an InventoryLevel.
Key objects:

InventoryItem (GraphQL)
InventoryLevel (GraphQL)
InventoryItem (REST)
InventoryLevel (REST)
Important fields:

InventoryItem.id, sku (via variant), tracked
InventoryLevel.location, available, updatedAt
Location.id, name (from Location object)

## 3. Mechanisms for product sync
### 3.1 Pulling products via Admin REST API
REST endpoints:

List products: GET /admin/api/202x-xx/products.json
Docs: https://shopify.dev/docs/api/admin-rest/latest/resources/product
Features:

Paginated (page-based or since 2022-01+ usually “page_info” cursor-based).
Returns product and variants in nested JSON.
Good for:
Smaller stores.
Occasional full syncs or nightly jobs.
Limitations:

Need to manage pagination & rate limits yourself.
Can be slow for very large catalogs.
### 3.2 Pulling products via Admin GraphQL API
You can query products with connections:

Query: products(first: X, after: "cursor") { edges { node { ... }}}
Docs:

Product object
Using GraphQL connections/pagination
Features:

More flexible field selection.
Can fetch nested data (variants, images, metafields) in one query.
Still synchronous & paginated.

### 3.3 Bulk Operations for large catalogs
For large stores, use Bulk Operations:

You run a mutation like bulkOperationRunQuery with a GraphQL query selecting:
products + nested variants, inventoryItem, inventoryLevels, etc.
Shopify processes it asynchronously.
You poll currentBulkOperation until status == COMPLETED.
Then you download the JSONL file from url.
Docs:

Bulk operations overview: https://shopify.dev/docs/api/usage/bulk-operations
Bulk queries: https://shopify.dev/docs/api/usage/bulk-operations/queries
Advantages:

No manual pagination.
Handles very large catalogs.
Results in a JSONL file, which you process line by line.
Usage pattern:

Use Bulk Operations for:
Initial full product sync.
Periodic full/partial resyncs.
## 4. Keeping products in sync over time
### 4.1 Initial full sync
Usual approach:

Use Bulk Operations to export all products (and optionally variants + inventory).
Download the JSONL.
Convert JSONL → structured JSON or directly into your OMS data model.
Upsert products and variants into the OMS.
This gives your OMS a full baseline.

### 4.2 Incremental sync / updates
After the baseline, you need to track changes:

Typical strategies:

Product webhooks

Subscribe to:
products/create
products/update
products/delete
Docs: Webhook topics
On each event:
Fetch latest product data via REST/GraphQL, or
Use the payload directly (for simple mappings).
Upsert into OMS.
Variant-level webhooks (if needed)

variants/create, variants/update, variants/delete also exist.
Inventory webhooks

inventory_levels/update for inventory changes.
Docs: https://shopify.dev/docs/api/admin-rest/latest/resources/webhook#event-topics-InventoryLevel
Periodic incremental bulk (optional)

For safety, you can run periodic bulk queries:
“Products updated since X” (if your query can filter by updatedAt) to catch missed webhook deliveries.
### 4.3 Direction of truth
You must decide which system is the source of truth for each area:

Product master data (name, description, type, etc.):
Shopify as master? Then OMS mostly reads from Shopify.
OMS/ERP as master? Then OMS might push changes back via REST/GraphQL.
Inventory:
Shopify-managed or OMS/warehouse-managed?
If OMS is master, it will:
Update InventoryLevel in Shopify.
Possibly ignore Shopify inventory webhooks or treat them as confirmations.
## 5. Handling JSONL for product sync (Bulk Operations)
When you export products with Bulk Operations:

You get JSONL where each line is a JSON object (product, variant, inventory item, etc.).
You need to:
Read the file line by line.
Parse each line as JSON.
Use fields like __typename and __parentId to reconstruct:
Product → Variants → Inventory relationships.
Typical entity types in JSONL:

Product
ProductVariant
InventoryItem
InventoryLevel
Possibly Metafield
Example logic:

When __typename == "Product", create a new product in your local structure.
When __typename == "ProductVariant", attach it to its parent product (via __parentId or product ID).
When __typename == "InventoryLevel", attach to the correct variant/InventoryItem and location.
After reconstruction, you map to your OMS product format.

## 6. Common fields to map into an OMS
### 6.1 Product-level fields
Shopify product ID (GID + numeric)
Title
Handle
Description (plain-text or HTML)
Product type (e.g. MARKETING_PKG_AUTO)
Vendor/brand
Tags
Status (active/draft/archived)
Created/updated timestamps
Product-level images
Options (Size, Color, Material, etc.)

### 6.2 Variant-level fields (per SKU)
Variant ID(s)
SKU (critical)
Barcode
Variant title
Option values (Size = M, Color = Red)
Price / compare-at price
Cost per item (if used in OMS margins)
Weight + units
Taxable flag
Requires shipping flag
Created/updated timestamps
Variant-specific images (if any)

### 6.3 Inventory fields
Inventory management:
Whether stock is tracked (Shopify vs external).
Total available quantity.
Per-location availability:
Location ID, name.
Available quantity.
Last updated timestamp (for conflict resolution).
### 6.4 Metafields / attributes
Any custom attributes that affect OMS logic:
e.g., marketing flags, packaging instructions, dropship flags, etc.
Mapped from Shopify Metafields into OMS attributes or custom fields.

## 7. Common product-sync challenges & patterns

### 7.1 SKU uniqueness and conflicts
Ideal: SKUs are unique across all variants.
Reality: Some stores re-use SKUs or have missing SKUs.
In an OMS context:
Prefer to use variant ID as technical key.
Use SKU as business key if (and only if) it’s unique.
Implement rules for duplicates (log, resolve manually, or treat as error).

### 7.2 ProductType usage (MARKETING_PKG_AUTO, etc.)
productType is just a string in Shopify.
Systems often use it to:
Mark special products (marketing packages, bundles, subscriptions).
In OMS:
Interpret specific values (e.g. MARKETING_PKG_AUTO) with custom rules:
Don’t allocate inventory.
Treat as service or marketing charge.
Or expand into underlying child products.

### 7.3 Large catalogs and performance
Use Bulk Operations for initial load and large resyncs.
Limit fields in bulk queries to only what the OMS needs.
Consider:
Breaking by date ranges or other filters if hitting limits.
Caching & only applying deltas once baseline is loaded.

### 7.4 Error handling and idempotency
Product sync operations should be idempotent:
Applying the same data twice shouldn’t break anything.
Upsert by product/variant ID or SKU.
Log and retry:
Failed writes to OMS.
Network errors when calling Shopify.

## 8. High-level product-sync architecture example
A typical robust setup:

Initial product load

Run a Bulk Operation to export all products (with variants and inventory).
Download and process JSONL into OMS product table(s).
Ongoing product changes

Subscribe to:
products/create, products/update, products/delete (and optionally variant-specific webhooks).
On each webhook:
Fetch updated product via REST/GraphQL.
Upsert into OMS.
Handle inventory updates via:
inventory_levels/update webhooks, or
OMS as master updating Shopify.
Periodic consistency checks

Optional: nightly/weekly bulk export + compare count/spot-checks to ensure OMS and Shopify are consistent.
Conflict rules

Decide what happens if:
Product edited in OMS vs Shopify at the same time.
SKU collisions occur.
If you tell me:

Whether your OMS is read-only from Shopify or also writes back (prices, inventory, etc.), and
Roughly what your OMS product/variant schema looks like,
I can go a step further and:

Propose a concrete product sync flow for your case,
And (if you want) sketch actual GraphQL bulk queries / REST calls and a mapping table Shopify field → OMS field.
Sources
API: Admin GraphQL > Latest > Objects > Product
Product (GraphQL)
API: Admin REST > Latest > Resources > Product
Product (REST)
API: Admin GraphQL > Latest > Objects > ProductVariant
ProductVariant (GraphQL)
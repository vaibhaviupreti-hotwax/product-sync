# Entity: ProductType

## Purpose
The `ProductType` entity serves as the **architectural classifier** for all records in the `Product` table. It defines the fundamental nature of an item, driving critical ERP logic such as whether the system should track inventory, require a shipping label, or allow a digital download. It acts as a "behavioral switch" for the fulfillment engine.

## Where Used In Code
- **Services**: `createProduct`, `updateProduct` (validates type existence), `getInventoryAvailableByFacility` (logic often filters by physical types).
- **Handlers**: Shipping handlers (checks `isPhysical` to trigger label generation).
- **APIs**: Shopify/NetSuite Connectors (mappings external "Product Categories" to internal types).
- **Sync Processors**: `ProductImportServices.java` (resolves incoming categories into standard system types).

## Entity Structure

| Field | Type | Required | Example | Description |
| :--- | :--- | :---: | :--- | :--- |
| **`productTypeId`** | `id` | **Yes** | `FINISHED_GOOD` | Unique identifier (Primary Key). |
| **`parentTypeId`** | `id` | No | `GOOD` | Self-reference for hierarchical classification. |
| **`isPhysical`** | `indicator` | No | `Y` | Determines if the item occupies warehouse space. |
| **`isDigital`** | `indicator` | No | `N` | Determines if the item is a virtual entitlement. |
| **`hasTable`** | `indicator` | No | `N` | Legacy OFBiz field indicating a dedicated extension table. |
| **`description`** | `description` | No | `Finished Good` | Human-readable label for the UI and reporting. |

## Seed / Initial Data
The entity is pre-populated during the system bootstrap using `ProductSeedData.xml`. It rarely changes after the initial implementation.

**Default Records:**
| productTypeId | parentTypeId | isPhysical | isDigital | description |
| :--- | :--- | :---: | :---: | :--- |
| `GOOD` | `NULL` | `Y` | `N` | All Physical Goods |
| `FINISHED_GOOD` | `GOOD` | `Y` | `N` | Sellable Inventory Item |
| `SERVICE` | `NULL` | `N` | `N` | Labor or Consulting |
| `DIGITAL_GOOD` | `GOOD` | `N` | `Y` | Software or E-book |

## Relationships
- **Self-Reference (Parent)**: `parentTypeId` points to another `ProductType` record (Hierarchy flow: `FINISHED_GOOD` $\rightarrow$ `GOOD`).
- **Child Entity**: `Product` (FK relationship on `productTypeId`).
- **Dependent Attribute**: `ProductTypeAttr` (Stores custom metadata configurations specific to a type).

## Data Population Flow
1. **Bootstrap**: Seed data is loaded via `gradlew load-seed`.
2. **Lookup**: When a Shopify product syncs, the service performs a `delegator.findOne("ProductType", ...)` to ensure the assigned type is valid.
3. **Persistence**: The `Product` entity is persisted only after the `productTypeId` is validated against this master table.

## Source Payload Mapping
**Example Shopify JSON:**
```json
{
  "product_type": "Apparel",
  "published_scope": "global"
}
```
**Field Mapping:**
- `product_type` (Apparel) $\rightarrow$ If "Apparel" is not a defined `productTypeId`, the HotWax sync logic maps it to the system default **`FINISHED_GOOD`**.

## Dummy Data Example
| productTypeId | parentTypeId | isPhysical | description |
| :--- | :--- | :---: | :--- |
| `MARKETING_KIT` | `GOOD` | `Y` | Promotional bundle of flyers |

## Normalization Explanation
- **Separation of Concerns**: By splitting `isPhysical` and `isDigital` into a Type table, we avoid duplicating this metadata on millions of individual Product rows.
- **Data Integrity**: Enforces a "Restricted Vocabulary." Users cannot invent new product behaviors without first defining them in this master table.
- **Transactional Efficiency**: Allows the fulfillment engine to exclude non-shippable items (Services/Digital) from warehouse picking lists using a simple join.

## Full Sync Flow
1. **External Payload**: Received from Shopify Webhook.
2. **Transformation Layer**: The `ProductTransform` service checks the `product_type` string.
3. **Master Check**: It looks up the `ProductType` table. If the string is empty or unrecognized, it defaults to `FINISHED_GOOD`.
4. **Service Layer**: Calls `createProduct` with the resolved `productTypeId`.
5. **DB Persistence**: Writes to the `PRODUCT` table with a valid FK reference.

## Important Business Rules
- **Referential Integrity**: A product **cannot** exist without a valid `productTypeId`.
- **Inheritance**: If a field is null on a sub-type, logic sometimes traverses the `parentTypeId` to find a default (though rare in basic product setup).
- **Mandatory Logic**: `isPhysical='Y'` is the trigger for all **Inventory** and **Shipment** entity creation downstream.

## Important Observations
- **Caching**: This entity is heavily cached (`cache="true"` in the entity engine). Changes require a cache clear or system restart to take effect globally.
- **Type-Safe Extensions**: HotWax rarely uses the `hasTable='Y'` feature; instead, custom data is typically handled via `ProductAttribute` or `ProductTypeAttr`.

---
**Documentation for `ProductType` is complete. Please provide the next Entity Name.**

---
Viewed EXTRA.MD:78-84
Edited EXTRA.MD

# Entity:ProductFeatureType

## Purpose
The `ProductFeatureType` entity acts as the **metadata category** for variations and characteristics. While a "Feature" is a specific value (e.g., Red), the "Feature Type" is the category (e.g., Color). It defines the "What" of a product characteristic, allowing the system to logically group variant options and drive faceted search filters on the frontend.

## Where Used In Code
- **Services**: `createProductFeature` (ensures valid type assignment), `getFeatureTypesByProduct` (used to generate product detail page dropdowns).
- **Handlers**: Shopify Connector (maps Shopify "Option Name" such as "Size" or "Color" to an internal `productFeatureTypeId`).
- **APIs**: Search APIs (uses these types as "facet" headers for website filtering).
- **Sync Processors**: `ProductFeatureServices.java` (logic to handle incoming product options from external ERPs/platforms).

## Entity Structure

| Field | Type | Required | Example | Description |
| :--- | :--- | :---: | :--- | :--- |
| **`productFeatureTypeId`** | `id` | **Yes** | `COLOR` | Unique identifier (Primary Key). |
| **`parentTypeId`** | `id` | No | `DIMENSION` | Supports hierarchical grouping (e.g., `HEIGHT` under `DIMENSION`). |
| **`description`** | `description` | No | `Color` | The label displayed as the filter header or dropdown title. |
| **`hasTable`** | `indicator` | No | `N` | Indicates if this type uses a physical extension table (Standard: `N`). |

## Seed / Initial Data
Standard commerce types are seeded during initialization to ensure the Shopify connector has a baseline for mapping.

**Default Records:**
| productFeatureTypeId | parentTypeId | description |
| :--- | :--- | :--- |
| `COLOR` | `NULL` | Color |
| `SIZE` | `NULL` | Size |
| `MATERIAL` | `NULL` | Material |
| `BRAND` | `NULL` | Brand |
| `DIMENSION` | `NULL` | Dimension |

## Relationships
- **Self-Reference**: `parentTypeId` points back to `ProductFeatureType` to build attribute trees.
- **Child Entity**: `ProductFeature` (One-to-Many; `COLOR` has many `ProductFeature` records like `RED`, `BLUE`).
- **Junction**: `ProductFeatureGroupAppl` (Links features to groups for specialized catalog browsing).

## Data Population Flow
1. **Initial Seed**: Loaded via `ProductSeedData.xml`.
2. **External Sync**: When a product is imported from Shopify, the connector looks at the "Options" array.
3. **Lookup**: The service searches for a `productFeatureTypeId` or `description` that matches the Shopify option name (e.g., "Shade").
4. **Resolution**: If found, it uses that ID; if not, it may dynamically create a new type (depending on the sync configuration).

## Source Payload Mapping
**Example Shopify GraphQL (Options):**
```json
{
  "options": [
    {
      "name": "Color",
      "values": ["Navy", "Crimson"]
    }
  ]
}
```
**Field Mapping:**
- `options[0].name` (Color) $\rightarrow$ Maps to **`productFeatureTypeId="COLOR"`** or searches **`description`**.

## Dummy Data Example
| productFeatureTypeId | parentTypeId | description |
| :--- | :--- | :--- |
| **`FABRIC_WEIGHT`** | `NULL` | Grams per Square Meter (GSM) |

## Normalization Explanation
- **Attribute Abstraction**: Instead of having columns like `color` or `size` in the `Product` table, we use this table to define categories dynamically.
- **Duplication Avoidance**: By categorizing features, the system ensures that "Cotton" is always treated as a `MATERIAL` and never accidentally sorted as a `SIZE`.
- **Search Optimization**: Faceted search requires consistent categorization. Grouping values under `productFeatureTypeId` allows the database to return all "Colors" for a specific category quickly.

## Full Sync Flow
1. **Payload Received**: Shopify variant data reaches the `ShopifyProductTransform` service.
2. **Option Parsing**: The service iterates through `Option1`, `Option2`, etc.
3. **Identity Check**: Queries the `ProductFeatureType` table to validate the category.
4. **Service Call**: Invokes `createProductFeature` using the resolved type ID.
5. **Persistence**: The category is used as a foreign key in the `ProductFeature` table.

## Important Business Rules
- **Referential Integrity**: A `ProductFeature` cannot be created without a valid `productFeatureTypeId`.
- **Description Mapping**: HotWax often uses the `description` field for "Fuzzy Matching" if the incoming external ID doesn't exactly match internal codes.
- **Hierarchy Logic**: When querying for `DIMENSION`, the system can be configured to recursively pull all child types (Width, Height, Depth).

## Important Observations
- **Facet Configuration**: In the HotWax Commerce UI, these types are used to enable/disable specific filters in the storefront.
- **Readability**: IDs are typically human-readable (e.g., `SIZE`) to simplify manual SQL debugging and XML data maintenance.

---
**Documentation for `ProductFeatureType` is complete. Please provide the next Entity Name.**
---
Edited EXTRA.MD
Viewed EXTRA.MD:79-91

# Entity: ProductCategoryType

## Purpose
The `ProductCategoryType` entity defines the **functional role** of a category. In an enterprise system, categories are used for more than just website navigation; they are used for tax grouping, promotional logic, internal search categorization, and replenishment rules. This entity ensures the ERP logic can distinguish between a category meant for "Winter Jackets" (Browsing) and one meant for "Digital Goods Tax" (Regulatory).

## Where Used In Code
- **Services**: `createProductCategory` (validates type existence), `getTaxCategory` (explicitly filters by types like `TAX_CATEGORY`).
- **Handlers**: Pricing and Promotion engines (looks for memberships in categories typed as `PROMOTIONAL_CATEGORY`).
- **APIs**: Catalog and Navigation APIs (excludes internal or tax categories from being exposed to the storefront).
- **Sync Processors**: `ProductCategoryServices.java` (decides which type to assign to incoming collections from Shopify or other platforms).

## Entity Structure

| Field | Type | Required | Example | Description |
| :--- | :--- | :---: | :--- | :--- |
| **`productCategoryTypeId`** | `id` | **Yes** | `CATALOG_CATEGORY` | Unique identifier (Primary Key). |
| **`parentTypeId`** | `id` | No | `INTERNAL_CATEGORY` | Supports hierarchical grouping of category types. |
| **`hasTable`** | `indicator` | No | `N` | Indicates if this type has an extension table (Standard: `N`). |
| **`description`** | `description` | No | `Catalog Category` | The human-readable name of the category's functional role. |

## Seed / Initial Data
The entity is populated with standard commerce roles during the initial data load to provide a framework for all downstream organizational logic.

**Default Records:**
| productCategoryTypeId | parentTypeId | description |
| :--- | :--- | :--- |
| `CATALOG_CATEGORY` | `NULL` | Browsing/Sales Category |
| `TAX_CATEGORY` | `NULL` | Regulatory Tax Group |
| `PROMOTIONAL_CATEGORY` | `NULL` | Used for Discounts/Sales |
| `QUICK_ADD_CATEGORY` | `NULL` | Internal UI helper group |
| `SEARCH_CATEGORY` | `NULL` | Used for filtering logic only |

## Relationships
- **Self-Reference**: `parentTypeId` allows for creating a hierarchy of roles (e.g., `HOLIDAY_PROMO` under `PROMOTIONAL_CATEGORY`).
- **Child Entity**: `ProductCategory` (One-to-Many; every category record must belong to a valid type).
- **Dependent Configuration**: `ProductCategoryTypeAttr` (Stores metadata specific to the type).

## Data Population Flow
1. **Seed Load**: Standard types are defined in `ProductSeedData.xml`.
2. **Category Creation**: When a new category is synced or manually added, the `productCategoryTypeId` must be specified.
3. **Dispatch Logic**: The system uses the type to decide where to route the category (e.g., if type is `TAX_CATEGORY`, it is sent to the Tax calculation service).

## Source Payload Mapping
**Example Shopify Webhook (Collections):**
```json
{
  "title": "Summer Collection",
  "rules": [{"column": "tag", "relation": "equals", "condition": "summer"}]
}
```
**Field Mapping:**
- Since Shopify "Collections" are primarily for sales, the HotWax sync logic maps this payload to **`productCategoryTypeId="CATALOG_CATEGORY"`**.

## Dummy Data Example
| productCategoryTypeId | parentTypeId | description |
| :--- | :--- | :--- |
| **`INTERNAL_PICKING`** | `NULL` | Group used only for warehouse picking priorities |

## Normalization Explanation
- **Behavioral Isolation**: By separating the "Role" (Type) from the "Category" (Name), we avoid adding hardcoded flags like `isTaxGroup` to the main category table.
- **Schema Scalability**: New category behaviors (e.g., "Subscription Tier") can be added by simply inserting a new row here without a database schema migration.
- **Query Precision**: Allows developers to write queries like "Select all categories where type is `CATALOG_CATEGORY`" to instantly filter out tax or internal metadata categories.

## Full Sync Flow
1. **Payload Arrival**: Sync service receives a collection or group from an external source.
2. **Purpose Resolution**: Logic determines the role based on source (e.g., Shopify collections $\rightarrow$ `CATALOG_CATEGORY`).
3. **Validation**: Performs a cache lookup on `ProductCategoryType`.
4. **Service Execution**: Invokes `createProductCategory` with the assigned type.
5. **DB Commit**: Writes the category to the database with the functional role link.

## Important Business Rules
- **Referential Integrity**: A `ProductCategory` cannot be created unless its type exists in this table.
- **UI Logic**: HotWax admin apps often use this type to decide which "tab" or "screen" a category should appear in.
- **Mandatory Logic**: The type is used as a filter in most catalog services; an incorrect type will cause a category to be invisible on the website.

## Important Observations
- **Hardcoded Dependencies**: Some core Java services (like Tax and Promo engines) have `productCategoryTypeId` values hardcoded as constants (e.g., `TAX_CATEGORY`).
- **Caching**: This is a static table; the Entity Engine caches it by default for high-performance lookups.

---
**Documentation for `ProductCategoryType` is complete. Please provide the next Entity Name.**
---
Viewed EXTRA.MD:251-258
Edited EXTRA.MD

# Entity: GoodIdentificationType

## Purpose
The `GoodIdentificationType` entity defines the **registry schema** for product identifiers. In an ERP, a product is rarely identified by a single ID; it has a SKU for the warehouse, a Barcode for the POS, an ISBN for books, and various external platform IDs (Shopify, Amazon). This entity categorizes these identifiers, allowing the system to perform high-speed lookups across multiple numbering schemes without polluting the main `Product` table with dozens of specific columns.

## Where Used In Code
- **Services**: `createGoodIdentification` (validates the type category), `findProductByGoodIdentification` (primary service used to resolve a scanned SKU/Barcode into an internal `productId`).
- **Handlers**: Shopify/NetSuite Connectors (uses specific types like `SHOPIFY_PROD_ID` or `SKU` to match external records during synchronization).
- **APIs**: POS Scan APIs (explicitly filters for `goodIdentificationTypeId="UPCA"` or `EAN` to handle scanner inputs).
- **Schedulers**: Inventory Feed jobs (joins with this table to export the correct SKU format for third-party logistics providers).

## Entity Structure

| Field | Type | Required | Example | Description |
| :--- | :--- | :---: | :--- | :--- |
| **`goodIdentificationTypeId`** | `id` | **Yes** | `SKU` | Unique identifier for the ID category (Primary Key). |
| **`parentTypeId`** | `id` | No | `BARCODE` | Supports grouping (e.g., `UPCA` and `EAN13` can fall under `BARCODE`). |
| **`hasTable`** | `indicator` | No | `N` | Indicates if an extension table exists (Standard: `N`). |
| **`description`** | `description` | No | `Stock Keeping Unit` | Human-readable name of the identifier type. |

## Seed / Initial Data
Standard industry identifiers are seeded during the initial build to ensure cross-platform compatibility out of the box.

**Default Records:**
| goodIdentificationTypeId | parentTypeId | description |
| :--- | :--- | :--- |
| `SKU` | `NULL` | Stock Keeping Unit |
| `UPCA` | `NULL` | Universal Product Code (US) |
| `EAN` | `NULL` | European Article Number |
| `ISBN` | `NULL` | International Standard Book Number |
| `SHOPIFY_PROD_ID` | `NULL` | Shopify Internal ID |

## Relationships
- **Self-Reference**: `parentTypeId` allows for hierarchical identifier management (e.g., grouping various barcode standards).
- **Child Entity**: `GoodIdentification` (One-to-Many; this table defines the *kinds* of IDs, while the child table stores the *actual values* like `HOODIE-RED-XL`).
- **Dependent Configuration**: `GoodIdentificationTypeAttr` (Used for storing validation regex or formatting rules for a specific ID type).

## Data Population Flow
1. **Seed Load**: Master types are defined in `ProductSeedData.xml`.
2. **Connector Config**: The Shopify Sync configuration specifies which HotWax `goodIdentificationTypeId` should receive the Shopify "Variant SKU."
3. **Validation**: During any "Identify Product" operation, the system checks this table to confirm the lookup key (e.g., `UPCA`) is a valid registered type.
4. **Resolution**: The system uses the type to narrow down search indexes (e.g., "Find the product where ID is '123' and type is 'SKU'").

## Source Payload Mapping
**Example Shopify Webhook (Variant):**
```json
{
  "sku": "TSHIRT-BLU-LG",
  "barcode": "88776655"
}
```
**Field Mapping:**
- `sku` ("TSHIRT-BLU-LG") $\rightarrow$ Maps to **`goodIdentificationTypeId="SKU"`**.
- `barcode` ("88776655") $\rightarrow$ Maps to **`goodIdentificationTypeId="UPCA"`** (configured in the sync layer).

## Dummy Data Example
| goodIdentificationTypeId | parentTypeId | description |
| :--- | :--- | :--- |
| **`ASIN`** | `NULL` | Amazon Standard Identification Number |

## Normalization Explanation
- **Identifier Abstraction**: Instead of adding a new column to the `Product` table every time a new marketplace (like Amazon or eBay) is added, we simply add a new row to this table.
- **Data Integrity**: By categorizing IDs, we prevent "Manufacturer IDs" from being used in the "SKU" field, which would break warehouse picking logic.
- **Transactional Reasoning**: Facilitates a "Universal Identity Service." A single Java method can take any string and a type ID, and instantly return a `productId` without needing complex conditional SQL.

## Full Sync Flow
1. **Payload Arrival**: Sync service receives variant data containing multiple ID strings.
2. **Metadata Lookup**: Logic identifies the destination type (e.g., Shopify ID $\rightarrow$ `SHOPIFY_PROD_ID`).
3. **Integrity Check**: Performs a cache-hit on `GoodIdentificationType` to verify the role.
4. **Service Dispatch**: Invokes `createGoodIdentification` for each unique ID provided in the payload.
5. **Persistence**: Multiple rows are written to the `GOOD_IDENTIFICATION` table, all linked back to the same product.

## Important Business Rules
- **Uniqueness across Products**: Within a specific `goodIdentificationTypeId` (like `SKU`), the `idValue` should be globally unique to prevent order fulfillment errors.
- **Search Priority**: System logic often prioritizes `SKU` for internal warehouse operations and `UPCA` for retail scanner operations.
- **Mandatory Parentage**: FK constraint ensures that if a `parentTypeId` is provided, it must be a valid row in this table.

## Important Observations
- **Performance Indexing**: The combination of `goodIdentificationTypeId` and `idValue` is the most frequently queried index in the entire Product module.
- **Platform Bridging**: This entity is the "Rosetta Stone" that allows HotWax to link a Shopify Product, an Amazon Listing, and a NetSuite Item to the same internal record.

---
**Documentation for `GoodIdentificationType` is complete. Please provide the next Entity Name.**
---
Viewed EXTRA.MD:338-345
Edited EXTRA.MD

# Entity: Product

## Purpose
The `Product` entity is the **central hub** of the entire commerce engine. It represents the "noun" of the system—any item that can be manufactured, stocked, priced, or sold. It utilizes a sophisticated **Virtual-Variant architecture** where a "Virtual" product acts as a style template (e.g., "Crew Neck Tee") and "Variant" products represent the actual physical units (e.g., "Crew Neck Tee - Red - Large").

## Where Used In Code
- **Services**: `createProduct`, `updateProduct`, `getProductInventory` (calculates stock for a specific ID), `calculateProductPrice`.
- **Handlers**: Order entry (links `OrderItem` to `Product`), Picking/Packing (links warehouse tasks to the physical unit).
- **APIs**: Shopify/BigCommerce Sync (creates the parent-child structure), Solr/Elasticsearch Indexers (scans this table to build the search catalog).
- **Schedulers**: `InventorySyncJob`, `PriceUpdateJob` (iterates through product IDs to refresh external platforms).

## Entity Structure

| Field | Type | Required | Example | Description |
| :--- | :--- | :---: | :--- | :--- |
| **`productId`** | `id` | **Yes** | `10001` | Unique internal primary key. |
| **`productTypeId`** | `id` | **Yes** | `FINISHED_GOOD` | Links to `ProductType` (e.g., Good, Service). |
| **`isVirtual`** | `indicator` | No | `Y` | 'Y' if this is a parent style/template. |
| **`isVariant`** | `indicator` | No | `N` | 'Y' if this is a specific sellable SKU. |
| **`productName`** | `name` | No | `Organic Cotton Hoodie` | The customer-facing marketing name. |
| **`internalName`** | `name` | No | `HOO-ORG-RED-XL` | Internal name used for reporting and warehouse. |
| **`description`** | `description` | No | `Cozy organic hoodie` | Short text for search results. |
| **`longDescription`** | `very-long` | No | `...` | Full marketing copy for the product page. |
| **`manufacturerPartyId`** | `id` | No | `ACME_CORP` | Links to the `Party` who produced the item. |

## Seed / Initial Data
Standard installs are usually empty. However, system-level products like "Shipping Charges" or "Misc Item" may be seeded for accounting fallbacks.

## Relationships
- **Parent Entities**: `ProductType` (FK), `Party` (Manufacturer).
- **Child Entities**: `ProductAssoc` (The link between Virtual and Variant), `GoodIdentification` (SKUs), `ProductPrice`, `ProductFeatureAppl`, `InventoryItem`.
- **Relationship Flow**: The `Product` table is the **Absolute Root**. Every other table in the Product module requires a valid `productId` to exist before they can hold data.

## Data Population Flow
1. **Payload Received**: Shopify Webhook sends a product with multiple variants.
2. **DTO Mapping**: System creates a `ShopifyProductDTO`.
3. **Identity Resolution**: Service checks `GoodIdentification` to see if the product already exists.
4. **Base Creation**:
   - `createProduct` is called for the **Virtual** parent.
   - `createProduct` is called for each **Variant** child.
5. **Association**: `createProductAssoc` is called to "glue" the children to the parent.
6. **Detailing**: Child services are called to populate SKUs, Features, and Prices.

## Source Payload Mapping
**Example Shopify JSON:**
```json
{
  "id": 1234567,
  "title": "Classic Denim Jacket",
  "vendor": "DenimCo",
  "body_html": "<strong>Premium quality...</strong>"
}
```
**Field Mapping:**
- `id` (1234567) $\rightarrow$ `GoodIdentification.idValue` (linked to `Product.productId`).
- `title` $\rightarrow$ `Product.productName`.
- `vendor` $\rightarrow$ `Product.manufacturerPartyId` (via lookup).
- `body_html` $\rightarrow$ `Product.longDescription`.

## Dummy Data Example
| productId | productTypeId | isVirtual | productName |
| :--- | :--- | :---: | :--- |
| **`SHIRT_001`** | `FINISHED_GOOD` | `Y` | Vintage Wash Tee |
| **`SHIRT_001_RED_S`** | `FINISHED_GOOD` | `N` | Vintage Wash Tee - Red - Small |

## Normalization Explanation
- **Attribute Offloading**: Marketing text stays in `Product`, but dynamic values (Stock in `InventoryItem`, Price in `ProductPrice`) are moved to child tables. This prevents the `Product` table from growing horizontally.
- **Virtual/Variant Split**: All "Shared Data" (Name, Image, Description) is stored once on the Virtual record. Only "Unique Data" (Color, Size, Weight) is stored on the Variant.
- **Transactional Reasoning**: High-frequency updates like **Stock Changes** do not lock the `Product` table, ensuring the website can still display names/descriptions while inventory is being updated.

## Full Sync Flow
1. **Validation**: Ensures the `productTypeId` is valid (e.g., `FINISHED_GOOD`).
2. **Transformation**: Maps Shopify's flat JSON to HotWax's relational map.
3. **Service Layer**: `ProductImportServices.java` orchestrates the sequence.
4. **Order of Persistence**:
   - 1. **Virtual Product** (Parent)
   - 2. **Variant Products** (Children)
   - 3. **Associations** (Parent-Child Link)
   - 4. **Identifications** (SKUs/Shopify IDs)
   - 5. **Features** (Color/Size links)

## Important Business Rules
- **Immutability**: The `productId` is the system's "Source of Truth" and should never change after creation.
- **Mandatory Parent**: A Variant product should ideally always have a Virtual parent in the fashion/apparel domain.
- **Primary Category**: While a product can be in many categories, the `primaryProductCategoryId` field is often used as a fallback for breadcrumb generation.

## Important Observations
- **Internal Name Strategy**: HotWax often auto-generates the `internalName` as `SKU + Name` to allow warehouse staff to search easily without joining tables.
- **Performance**: Indexing `isVirtual` and `isVariant` is critical for filtering the product catalog versus the inventory warehouse view.

---
**Documentation for `Product` is complete. Please provide the next Entity Name.**
---
Viewed EXTRA.MD:434-441
Edited EXTRA.MD

# Entity: ProductAssoc

## Purpose
The `ProductAssoc` entity is the **connective tissue** of the product catalog. It defines directed relationships between two products, most critically linking **Virtual** parent templates to their **Variant** sellable children. Beyond variations, it manages the logic for product bundles (Components), marketing cross-sells (Related products), and versioning (Upgrades/Replacements). It is a temporal entity, meaning it tracks the history of these relationships over time.

## Where Used In Code
- **Services**: `createProductAssoc` (standard link service), `getVariantIdsByVirtualId` (the most frequent lookup in the catalog engine), `getAssocProducts` (generic relationship resolver).
- **Handlers**: Order Management (explodes a bundle into its constituent parts if the type is `PRODUCT_COMPONENT`).
- **APIs**: Shopify/ERP Connectors (maintains the parent-child hierarchy during synchronization).
- **Sync Processors**: `ProductAssociationServices.java` (logic for expiring old associations when a product is moved between style groups).

## Entity Structure

| Field | Type | Required | Example | Description |
| :--- | :--- | :---: | :--- | :--- |
| **`productId`** | `id` | **Yes** | `HOODIE_STYLE_01` | The "Parent" or "Source" product ID. |
| **`productIdTo`** | `id` | **Yes** | `HOODIE_RED_XL` | The "Child" or "Destination" product ID. |
| **`productAssocTypeId`** | `id` | **Yes** | `PRODUCT_VARIANT` | The nature of the link (e.g., Variant, Component). |
| **`fromDate`** | `date-time` | **Yes** | `2024-05-12 10:00:00` | When the relationship became active (Part of PK). |
| **`thruDate`** | `date-time` | No | `2024-12-31 23:59:59` | When the relationship expired. |
| **`sequenceNum`** | `number` | No | `5` | Sorting order (e.g., order of items in a bundle). |
| **`quantity`** | `fixed-point` | No | `2.0` | Used for bundles (e.g., a "2-Pack" of socks). |

## Seed / Initial Data
The types of associations are seeded in `ProductSeedData.xml`. These define the "rules" for how products can interact.

**Default Records:**
| productAssocTypeId | description | Business Use Case |
| :--- | :--- | :--- |
| `PRODUCT_VARIANT` | Variant Link | Connecting Style Parent to SKU Child. |
| `PRODUCT_COMPONENT` | Bundle/Kit | Defining items inside a gift basket or set. |
| `ALSO_BOUGHT` | Cross-sell | Marketing suggestions on the product page. |
| `PRODUCT_UPGRADE` | Upsell | Suggesting a newer model to the customer. |

## Relationships
- **Parent Entities**: `Product` (as `productId`), `Product` (as `productIdTo`), `ProductAssocType` (FK).
- **Child Entity**: `ProductAssocAttr` (Used for storing metadata about the link itself).
- **Relationship Flow**: Creates a **Directed Graph**. Data flows from the "Source" (`productId`) to the "Target" (`productIdTo`).

## Data Population Flow
1. **Sync Arrival**: Shopify Sync reports a new variant belonging to a specific product handle.
2. **Existence Check**: The service confirms both the style record and the variant record already exist in the `PRODUCT` table.
3. **Duplicate Filter**: The code checks if an active `PRODUCT_VARIANT` link already exists for this pair to avoid Primary Key collisions.
4. **Service Execution**: `createProductAssoc` is called with the current timestamp as the `fromDate`.
5. **Persistence**: The link is saved, instantly making the variant "visible" under the parent on the storefront.

## Source Payload Mapping
**Example Shopify JSON (Variant context):**
```json
{
  "product_id": 789012,
  "id": 345678,
  "sku": "TEE-BLU-MD"
}
```
**Field Mapping:**
- Shopify `product_id` (789012) $\rightarrow$ `ProductAssoc.productId` (after internal ID resolution).
- Shopify `id` (345678) $\rightarrow$ `ProductAssoc.productIdTo` (after internal ID resolution).
- Type Constant $\rightarrow$ `ProductAssoc.productAssocTypeId = "PRODUCT_VARIANT"`.

## Dummy Data Example
| productId | productIdTo | productAssocTypeId | quantity | Use Case |
| :--- | :--- | :--- | :---: | :--- |
| **`PACK_OF_3_TEES`** | **`TEE_BASIC_WHITE`** | `PRODUCT_COMPONENT` | `3.0` | Multi-pack bundle |

## Normalization Explanation
- **Many-to-Many Support**: A single product (e.g., "White Tee") can be a `PRODUCT_VARIANT` of a style and simultaneously a `PRODUCT_COMPONENT` of a "Summer Bundle." This table allows that without redundant columns in the `Product` table.
- **Temporal Decoupling**: We don't delete relationships; we "expire" them using `thruDate`. This preserves the integrity of historical orders that were placed when the relationship was active.
- **Relational Integrity**: By using IDs for both sides, the system ensures that you cannot link to a product that doesn't exist in the master registry.

## Full Sync Flow
1. **Payload Parse**: Sync job identifies the parent-child relationship from the external platform.
2. **Integrity Validation**: Verifies that both `productId` and `productIdTo` are present in the system.
3. **Conflict Resolution**: If the variant is being moved to a new parent, the service automatically sets the `thruDate` of the *old* association to `now`.
4. **Creation**: Inserts the new association row.
5. **Cache Invalidation**: Clears the `productAssoc` cache so the website immediately reflects the new hierarchy.

## Important Business Rules
- **PK Constraint**: The combination of `productId`, `productIdTo`, `productAssocTypeId`, and `fromDate` must be unique.
- **Quantity Logic**: For `PRODUCT_VARIANT`, quantity is always `1.0`. For `PRODUCT_COMPONENT`, it defines the ratio of the kit.
- **No Recursion**: Business logic prevents a product from being its own parent.

## Important Observations
- **Explosion Logic**: When an order for a bundle is received, the system "explodes" the `PRODUCT_COMPONENT` links to create hidden child order items for warehouse picking.
- **Temporal Querying**: Services always filter by `(fromDate <= now AND (thruDate IS NULL OR thruDate > now))` to ensure only currently active relationships are used.

---
**Documentation for `ProductAssoc` is complete. Please provide the next Entity Name.**
---
# Entity: GoodIdentification

## Purpose
The `GoodIdentification` entity is the **"Universal Alias Registry"** for products. While the system identifies a product internally via a `productId` (often a sequential number), the outside world uses various strings like SKUs, Barcodes, UPCs, and external platform IDs (Shopify Variant IDs, Amazon ASINs). This entity maps these many-to-one human-friendly identifiers to the internal system ID, enabling the ERP to resolve any external scan or API payload into the correct internal product record.

## Where Used In Code
- **Services**: `createGoodIdentification` (populates aliases), `findProductByGoodIdentification` (the core lookup service used by POS and Order Import), `getProductSku` (utility to retrieve the primary SKU string).
- **Handlers**: POS Barcode Scanner (resolves scanner input to `productId`), Shopify Order Webhook (resolves Shopify SKU/ID to internal product).
- **APIs**: Product Search APIs (allows searching by SKU or Barcode), Marketplace Connectors (stores/retrieves cross-platform mapping IDs).
- **Schedulers**: Inventory Export Jobs (joins with this table to include the `SKU` in the CSV/JSON feed for external partners).

## Entity Structure

| Field | Type | Required | Example | Description |
| :--- | :--- | :---: | :--- | :--- |
| **`productId`** | `id` | **Yes** | `10001` | Internal ID (Part of Primary Key). |
| **`goodIdentificationTypeId`** | `id` | **Yes** | `SKU` | Category of ID (Part of Primary Key). |
| **`idValue`** | `id-long` | **Yes** | `HOODIE-RED-XL` | The actual identifier string (e.g., Barcode). |

## Seed / Initial Data
The table itself is empty upon installation and is populated dynamically as products are created. However, it relies on the `GoodIdentificationType` master data (Seeded) to define valid categories.

## Relationships
- **Parent Entities**: `Product` (FK), `GoodIdentificationType` (FK).
- **Relationship Flow**: **Product (1) $\rightarrow$ GoodIdentification (Many)**. A single product can have multiple IDs: one row for its SKU, one for its UPC, and one for its Shopify ID.
- **Integrity**: Deleting a product will cascade and remove its aliases from this table.

## Data Population Flow
1. **Sync Arrival**: A Shopify product import starts.
2. **Base Product**: The `Product` record is created first.
3. **Identifier Loop**: For each ID in the Shopify payload:
   - Identify the type (e.g., `sku` $\rightarrow$ `SKU`).
   - Call `createGoodIdentification(productId, typeId, value)`.
4. **Validation**: The system performs a "Unique Check" to ensure no other product is already using this SKU.
5. **Persistence**: The record is committed to the `GOOD_IDENTIFICATION` table.

## Source Payload Mapping
**Example Shopify JSON:**
```json
{
  "sku": "VINTAGE-TEE-BLU-S",
  "barcode": "8877665544"
}
```
**Field Mapping:**
- `sku` ("VINTAGE-TEE-BLU-S") $\rightarrow$ `GoodIdentification.idValue` (where `typeId="SKU"`).
- `barcode` ("8877665544") $\rightarrow$ `GoodIdentification.idValue` (where `typeId="UPCA"`).

## Dummy Data Example
| productId | goodIdentificationTypeId | idValue | Use Case |
| :--- | :--- | :--- | :--- |
| **`5001`** | **`SKU`** | **`JOGGER-BLK-32`** | Internal Warehouse ID |
| **`5001`** | **`SHOPIFY_PROD_ID`** | **`gid://shopify/Variant/123`** | Link to Shopify Store |

## Normalization Explanation
- **Identifier Agnosticism**: By moving IDs to a child table, the system can support any number of identifiers (EAN, ASIN, ISBN) without ever having to modify the `Product` table schema.
- **Search Optimization**: This table is optimized for "Exact Match" queries. Searching a single indexed string column is faster than searching across multiple columns in the heavy `Product` table.
- **Data Governance**: External IDs are "Transactional metadata"—they can change if you move platforms. Keeping them here prevents "churn" on the core `Product` entity.

## Full Sync Flow
1. **Validation**: The `findProductByGoodIdentification` service is called to see if the SKU already exists.
2. **Condition**: If the SKU exists but for a different product, a "Duplicate SKU Error" is raised.
3. **Transformation**: Map external fields to the correct `goodIdentificationTypeId`.
4. **Service Execution**: Calls `createGoodIdentification` (or `updateGoodIdentification` if only the value changed).
5. **Persistence**: Rows are committed to the DB.

## Important Business Rules
- **Global Uniqueness**: In most commerce configurations, the pair of `(goodIdentificationTypeId, idValue)` must be unique system-wide (e.g., only one product can have the SKU `TEE-RED`).
- **Referential Integrity**: An ID cannot be orphaned; it must always point to an existing `productId`.
- **Search Fallback**: When an order comes in, the system usually searches by `SHOPIFY_PROD_ID` first (most accurate), then falls back to `SKU`.

## Important Observations
- **The "Rosetta Stone"**: This table is the only place where HotWax remembers how its internal ID relates to external systems. Without this data, sync between Shopify and the OMS is impossible.
- **Indexing**: An index on `idValue` is mandatory for system performance, as every single order and inventory update triggers a lookup on this column.

---
**Documentation for `GoodIdentification` is complete. Please provide the next Entity Name.**
---
Viewed EXTRA.MD:526-535
Edited EXTRA.MD

# Entity: ProductAttribute

## Purpose
The `ProductAttribute` entity is the **Extensible Metadata Store** for products. It implements the EAV (Entity-Attribute-Value) pattern to store unstructured or business-specific data that does not warrant a dedicated column in the main `Product` table. It is primarily used for **Operational Flags** (e.g., `isPreorder`), **Vendor Metadata**, and **Shopify Tags/Metafields** that drive custom logic within the OMS and eCommerce storefront.

## Where Used In Code
- **Services**: `createProductAttribute`, `updateProductAttribute`, `getProductAttribute` (retrieves a specific flag for logic execution).
- **Handlers**: Order Validation (checks for an attribute like `ALLOW_BACKORDER='Y'` to decide if an order should be rejected or held).
- **APIs**: Shopify/BigCommerce Connectors (stores "Tags" and "Metafields" that are not part of the standard ERP schema).
- **Sync Processors**: `ProductImportServices.java` (iterates through custom fields in a JSON payload and flattens them into this table).

## Entity Structure

| Field | Type | Required | Example | Description |
| :--- | :--- | :---: | :--- | :--- |
| **`productId`** | `id` | **Yes** | `10001` | Internal ID of the product (Part of PK). |
| **`attrName`** | `id-long` | **Yes** | `IS_PREORDER` | The key/name of the attribute (Part of PK). |
| **`attrValue`** | `value` | No | `Y` | The actual string value or indicator stored. |
| **`attrDescription`** | `description` | No | `Flag for pre-order logic` | Optional label explaining the attribute's use. |

## Seed / Initial Data
The table is typically empty upon installation. Unlike `ProductType`, the "keys" (`attrName`) are not usually seeded; they are defined by the business requirements during the integration phase.

## Relationships
- **Parent Entity**: `Product` (Foreign Key link on `productId`).
- **Relationship Flow**: **Product (1) $\rightarrow$ ProductAttribute (Many)**.
- **Logic Bridge**: While not a hard database relationship, many Java services depend on the *existence* of specific `attrName` strings to trigger custom business workflows.

## Data Population Flow
1. **Sync Arrival**: The Shopify sync receives a product with a tag `preorder-june`.
2. **Logic Resolution**: The transformation service parses the tag and decides it represents a `PREORDER` status.
3. **Service Dispatch**: Calls `createProductAttribute(productId, "IS_PREORDER", "Y")`.
4. **Collision Check**: The entity engine ensures that if "IS_PREORDER" already exists for this product, it performs an **update** rather than a create.
5. **Persistence**: The key-value pair is committed to the `PRODUCT_ATTRIBUTE` table.

## Source Payload Mapping
**Example Shopify JSON:**
```json
{
  "tags": "new-arrival, season-summer",
  "metafields": [{"key": "care_instructions", "value": "Dry Clean Only"}]
}
```
**Field Mapping:**
- `tags` ("new-arrival") $\rightarrow$ `ProductAttribute(attrName="SHOP_TAG", attrValue="new-arrival")`.
- `metafields.key` ("care_instructions") $\rightarrow$ `ProductAttribute(attrName="CARE_INSTRUCTIONS", attrValue="Dry Clean Only")`.

## Dummy Data Example
| productId | attrName | attrValue | Use Case |
| :--- | :--- | :--- | :--- |
| **`6001`** | **`IS_PREORDER`** | **`Y`** | Triggers backorder logic |
| **`6001`** | **`WASH_CARE`** | **`COLD_WASH`** | Displayed on product page |

## Normalization Explanation
- **Vertical Growth vs. Horizontal Bloat**: Instead of adding 50 columns to the `Product` table for every possible vendor attribute, we grow the database **vertically**. This keeps the core product table slim and fast.
- **Unstructured Support**: Allows the system to support "Unknown" data from external platforms. If Shopify adds a new field tomorrow, we can store it here without a database migration.
- **Logic Isolation**: Keeps purely **operational/logical** data (like sync flags) separate from **commercial/physical** data (like features and descriptions).

## Full Sync Flow
1. **Payload Extraction**: Connector pulls custom fields or tags from the source system.
2. **Key Normalization**: The service converts keys to a standard format (e.g., `pre-order` $\rightarrow$ `IS_PREORDER`).
3. **Upsert Logic**: Checks if the record exists; if yes, updates `attrValue`.
4. **Service Execution**: Persists via the `delegator`.
5. **Downstream Trigger**: A separate service may listen for these changes to update Solr facets or recalculate order statuses.

## Important Business Rules
- **Unique Constraint**: The combination of `productId` and `attrName` is the Primary Key. You cannot have two different values for the same attribute name on a single product.
- **Indicator Pattern**: Standard HotWax practice uses 'Y' and 'N' for boolean flags stored in the `attrValue` column.
- **Service Dependency**: Deleting a record like `IS_PREORDER` might stop an item from being flagged correctly in the warehouse, even if the `Product` record itself is fine.

## Important Observations
- **No Temporal Tracking**: This table does **not** have `fromDate` or `thruDate`. It only stores the "Current State." If you need to know what the attribute was last month, you must check the audit logs.
- **Search vs. Filter**: While these can be used for filtering, they are not as efficient as `ProductFeature` for high-volume faceted search. They are best used for **logic triggers** in Java services.

---
**Documentation for `ProductAttribute` is complete. Please provide the next Entity Name.**
---
Viewed EXTRA.MD:684-691
Edited EXTRA.MD

# Entity: ProductFeature

## Purpose
The `ProductFeature` entity serves as the **global dictionary** for all possible product characteristics. It stores specific values (e.g., "Red", "Extra Large", "100% Cotton") categorized by their respective types. Unlike `ProductAttribute`, which is used for internal operational flags, `ProductFeature` is designed for **customer-facing variations** and **catalog filtering**. It ensures that a characteristic like "Red" is defined once and shared across thousands of products, enabling consistent search results and a normalized data structure.

## Where Used In Code
- **Services**: `createProductFeature` (populates the dictionary), `getProductFeaturesByType` (retrieves all sizes or colors for a UI dropdown), `getFeatureDescription`.
- **Handlers**: Shopify/BigCommerce Connectors (resolves incoming variant options like "Color: Navy" into these feature records).
- **APIs**: Search Faceting APIs (uses these records to build the "Filter by..." sidebars on the storefront).
- **Sync Processors**: `ProductFeatureServices.java` (orchestrates the creation of features during the initial product import from an external platform).

## Entity Structure

| Field | Type | Required | Example | Description |
| :--- | :--- | :---: | :--- | :--- |
| **`productFeatureId`** | `id` | **Yes** | `COLOR_RED` | Unique primary key (often follows `TYPE_VALUE` pattern). |
| **`productFeatureTypeId`** | `id` | **Yes** | `COLOR` | FK to `ProductFeatureType` (e.g., Size, Color, Material). |
| **`description`** | `description` | No | `Vibrant Red` | The human-readable name shown on the website. |
| **`uomId`** | `id` | No | `LEN_in` | Link to Unit of Measure (used for numeric features like length). |
| **`numberSpecified`** | `fixed-point` | No | `15.5` | The actual numeric value if the feature is a measurement. |
| **`defaultAmount`** | `currency-amount` | No | `5.00` | Optional price surcharge associated with this feature. |

## Seed / Initial Data
While the *Types* are seeded, the *Features* themselves are typically created dynamically during the product sync. However, standard "Master Features" (like basic colors or sizes) may be pre-loaded in `ProductSeedData.xml` to maintain ID consistency across environments.

## Relationships
- **Parent Entities**: `ProductFeatureType` (FK), `Uom` (FK for units).
- **Child Entity**: `ProductFeatureAppl` (The junction table that links this feature to a specific `Product`).
- **Relationship Flow**: **Type $\rightarrow$ Feature $\rightarrow$ Application**. This flow ensures that the "Question" (What is the color?) is linked to the "Answer" (Red), which is then linked to the "Subject" (The Hoodie).

## Data Population Flow
1. **Sync Arrival**: Shopify reports a variant with `Option1: "Navy"`.
2. **Context Identification**: The sync logic identifies that `Option1` maps to the `COLOR` type.
3. **ID Formulation**: The system generates a candidate ID (e.g., `COLOR_NAVY`).
4. **Existence Lookup**: `delegator.findOne("ProductFeature", ...)` checks if this feature already exists.
5. **Creation**: If it does not exist, `createProductFeature` is called to add it to the global dictionary.
6. **Persistence**: The record is committed to the `PRODUCT_FEATURE` table.

## Source Payload Mapping
**Example Shopify JSON (Variant):**
```json
{
  "option1": "Heather Grey",
  "name": "Color"
}
```
**Field Mapping:**
- `option1` ("Heather Grey") $\rightarrow$ `ProductFeature.description`.
- `name` ("Color") $\rightarrow$ `ProductFeature.productFeatureTypeId` (via mapping to `COLOR`).
- System Logic $\rightarrow$ `ProductFeature.productFeatureId = "COLOR_HEATHER_GREY"`.

## Dummy Data Example
| productFeatureId | productFeatureTypeId | description | uomId |
| :--- | :--- | :--- | :--- |
| **`SIZE_MD`** | **`SIZE`** | **Medium** | `NULL` |
| **`WT_10_OZ`** | **`WEIGHT`** | **10 Ounces** | `WT_oz` |

## Normalization Explanation
- **Data Deduplication**: Instead of storing the string "100% Organic Cotton" on 500 different product rows, it is stored once here. The products simply point to this ID.
- **Update Efficiency**: If the marketing team decides to rename "Navy" to "Midnight Blue," you only update **one row** in this table. All associated products on the website reflect the change instantly.
- **Search Faceting**: Search engines (Solr/Elasticsearch) rely on this table to group products. It allows the system to answer the query "Show me everything that is `COLOR_RED`" by searching a single ID column.

## Full Sync Flow
1. **Integrity Validation**: Ensures the `productFeatureTypeId` exists.
2. **Constraint Check**: Verifies that `uomId` is valid if a numeric value is provided.
3. **Lookup/Upsert**: Standard logic to find an existing feature or create a new one to prevent duplicates.
4. **Service Execution**: The `delegator` persists the record.
5. **Mapping Continuation**: The resulting `productFeatureId` is passed to the `ProductFeatureAppl` service to link it to the current product.

## Important Business Rules
- **ID Stability**: The `productFeatureId` should never change once created, as it is a foreign key in many transaction-heavy tables.
- **Mandatory Type**: A feature cannot be "orphaned"; it must always belong to a category.
- **Unit Logic**: If `numberSpecified` is provided, `uomId` should ideally also be provided to ensure the number has physical meaning (e.g., 10 *inches* vs 10 *cm*).

## Important Observations
- **Self-Documenting IDs**: HotWax strongly prefers human-readable IDs (e.g., `MAT_SILK`) over numeric sequences to make manual data audits and XML exports easier to read.
- **Front-end Mapping**: The `description` field is usually the value that ends up in the Shopify "Option" dropdown or the website's color swatch label.

---
**Documentation for `ProductFeature` is complete. Please provide the next Entity Name.**
---
Viewed EXTRA.MD:768-775
Edited EXTRA.MD

# Entity: ProductFeatureApplType

## Purpose
The `ProductFeatureApplType` entity defines the **behavioral role** of a feature's relationship to a product. It serves as the logic-driver for the user interface (UI) and the commerce engine. It determines if a feature is a "Choice" that a customer must make (e.g., selecting a size on a Virtual product) or a "Fact" that describes a specific item (e.g., the specific color of a physical Variant). This entity effectively tells the storefront how to render the product: as an interactive dropdown or as a static attribute list.

## Where Used In Code
- **Services**: `createProductFeatureAppl` (validates the role assigned to the junction), `getProductFeaturesByType` (used to filter features by their application role).
- **Handlers**: Shopify/Marketplace Connectors (logic that determines if an incoming option is a "Variant Option" or an "Informational Tag").
- **APIs**: Product Detail Page (PDP) APIs (uses this type to decide which features belong in the "Options Picker" vs. the "Specifications" table).
- **Sync Processors**: `ProductFeatureServices.java` (orchestrates the assignment of roles during the Virtual-Variant hierarchy creation).

## Entity Structure

| Field | Type | Required | Example | Description |
| :--- | :--- | :---: | :--- | :--- |
| **`productFeatureApplTypeId`** | `id` | **Yes** | `SELECTABLE_FEATURE` | Unique identifier (Primary Key). |
| **`parentTypeId`** | `id` | No | `REQUIRED_FEATURE` | Supports hierarchical grouping of application roles. |
| **`hasTable`** | `indicator` | No | `N` | Indicates if an extension table exists (Standard: `N`). |
| **`description`** | `description` | No | `Selectable Feature` | Human-readable name of the application role. |

## Seed / Initial Data
The core roles that drive standard commerce behavior are seeded during the initial system setup.

**Default Records:**
| productFeatureApplTypeId | description | Business Use Case |
| :--- | :--- | :--- |
| `SELECTABLE_FEATURE` | Choice | Used on Virtual products to create dropdown menus. |
| `STANDARD_FEATURE` | Fact | Used on Variant products to define their specific identity. |
| `REQUIRED_FEATURE` | Mandatory | A feature that must be present for the item to be sellable. |
| `OPTIONAL_FEATURE` | Add-on | Information that is helpful but not part of the variant logic. |

## Relationships
- **Self-Reference**: `parentTypeId` allows for specialized roles (e.g., `PRIMARY_DISTINGUISHING_FEATURE` under `STANDARD_FEATURE`).
- **Child Entity**: `ProductFeatureAppl` (One-to-Many; this table defines the *rule*, while the child table *executes* the link to a product).
- **Relationship Flow**: **Role Rule $\rightarrow$ Feature Application**.

## Data Population Flow
1. **Seed Load**: Standard roles are loaded via `ProductSeedData.xml`.
2. **Context Check (Virtual)**: When the Shopify sync creates a **Virtual** parent style, it assigns features (Color/Size) as `SELECTABLE_FEATURE`.
3. **Context Check (Variant)**: When the sync creates a physical **Variant** SKU, it assigns those same features as `STANDARD_FEATURE`.
4. **Validation**: The system performs a cache-lookup to ensure the requested role is valid before persisting the link.

## Source Payload Mapping
**Example Sync Logic Field Mapping:**
- Shopify `options` array $\rightarrow$ Maps to `productFeatureApplTypeId="SELECTABLE_FEATURE"` for the parent product.
- Shopify `variant.option1` $\rightarrow$ Maps to `productFeatureApplTypeId="STANDARD_FEATURE"` for the child SKU.

## Dummy Data Example
| productFeatureApplTypeId | parentTypeId | description |
| :--- | :--- | :--- |
| **`MARKETING_ONLY`** | `OPTIONAL_FEATURE` | Features used only for SEO/Marketing text |

## Normalization Explanation
- **Behavioral Abstraction**: By separating the "Role" into a type table, the Java code doesn't have to hardcode "dropdown logic." The UI simply asks: "Show me all features where the application type is `SELECTABLE_FEATURE`."
- **Consistency**: Enforces a strict vocabulary for product characteristics. This ensures that the storefront, the warehouse, and the reporting engine all interpret a feature (like "Red") in the same context.
- **Cache Efficiency**: Because these roles are static and few in number, they are held in the system's L1 cache, making PDP rendering extremely fast.

## Full Sync Flow
1. **Hierarchy Identification**: Sync service detects if the incoming payload represents a Style (Parent) or an SKU (Child).
2. **Role Selection**:
   - Style context $\rightarrow$ Selects `SELECTABLE_FEATURE`.
   - SKU context $\rightarrow$ Selects `STANDARD_FEATURE`.
3. **Integrity Check**: Verifies the selected ID against the `ProductFeatureApplType` table.
4. **Persistence**: The resulting role ID is saved into the `ProductFeatureAppl` junction table.

## Important Business Rules
- **Virtual Requirement**: A product marked `isVirtual='Y'` must have `SELECTABLE_FEATURE` links to be functional on a website; otherwise, the customer has nothing to "select."
- **Variant Identity**: A product marked `isVariant='Y'` must have `STANDARD_FEATURE` links so the system can distinguish it from other variants (e.g., "This specific SKU *is* Red and *is* Small").
- **Mandatory ID**: The primary key `productFeatureApplTypeId` is often used as a constant in Java code for filtering collections.

## Important Observations
- **The "Interactive Switch"**: This entity is effectively the "Toggle" that tells the frontend whether to show a static label or an interactive size-picker.
- **Historical Stability**: These types are almost never modified or deleted, as doing so would break the rendering logic for the entire catalog.

---
**Documentation for `ProductFeatureApplType` is complete. Please provide the next Entity Name.**
---
Viewed EXTRA.MD:768-777
Edited EXTRA.MD

# Entity: ProductFeatureAppl

## Purpose
The `ProductFeatureAppl` entity is the **central junction** that binds a `Product` to its `ProductFeature` records. It is the "glue" that assigns specific characteristics (like Color: Red or Material: Cotton) to an item. Beyond simple linking, it stores the **contextual role** of the relationship (Selectable vs. Standard), making it the engine behind product variations, search filtering, and the dynamic rendering of the Product Detail Page (PDP).

## Where Used In Code
- **Services**: `createProductFeatureAppl` (creates the link), `removeProductFeatureAppl` (expires a link), `getProductFeatures` (the primary query used to build product attribute lists).
- **Handlers**: Shopify/Marketplace Connectors (orchestrates the linking of features to variants during the sync process).
- **APIs**: Product Search APIs (joins `Product` $\leftrightarrow$ `ProductFeatureAppl` $\leftrightarrow$ `ProductFeature` to filter results by color, size, etc.), PDP APIs.
- **Sync Processors**: `ProductFeatureServices.java` (manages the persistence and cleanup of these links during high-volume imports).

## Entity Structure

| Field | Type | Required | Example | Description |
| :--- | :--- | :---: | :--- | :--- |
| **`productId`** | `id` | **Yes** | `10001` | The internal Product ID (Part of Primary Key). |
| **`productFeatureId`** | `id` | **Yes** | `COLOR_RED` | The internal Feature ID (Part of Primary Key). |
| **`productFeatureApplTypeId`** | `id` | No | `STANDARD_FEATURE` | The role of the feature (Fact vs. Choice). |
| **`fromDate`** | `date-time` | **Yes** | `2024-05-12 10:00:00` | When the link became active (Part of Primary Key). |
| **`thruDate`** | `date-time` | No | `NULL` | When the link expired (used for catalog history). |
| **`sequenceNum`** | `number` | No | `1` | Controls the order of features in the UI dropdowns. |
| **`amount`** | `currency-amount` | No | `2.00` | Optional price surcharge for this specific feature application. |

## Seed / Initial Data
The table is populated dynamically as products and variants are created. It relies on `ProductFeatureApplType` (Seeded) for its behavioral logic.

## Relationships
- **Parent Entities**: `Product` (FK), `ProductFeature` (FK), `ProductFeatureApplType` (FK).
- **Relationship Flow**: **Product $\leftrightarrow$ Junction $\leftrightarrow$ Feature**. This bridge is what makes the product catalog searchable. A search for "Red" finds the `productFeatureId`, then finds all `productId` records linked to it in this table.

## Data Population Flow
1. **External Payload**: Shopify reports a new variant with "Color: Crimson".
2. **Dictionary Resolution**: The system finds or creates the `COLOR_CRIMSON` record in the `ProductFeature` table.
3. **Application Loop**: For the product record:
   - Identify the role (e.g., `SELECTABLE_FEATURE` for Virtual, `STANDARD_FEATURE` for Variant).
   - Call `createProductFeatureAppl(productId, "COLOR_CRIMSON", roleId, fromDate)`.
4. **Integrity Check**: Checks for an existing active link between this product and feature to prevent duplicates.
5. **Persistence**: Record is committed to the `PRODUCT_FEATURE_APPL` table.

## Source Payload Mapping
**Example Shopify Sync Mapping:**
- `product_id` (Parent) + `Option Value` $\rightarrow$ `ProductFeatureAppl` (Role: `SELECTABLE`).
- `variant_id` (Child) + `Option Value` $\rightarrow$ `ProductFeatureAppl` (Role: `STANDARD`).

## Dummy Data Example
| productId | productFeatureId | productFeatureApplTypeId | fromDate |
| :--- | :--- | :--- | :--- |
| **`STYLE_HOODIE`** | **`COLOR_RED`** | **`SELECTABLE_FEATURE`** | **`2024-01-01`** |
| **`SKU_HOODIE_RED`** | **`COLOR_RED`** | **`STANDARD_FEATURE`** | **`2024-01-01`** |

## Normalization Explanation
- **Efficient Many-to-Many**: One product has many features, and one feature (Red) is used by thousands of products. This table is the normalized bridge that prevents data redundancy in the `Product` table.
- **Contextual Flexibility**: By storing the `productFeatureApplTypeId` in the junction rather than on the feature, the system allows the *same* feature (e.g., "Cotton") to be a "Selectable Choice" on one product and a "Standard Fact" on another.
- **Temporal Management**: Using `fromDate` and `thruDate` in the PK allows the catalog to evolve. You can remove a feature for a future season without deleting historical data that is tied to past orders.

## Full Sync Flow
1. **Feature Extraction**: Sync service parses all options and tags from the incoming variant.
2. **Dictionary Check**: Ensures every value is present in the `ProductFeature` master list.
3. **Link Creation**: Executes `createProductFeatureAppl` for every valid pair.
4. **Collision Handling**: If a feature is removed in Shopify, the HotWax sync job sets the `thruDate` on the corresponding row in this table.
5. **DB Persistence**: Final commit.

## Important Business Rules
- **Referential Integrity**: A record cannot exist if the `productId` or `productFeatureId` are missing from their respective master tables.
- **Temporal Overlap**: The system logic generally prevents two active rows for the same `productId` + `productFeatureId` to ensure data clarity.
- **Search Visibility**: The storefront search index (Solr) is rebuilt by scanning this table. If a feature link is deleted here, the product will no longer appear when filtering by that characteristic.

## Important Observations
- **Volume**: In large catalogs, this is one of the most record-heavy tables in the database.
- **Performance**: Queries on this table must always include the `thruDate IS NULL` check to avoid pulling stale historical relationships.
- **Ordering**: The `sequenceNum` is used by the frontend to ensure that "Small" appears before "Medium" and "Large" in the size dropdown.

---
**Documentation for `ProductFeatureAppl` is complete. Please provide the next Entity Name.**
---
Viewed EXTRA.MD:925-933
Edited EXTRA.MD

# Entity: ProductCategory

## Purpose
The `ProductCategory` entity is the **logical organizational container** for products. It serves as a named "shelf" or "department" where items are grouped for specific business purposes, most commonly for storefront navigation (Collections), tax calculations (Tax Groups), and promotional eligibility (Sale Categories). It is the structural anchor that allows the system to apply bulk rules (like "10% off all Shoes") to a set of products.

## Where Used In Code
- **Services**: `createProductCategory`, `updateProductCategory`, `getCategoryProducts` (retrieves the membership list).
- **Handlers**: Shopify/BigCommerce Connector (synchronizes external "Collections" or "Brands" into internal category records).
- **APIs**: Storefront Navigation APIs (used to build multi-level menu trees and breadcrumbs).
- **Sync Processors**: `ProductCategoryServices.java` (manages the persistence of category metadata and images).

## Entity Structure

| Field | Type | Required | Example | Description |
| :--- | :--- | :---: | :--- | :--- |
| **`productCategoryId`** | `id` | **Yes** | `MENS_HOODIES` | Unique identifier (Primary Key). |
| **`productCategoryTypeId`** | `id` | **Yes** | `CATALOG_CATEGORY` | Links to `ProductCategoryType` (determines behavior). |
| **`categoryName`** | `name` | No | `Men's Hoodies` | The customer-facing title of the category. |
| **`description`** | `description` | No | `Soft cotton fleece...` | Short description for search results. |
| **`longDescription`** | `very-long` | No | `...` | Full marketing copy for the collection page. |
| **`categoryImageUrl`** | `url` | No | `/images/banners/cat.jpg` | URL for the category banner or thumbnail. |

## Seed / Initial Data
Standard categories like "ROOT" (for the catalog tree) and "TAX_ROOT" are often seeded during initialization. Specific business categories are created during the product/collection sync process.

## Relationships
- **Parent Entity**: `ProductCategoryType` (FK defines if this is a Catalog, Tax, or Promo category).
- **Child Entities**: `ProductCategoryMember` (The junction that puts products inside this category), `ProductCategoryRollup` (The junction that builds sub-category trees).
- **Relationship Flow**: **Catalog $\rightarrow$ Category $\rightarrow$ Product**. This entity is the "Box" that holds the products.

## Data Population Flow
1. **Sync Arrival**: Shopify reports a new "Collection" called "Summer Sale" with a specific URL handle.
2. **Identification**: The sync service checks if a `productCategoryId` matching the handle exists.
3. **Creation**: If new, `createProductCategory` is called with the role `CATALOG_CATEGORY`.
4. **Enrichment**: The title, description, and image URLs are populated from the Shopify payload.
5. **Persistence**: The record is committed to the `PRODUCT_CATEGORY` table.

## Source Payload Mapping
**Example Shopify JSON:**
```json
{
  "title": "Running Shoes",
  "handle": "running-shoes",
  "body_html": "<strong>Engineered for speed...</strong>"
}
```
**Field Mapping:**
- `handle` ("running-shoes") $\rightarrow$ `ProductCategory.productCategoryId`.
- `title` ("Running Shoes") $\rightarrow$ `ProductCategory.categoryName`.
- `body_html` $\rightarrow$ `ProductCategory.longDescription`.

## Dummy Data Example
| productCategoryId | productCategoryTypeId | categoryName |
| :--- | :--- | :--- |
| **`CLEARANCE_SALE`** | **`PROMOTIONAL_CATEGORY`** | **Final Clearance 70% Off** |
| **`VAT_EXEMPT`** | **`TAX_CATEGORY`** | **Tax Exempt Group** |

## Normalization Explanation
- **Multifunctional Design**: By using `productCategoryTypeId`, the same table structure handles both marketing categories and regulatory tax groupings, reducing the total number of tables in the schema.
- **Hierarchy Offloading**: Parent-child relationships between categories (e.g., "Men's Apparel" $\rightarrow$ "Hoodies") are stored in a separate table (`ProductCategoryRollup`). This allows a single category to exist in multiple parent trees.
- **Data Isolation**: Marketing copy and images are stored here, while the frequently updated list of products is stored in a dedicated member table to improve query performance.

## Full Sync Flow
1. **Payload Receipt**: Sync job receives collection data from an external platform.
2. **Validation**: Ensures the identifier is unique and the type is valid.
3. **Service Execution**: Calls the `delegator` to insert or update the record.
4. **Image Handling**: If a new image URL is provided, the system may trigger a download/re-host job.
5. **Cache Invalidation**: Clears the category cache to ensure the storefront navigation updates immediately.

## Important Business Rules
- **Identifier Stability**: The `productCategoryId` (often the Shopify "handle") is used as the SEO URL slug; changing it can break links.
- **Type Mandatory**: A category without a valid `productCategoryTypeId` will be ignored by critical services like the Tax and Promotion engines.
- **Uniqueness**: The Primary Key ensures that you cannot have two categories with the same ID.

## Important Observations
- **SEO Impact**: The `categoryName` and `description` fields are the primary sources for HTML meta tags on the collection pages.
- **Performance**: In very large catalogs, looking up category hierarchies is optimized via a recursive CTE or a flattened cache table in memory.

---
**Documentation for `ProductCategory` is complete. Please provide the next Entity Name.**
---
Viewed EXTRA.MD:1010-1017
Edited EXTRA.MD

# Entity: ProductCategoryMember

## Purpose
The `ProductCategoryMember` entity is the **many-to-many junction** that places a `Product` inside a `ProductCategory`. It is the engine of "Catalog Placement." While a product exists independently in the master registry, this table defines where it appears to the customer. Critically, it also stores the **Sequence Number**, which dictates the exact visual order of products on a collection page (e.g., ensuring a "Best Seller" appears at the top of the list).

## Where Used In Code
- **Services**: `addProductToCategory`, `removeProductFromCategory`, `getCategoryProducts` (retrieves the membership list sorted by `sequenceNum`).
- **Handlers**: Shopify/Marketplace Connectors (orchestrates the linking of imported products to their respective collections/categories).
- **APIs**: Catalog Browsing APIs (used by the storefront to render the "Products in this Category" page).
- **Sync Processors**: `ProductCategoryServices.java` (manages the persistence, expiration, and sorting of memberships).

## Entity Structure

| Field | Type | Required | Example | Description |
| :--- | :--- | :---: | :--- | :--- |
| **`productCategoryId`** | `id` | **Yes** | `SUMMER_SALE` | The internal Category ID (Part of Primary Key). |
| **`productId`** | `id` | **Yes** | `10001` | The internal Product ID (Part of Primary Key). |
| **`fromDate`** | `date-time` | **Yes** | `2024-05-12 10:00:00` | When the membership started (Part of Primary Key). |
| **`thruDate`** | `date-time` | No | `2024-12-31 23:59:59` | When the membership expired (used for seasonal items). |
| **`sequenceNum`** | `number` | No | `1` | Controls the display order on the collection page. |
| **`comments`** | `comment` | No | `Manual override` | Internal notes about why the product was added. |

## Seed / Initial Data
The table is empty upon installation and is populated dynamically as products are organized into categories by the merchandising team or automated sync jobs.

## Relationships
- **Parent Entities**: `ProductCategory` (FK), `Product` (FK).
- **Relationship Flow**: **Category $\leftrightarrow$ Junction $\leftrightarrow$ Product**. This bridge "activates" a product's visibility. Without a record in this table, a product may exist in the system but will be "homeless" and invisible on the storefront menus.

## Data Population Flow
1. **Sync Arrival**: Shopify reports that product "Organic Hoodie" is now part of the "Featured Items" collection.
2. **ID Resolution**: The service resolves the Shopify handle to the internal `productCategoryId` and `productId`.
3. **Collision Check**: The logic checks for an existing active membership (`thruDate is null`) for this specific pair to avoid PK errors.
4. **Service Execution**: `createProductCategoryMember` is called with the current timestamp as the `fromDate`.
5. **Persistence**: The row is committed to the `PRODUCT_CATEGORY_MEMBER` table.

## Source Payload Mapping
**Example Shopify JSON (Collection Membership):**
```json
{
  "product_id": 998877,
  "collection_id": 556677,
  "position": 1
}
```
**Field Mapping:**
- `collection_id` (556677) $\rightarrow$ `ProductCategoryMember.productCategoryId`.
- `product_id` (998877) $\rightarrow$ `ProductCategoryMember.productId`.
- `position` (1) $\rightarrow$ `ProductCategoryMember.sequenceNum`.

## Dummy Data Example
| productCategoryId | productId | sequenceNum | fromDate |
| :--- | :--- | :---: | :--- |
| **`NEW_ARRIVALS`** | **`HOODIE_01`** | **`1`** | **`2024-05-01`** |
| **`NEW_ARRIVALS`** | **`TEE_01`** | **`2`** | **`2024-05-01`** |

## Normalization Explanation
- **Many-to-Many Bridge**: Normalization rules dictate that a product shouldn't have a `categoryId` column if it can belong to multiple categories (e.g., "Men's Apparel" AND "Clearance"). This junction table resolves that complexity.
- **Contextual Sorting**: The `sequenceNum` belongs in the junction, not on the Product. This allows the *same* product to be ranked #1 in "New Arrivals" but ranked #50 in "All Products."
- **Temporal Merchandising**: By utilizing `fromDate` and `thruDate` as part of the membership, merchandising teams can "pre-schedule" products to appear in a category for a specific holiday window.

## Full Sync Flow
1. **Validation**: Ensures that both the Category and Product exist in their respective master tables.
2. **Membership Cleanup**: If the external payload indicates the product was removed from a collection, the system sets the `thruDate` to `now`.
3. **Creation**: A new row is inserted for any new memberships.
4. **Sort Refresh**: The service iterates through the memberships to update `sequenceNum` values to match the external platform's manual sort order.

## Important Business Rules
- **Active Membership Filter**: All storefront queries MUST filter by `(fromDate <= now AND (thruDate IS NULL OR thruDate > now))` to prevent expired items from showing up.
- **Unique Link Constraint**: The combination of Category, Product, and `fromDate` must be unique to prevent data corruption.
- **Primary Category Logic**: In many HotWax services, if a product is in multiple categories, the system uses the one with the lowest `sequenceNum` or the most recent `fromDate` as the "Primary" for breadcrumb generation.

## Important Observations
- **Performance at Scale**: In a catalog with 100k products and 500 categories, this table can easily exceed 1 million rows. Indexing `productCategoryId` and `sequenceNum` is mandatory for sub-second page loads.
- **The "Sort Key"**: For many commerce brands, the `sequenceNum` is the most frequently updated field in the database, as merchandising teams "re-rank" their collections daily.

---
**Documentation for `ProductCategoryMember` is complete. Please provide the next Entity Name.**
---
Viewed EXTRA.MD:1093-1099
Edited EXTRA.MD

# Entity: ProductCategoryRollup

## Purpose
The `ProductCategoryRollup` entity is the **hierarchical engine** of the product catalog. It defines the "Parent-Child" relationships between different categories, allowing for the construction of a deep, multi-level navigation tree (e.g., `Clothing` $\rightarrow$ `Mens` $\rightarrow$ `Tops` $\rightarrow$ `T-Shirts`). Because it is a many-to-many junction, it supports **poly-hierarchy**, meaning a single category (like "Special Offers") can be a child of multiple different parent categories simultaneously.

## Where Used In Code
- **Services**: `createProductCategoryRollup`, `getCategoryParents` (used for breadcrumbs), `getChildCategories` (used for menu generation).
- **Handlers**: Catalog Management Tools (drag-and-drop menu builders in the admin UI).
- **APIs**: Storefront Navigation APIs (recursively traverses this table to build the site-map and mega-menu).
- **Sync Processors**: `CategoryRollupServices.java` (manages the persistence and expiration of hierarchy links during catalog imports).

## Entity Structure

| Field | Type | Required | Example | Description |
| :--- | :--- | :---: | :--- | :--- |
| **`productCategoryId`** | `id` | **Yes** | `TOPS_CAT` | The **Child** category ID (Part of Primary Key). |
| **`parentProductCategoryId`** | `id` | **Yes** | `MENS_APPAREL` | The **Parent** category ID (Part of Primary Key). |
| **`fromDate`** | `date-time` | **Yes** | `2024-05-12 10:00:00` | When the relationship became active (Part of PK). |
| **`thruDate`** | `date-time` | No | `NULL` | When the relationship was dissolved. |
| **`sequenceNum`** | `number` | No | `2` | Controls the order of sub-categories in a menu. |

## Seed / Initial Data
The table is typically empty upon installation. The hierarchy is defined by the commerce managers as they build out their specific catalog structures.

## Relationships
- **Parent Entities**: `ProductCategory` (as `productCategoryId`), `ProductCategory` (as `parentProductCategoryId`).
- **Relationship Flow**: **Parent Category $\rightarrow$ Rollup $\rightarrow$ Child Category**. This creates a directed graph. A search starting at a "Root" category will flow through these records to find all nested products.

## Data Population Flow
1. **Catalog Design**: A merchandiser decides that the "Laptops" category should fall under the "Electronics" parent.
2. **First ID**: The system retrieves the internal ID for `ELECTRONICS`.
3. **Second ID**: The system retrieves the internal ID for `LAPTOPS`.
4. **Circular Check**: A Java service validates that `LAPTOPS` is not already a parent of `ELECTRONICS` to prevent infinite loops.
5. **Persistence**: `createProductCategoryRollup` is executed, and the link is committed to the database.

## Source Payload Mapping
**Example External Payload (Menu Structure):**
```json
{
  "parent_node": "outdoor-gear",
  "child_node": "camping-tents",
  "display_order": 5
}
```
**Field Mapping:**
- `parent_node` ("outdoor-gear") $\rightarrow$ `ProductCategoryRollup.parentProductCategoryId`.
- `child_node` ("camping-tents") $\rightarrow$ `ProductCategoryRollup.productCategoryId`.
- `display_order` (5) $\rightarrow$ `ProductCategoryRollup.sequenceNum`.

## Dummy Data Example
| productCategoryId | parentProductCategoryId | sequenceNum | fromDate |
| :--- | :--- | :---: | :--- |
| **`MENS_TOPS`** | **`MENS_APPAREL`** | **`1`** | **`2024-01-01`** |
| **`MENS_BOTTOMS`** | **`MENS_APPAREL`** | **`2`** | **`2024-01-01`** |

## Normalization Explanation
- **Poly-hierarchy Support**: Instead of a flat `parentCategoryId` column in the `ProductCategory` table (which would limit a category to only one parent), this junction allows for complex, overlapping trees.
- **Temporal Flexibility**: The use of `fromDate` and `thruDate` allows merchandisers to schedule seasonal menu changes. A "Holiday Gift Guide" can be rolled up under the "Main Menu" only for the months of November and December.
- **Sequence Independence**: Storing `sequenceNum` in the junction allows sub-categories to be ranked differently depending on where they are viewed (e.g., "Sale" might be the first item under "Clearance" but the last item under "Main Menu").

## Full Sync Flow
1. **Integrity Validation**: Ensures both the child and parent categories exist in the `ProductCategory` master table.
2. **Cycle Detection**: The service traverses the existing tree to ensure that the new link does not create a recursive loop.
3. **Expiration**: If a category is moved to a new parent in the source system, the old link's `thruDate` is set to `now`.
4. **Creation**: The new rollup record is persisted.
5. **Cache Refresh**: Triggers a rebuild of the "Flattened Category Tree" in memory to maintain high-speed storefront navigation.

## Important Business Rules
- **The "Top-Level" Rule**: Categories that have no records in the `parentProductCategoryId` column (or are children of the Root) are treated as "Department" or "Top-Level" categories.
- **Active Link Filtering**: Queries must always filter by `thruDate IS NULL` to avoid rendering stale navigation paths.
- **Unique Entry**: For a specific parent-child pair, only one active relationship of the same type can exist at one time.

## Important Observations
- **Recursive CTEs**: In modern SQL implementations of HotWax, recursive Common Table Expressions are often used on this table to find all products nested deep within multiple sub-categories.
- **The "Mega-Menu" Engine**: The entire structure of a modern e-commerce "Mega-Menu" is simply a visual representation of the data rows in this table.

---
**Documentation for `ProductCategoryRollup` is complete. Please provide the next Entity Name.**
---
Viewed EXTRA.MD:1094-1100
Edited EXTRA.MD

# Entity: ProductStore

## Purpose
The `ProductStore` entity is the **highest-level configuration hub** for a commerce environment. It defines the "Rules of Engagement" for a specific sales channel, such as a Shopify storefront, an Amazon marketplace, or a physical retail outlet. It centralizes critical business logic settings including taxation, inventory fulfillment strategies, payment gateway configurations, and default currencies. Every order placed in the system must be associated with a `ProductStore`, as it provides the necessary context for the fulfillment engine to process the transaction.

## Where Used In Code
- **Services**: `createProductStore`, `checkInventoryAvailability` (uses the store's inventory flags), `calculateTax`, `reserveStoreInventory`.
- **Handlers**: Order Entry (every `OrderHeader` must point to a `productStoreId`), Payment Processing (references store-level gateway settings).
- **APIs**: Shopify/BigCommerce Connector (links a specific external store URL to an internal `productStoreId` to route orders and inventory).
- **Schedulers**: Fulfillment/Auto-Allocation jobs (processes orders based on the specific warehouse rules defined in the store record).

## Entity Structure

| Field | Type | Required | Example | Description |
| :--- | :--- | :---: | :--- | :--- |
| **`productStoreId`** | `id` | **Yes** | `SHOPIFY_USA` | Unique identifier (Primary Key). |
| **`storeName`** | `name` | No | `HotWax US Online` | Human-readable name of the sales channel. |
| **`inventoryFacilityId`** | `id` | No | `MAIN_WAREHOUSE` | The default warehouse for this store's orders. |
| **`defaultCurrencyUomId`** | `id` | No | `USD` | The primary currency used for transactions. |
| **`payToPartyId`** | `id` | No | `COMPANY_USA` | The internal legal entity receiving the revenue. |
| **`checkInventory`** | `indicator` | No | `Y` | Flag to enable/disable stock checks during order entry. |
| **`reserveInventory`** | `indicator` | No | `Y` | Flag to determine if stock should be immediately "locked" for an order. |
| **`isImmediatelyFulfilled`** | `indicator` | No | `N` | 'Y' for physical POS stores (where items are taken on the spot). |

## Seed / Initial Data
Standard installs usually include a `DEFAULT_STORE` to allow basic system testing. However, production stores are configured manually during the enterprise implementation phase.

## Relationships
- **Parent Entities**: `Party` (via `payToPartyId`), `Facility` (via `inventoryFacilityId`), `Uom` (via `defaultCurrencyUomId`).
- **Child Entities**: `ProductStoreCatalog` (Links the store to its sellable products), `ProductStoreFacility` (A list of all warehouses this store is allowed to ship from), `OrderHeader`.
- **Relationship Flow**: **Store $\rightarrow$ Catalog $\rightarrow$ Products**. The Store acts as the "Environment" that exposes specific Catalogs to the customer.

## Data Population Flow
1. **Configuration**: An admin defines the store parameters (e.g., "Allow backorders", "Use US Tax Service").
2. **Identification**: A unique, human-readable ID is assigned (e.g., `RETAIL_NYC_01`).
3. **Fulfillment Link**: The store is mapped to a primary `inventoryFacilityId` for stock availability lookups.
4. **Integration Setup**: The Shopify Connector is configured to map all orders from `myshopify.com` to this specific `productStoreId`.
5. **Persistence**: The record is saved into the `PRODUCT_STORE` table.

## Source Payload Mapping
**Example Connector/Configuration Mapping:**
- Shopify Store Name $\rightarrow$ `ProductStore.storeName`.
- Shopify Currency $\rightarrow$ `ProductStore.defaultCurrencyUomId`.
- Internal OMS Facility $\rightarrow$ `ProductStore.inventoryFacilityId`.

## Dummy Data Example
| productStoreId | storeName | inventoryFacilityId | defaultCurrencyUomId |
| :--- | :--- | :--- | :--- |
| **`MARKETPLACE_AMZ`** | **Amazon North America** | **AMZ_FBA_WH** | **USD** |

## Normalization Explanation
- **Centralized Strategy**: Instead of storing fulfillment logic (like "Reserve Stock") on every individual Product or Order, it is stored once here to ensure consistent behavior across the entire sales channel.
- **Channel Multi-Tenancy**: Allows a single ERP instance to run a US Shopify Store, a UK Shopify Store, and a physical flagship store, each with its own tax rules and warehouses, while sharing the same underlying `Product` table.
- **Transactional Logic Offloading**: Heavy settings like `checkInventory` are read once and cached, preventing the system from needing to re-evaluate business logic for every line item in an order.

## Full Sync Flow
1. **Context Resolution**: When a new order payload is received, the sync service identifies the source and loads the corresponding `ProductStore` record.
2. **Behavioral Branching**: If `checkInventory='Y'`, the service triggers a call to the inventory module. If `N`, it skips stock validation.
3. **Accounting Routing**: The service uses `payToPartyId` to ensure the financial entries for the order are attributed to the correct internal company.
4. **Fulfillment Allocation**: The `inventoryFacilityId` is used to create the initial `OrderItemShipGrp` to tell the warehouse team where to pick the items.

## Important Business Rules
- **Referential Integrity**: An `OrderHeader` cannot be created without a valid `productStoreId`.
- **Currency Enforcement**: All price calculations and payment authorizations are driven by the `defaultCurrencyUomId` defined in this table.
- **Stock Master Switch**: The `checkInventory` flag is the primary control for preventing overselling on digital storefronts.

## Important Observations
- **The "Source of Power"**: This entity is essentially the "Brain" of the HotWax OMS. It dictates how the system perceives and reacts to incoming demand.
- **Caching**: This table is held in the L1 entity cache by default. Any changes to critical flags (like switching `reserveInventory` to `N`) require a cache clear to take effect.
- **Group Logic**: `ProductStoreGroup` is often used alongside this table to apply common pricing or promotional rules to multiple stores simultaneously.

---
**Documentation for `ProductStore` is complete. Please provide the next Entity Name.**
---
Viewed EXTRA.MD:1253-1258
Edited EXTRA.MD

# Entity: ProdCatalog

## Purpose
The `ProdCatalog` entity is the **high-level container** for all product categories. While a `ProductStore` defines the operational rules (taxes, shipping, currency), the `ProdCatalog` defines the **commercial content**. It acts as a logical umbrella that groups multiple category trees (e.g., Navigation, Promotions, Tax) into a single unit. This structure allows a business to maintain different "Catalog Versions" (e.g., "Spring 2024" vs. "Winter 2024") and easily assign them to one or more sales channels.

## Where Used In Code
- **Services**: `createProdCatalog`, `updateProdCatalog`, `getProdCatalogCategories` (retrieves all root-level categories for the catalog).
- **Handlers**: Backend Catalog Manager (used by merchandisers to switch contexts between different product assortments).
- **APIs**: Storefront APIs (used at the start of a user session to resolve which product tree should be rendered).
- **Sync Processors**: `ProductCatalogServices.java` (manages the persistence of catalog-level metadata and descriptive names).

## Entity Structure

| Field | Type | Required | Example | Description |
| :--- | :--- | :---: | :--- | :--- |
| **`prodCatalogId`** | `id` | **Yes** | `MAIN_CATALOG` | Unique identifier (Primary Key). |
| **`catalogName`** | `name` | No | `Global Master Catalog` | Human-readable name used in the admin interface. |
| **`useCassandra`** | `indicator` | No | `N` | Legacy infrastructure flag (Standard: `N`). |

## Seed / Initial Data
Most HotWax implementations seed a `DEMO_CATALOG` or a `MAIN_CATALOG` to provide a baseline structure for initial configuration.

## Relationships
- **Parent Entities**: None (This is a top-level organizational entity).
- **Child Entities**: `ProdCatalogCategory` (The junction that links root categories to this catalog), `ProductStoreCatalog` (The junction that links this catalog to specific stores).
- **Relationship Flow**: **Store $\rightarrow$ StoreCatalog $\rightarrow$ ProdCatalog $\rightarrow$ CatalogCategory $\rightarrow$ Category $\rightarrow$ Product**.

## Data Population Flow
1. **Creation**: An admin creates a "Master Catalog" record to represent the company's entire inventory.
2. **Category Assignment**: The "Root" navigation category is linked to this catalog.
3. **Store Linkage**: The catalog is associated with a specific `ProductStore` (e.g., "Shopify US").
4. **Validation**: The system ensures the `prodCatalogId` is unique and the name is descriptive.
5. **Persistence**: The record is saved in the `PROD_CATALOG` table.

## Source Payload Mapping
**Example Business Mapping:**
- A Shopify "Online Store" sales channel $\rightarrow$ Typically mapped to a single `ProdCatalog` that contains all the Shopify "Collections."

## Dummy Data Example
| prodCatalogId | catalogName |
| :--- | :--- |
| **`B2B_CATALOG`** | **Wholesale/Professional Catalog** |

## Normalization Explanation
- **Separation of Concerns**: Decouples the "What" (the items being sold) from the "How" (the store's tax/currency rules). This allows the same product assortment to be sold across different countries/stores with different rules.
- **Assortment Management**: Provides a clean entry point for bulk updates. Instead of updating 100 categories, a merchandiser can simply point a `ProductStore` to a new `ProdCatalog` to change the entire website assortment.
- **Administrative Abstraction**: Allows for granular permissions. A user can be assigned as a "Manager" for a specific `prodCatalogId`, limiting their ability to edit other brand catalogs.

## Full Sync Flow
1. **Catalog Resolution**: When a customer lands on the website, the system identifies the store and then looks up the linked `ProdCatalog`.
2. **Tree Traversal**: The system loads all `ProductCategory` records associated with that catalog via the `ProdCatalogCategory` junction.
3. **Rendering**: The navigable menu (Tops, Bottoms, Accessories) is built by traversing the categories found within this catalog's scope.

## Important Business Rules
- **Unique Entry**: Each catalog must have a unique ID.
- **Visibility Chain**: For a product to be visible on the storefront, it must be in a Category, which must be in a `ProdCatalog`, which must be linked to the active `ProductStore`. If any link in this chain is missing, the product is "orphaned."

## Important Observations
- **Versioning and Seasons**: Many brands use this entity to manage seasonal transitions. They build a "Holiday Catalog" in the background and then "flip the switch" by updating the `ProductStoreCatalog` link on the launch date.
- **Caching**: This entity is heavily cached. Changes to the catalog name or structure often require a manual cache refresh to appear on high-traffic storefronts.

---
**Documentation for `ProdCatalog` is complete. Please provide the next Entity Name.**
---
# Entity: ProductStoreCatalog

## Purpose
The `ProductStoreCatalog` entity is the **many-to-many junction** that links a `ProductStore` (the rules) to a `ProdCatalog` (the content). It is the final "Selection Engine" that determines which specific product assortment is visible on which sales channel. This table allows for complex configurations, such as a store showing a primary "Master Catalog" alongside a temporary "Flash Sale Catalog," while also defining the visual order (`sequenceNum`) in which these catalogs appear to the customer.

## Where Used In Code
- **Services**: `createProductStoreCatalog`, `getStoreCatalogs` (the primary query executed during storefront initialization to resolve the product scope).
- **Handlers**: Storefront Session Manager (identifies which catalogs to load based on the current store context).
- **APIs**: Catalog Browsing APIs (used to resolve the list of browsable categories and items for a specific store).
- **Sync Processors**: `ProductStoreCatalogServices.java` (manages the persistence, expiration, and sorting of these critical links).

## Entity Structure

| Field | Type | Required | Example | Description |
| :--- | :--- | :---: | :--- | :--- |
| **`productStoreId`** | `id` | **Yes** | `SHOPIFY_USA` | The internal Store ID (Part of Primary Key). |
| **`prodCatalogId`** | `id` | **Yes** | `SPRING_2024` | The internal Catalog ID (Part of Primary Key). |
| **`fromDate`** | `date-time` | **Yes** | `2024-05-12 10:00:00` | When the link became active (Part of Primary Key). |
| **`thruDate`** | `date-time` | No | `2024-12-31 23:59:59` | When the link was dissolved (used for seasonal swaps). |
| **`sequenceNum`** | `number` | No | `1` | Defines the display order if a store has multiple catalogs. |

## Seed / Initial Data
Standard seed data often includes a link between a `DEFAULT_STORE` and a `MAIN_CATALOG` to ensure the system is functional out of the box. Production links are created during the final catalog setup.

## Relationships
- **Parent Entities**: `ProductStore` (FK), `ProdCatalog` (FK).
- **Relationship Flow**: **Store $\leftrightarrow$ Junction $\leftrightarrow$ Catalog**. This is the definitive bridge in the HotWax commerce hierarchy. It tells the system: "For the USA Shopify Store, use the Spring 2024 collection."

## Data Population Flow
1. **Catalog Readiness**: A merchandiser finishes building a new "Sale Catalog."
2. **Identification**: The system retrieves the `productStoreId` (e.g., `SHOPIFY_STORE`) and the `prodCatalogId` (e.g., `SALE_CAT`).
3. **Collision Check**: A service verifies that there isn't already an active link for this pair to prevent database errors.
4. **Service Execution**: `createProductStoreCatalog` is called with a `fromDate` of `now`.
5. **Persistence**: The record is committed to the `PRODUCT_STORE_CATALOG` table.

## Source Payload Mapping
**Example Connector Logic Mapping:**
- If the incoming Shopify Order is from `uk-store.myshopify.com` $\rightarrow$ The system loads the `ProductStoreCatalog` where `productStoreId="SHOPIFY_UK"`.

## Dummy Data Example
| productStoreId | prodCatalogId | sequenceNum | fromDate |
| :--- | :--- | :---: | :--- |
| **`RETAIL_NYC`** | **`POS_CATALOG`** | **`1`** | **`2024-01-01`** |

## Normalization Explanation
- **Many-to-Many Flexibility**: One store can host multiple catalogs (e.g., a "Regular" catalog and a "Wholesale" catalog) and one master catalog can be reused across dozens of physical and digital stores.
- **Scheduled Assortments**: By utilizing `fromDate` and `thruDate`, retailers can pre-program catalog swaps. The "Holiday Catalog" can be set to automatically expire at midnight on Dec 26th, instantly reverting the store to the "Regular Catalog."
- **Separation of Concerns**: This junction ensures that the product assortment can be changed without touching the store's sensitive tax, shipping, or payment settings in the `ProductStore` table.

## Full Sync Flow
1. **Integrity Validation**: The service confirms that both the Store ID and Catalog ID exist in the system.
2. **Temporal Cleanup**: If a catalog is being replaced, the system sets the `thruDate` on the old link to the current time.
3. **Creation**: A new row is inserted into the junction table for the new assignment.
4. **Cache Invalidation**: The "Store-Catalog Map" is cleared from memory, forcing the website to refresh its navigation and product lists.

## Important Business Rules
- **Active Filter Mandatory**: Storefront logic must always filter by `(fromDate <= now AND (thruDate IS NULL OR thruDate > now))` to ensure only the currently intended products are sellable.
- **The "Empty Store" Condition**: If this table has no records for a specific `productStoreId`, that store will appear to have zero products, even if the database contains millions of items.
- **Sequence Logic**: If a product exists in multiple catalogs linked to the same store, the system typically uses the catalog with the lowest `sequenceNum` for display preferences.

## Important Observations
- **The "Connection Engine"**: This is the smallest but perhaps most critical table in the catalog hierarchy. It is the "Master Switch" that turns a collection of products into a functional storefront.
- **Performance**: This table is tiny (usually < 100 rows per brand) and is held in the L1 entity cache for near-instant resolution.

---
**Documentation for `ProductStoreCatalog` is complete. Please provide the next Entity Name.**
---
Viewed EXTRA.MD:1385-1392
Edited EXTRA.MD

# Entity: ProductPrice

## Purpose
The `ProductPrice` entity is the **central valuation registry** for the entire commerce system. It stores the monetary value of products across multiple dimensions, including different currencies, price types (e.g., MSRP, List Price, Sale Price), and usage contexts (e.g., Purchase vs. Recurring). Because it utilizes a **temporal Primary Key** (`fromDate`), it also functions as a permanent audit trail, allowing the system to historically reconstruct exactly what a product cost at any specific point in time—a feature critical for financial reconciliation and order auditing.

## Where Used In Code
- **Services**: `calculateProductPrice` (the most critical service, used during checkout to resolve the current price), `createProductPrice`, `updateProductPrice`.
- **Handlers**: Order Entry (fetches the `DEFAULT_PRICE`), Storefront PDP (displays the "Compare at" `LIST_PRICE` alongside the current price).
- **APIs**: Shopify/Marketplace Sync (broadcasts price changes to external platforms whenever a row is updated in HotWax).
- **Sync Processors**: `ProductPriceServices.java` (manages high-volume price updates from ERP feeds or bulk CSV uploads).

## Entity Structure

| Field | Type | Required | Example | Description |
| :--- | :--- | :---: | :--- | :--- |
| **`productId`** | `id` | **Yes** | `10001` | The internal ID of the product (Part of PK). |
| **`productPriceTypeId`** | `id` | **Yes** | `DEFAULT_PRICE` | Category (e.g., Default, List, MSRP) (Part of PK). |
| **`productPricePurposeId`** | `id` | **Yes** | `PURCHASE` | Usage context (Standard: `PURCHASE`) (Part of PK). |
| **`currencyUomId`** | `id` | **Yes** | `USD` | Transactional currency (Part of PK). |
| **`fromDate`** | `date-time` | **Yes** | `2024-05-12 10:00` | Start of this price's validity (Part of PK). |
| **`thruDate`** | `date-time` | No | `NULL` | End of validity (used for expiring old prices). |
| **`price`** | `currency-amount` | **Yes** | `49.99` | The actual numeric value in the specified currency. |

## Seed / Initial Data
The behavioral types for pricing are seeded during system initialization.

**Default Records:**
| productPriceTypeId | description | Business Use Case |
| :--- | :--- | :--- |
| `DEFAULT_PRICE` | Base Price | The actual selling price on the storefront. |
| `LIST_PRICE` | Retail/List | Used for "Strike-through" pricing (MSRP). |
| `AVERAGE_COST` | Cost Price | Internal cost used for margin reporting. |

## Relationships
- **Parent Entities**: `Product` (FK), `ProductPriceType` (FK), `Uom` (Currency FK).
- **Relationship Flow**: **Product $\rightarrow$ Price**. One product can have dozens of rows in this table representing different currencies (USD, EUR, CAD) and price types simultaneously.

## Data Population Flow
1. **Sync Arrival**: An external ERP feed reports that "Blue Hoodie" now costs $45.00 instead of $50.00.
2. **Current State Lookup**: The system identifies the currently active row for `productId="BLUE_HOODIE"`, `type="DEFAULT_PRICE"`, and `currency="USD"`.
3. **Versioning Logic**:
   - The `thruDate` on the existing row is set to the current timestamp.
   - A new row is inserted with `price=45.00` and `fromDate=now`.
4. **Persistence**: The update and insert are committed as a single transaction to ensure price integrity.

## Source Payload Mapping
**Example Shopify Sync Mapping:**
- `shopify.price` $\rightarrow$ `ProductPrice(productPriceTypeId="DEFAULT_PRICE")`.
- `shopify.compare_at_price` $\rightarrow$ `ProductPrice(productPriceTypeId="LIST_PRICE")`.

## Dummy Data Example
| productId | priceTypeId | currencyUomId | price | fromDate |
| :--- | :--- | :---: | :---: | :--- |
| **`TEE_01`** | **`DEFAULT_PRICE`** | **`USD`** | **`19.99`** | **`2024-01-01`** |
| **`TEE_01`** | **`DEFAULT_PRICE`** | **`CAD`** | **`24.99`** | **`2024-01-01`** |

## Normalization Explanation
- **Dimensionality**: Instead of having columns like `price_usd`, `price_sale_usd`, and `price_eur` (which would require schema changes for every new currency), the vertical structure allows for infinite pricing scenarios.
- **Immutability of History**: By including `fromDate` in the Primary Key, the system never truly "overwrites" a price. It "expires" the old one and "births" a new one, preserving the financial history of the product.
- **Contextual Pricing**: Allows for complex setups where the same product has a `DEFAULT_PRICE` for retail customers and a `WHOLESALE_PRICE` for business partners, all managed within the same table.

## Full Sync Flow
1. **Calculation Request**: The `calculateProductPrice` service is called during checkout.
2. **Temporal Selection**: The service filters for the row where `(fromDate <= now AND (thruDate IS NULL OR thruDate > now))`.
3. **Priority Logic**: If multiple valid prices are found, the system follows a priority hierarchy (e.g., Sale Price overrides Default Price).
4. **Output**: The resolved price is passed to the `OrderItem` for persistent storage on the order.

## Important Business Rules
- **PK Unique Requirement**: The combination of Product, Type, Purpose, Currency, and `fromDate` must be unique.
- **One Active Price**: Business logic enforces that only one row of a specific Type/Purpose/Currency can be active at any given time.
- **Currency Absolute**: HotWax does not do "on-the-fly" currency conversion in this table; every currency must have its own absolute numeric row.

## Important Observations
- **The "Price Cache"**: Because price lookup is extremely high-traffic, HotWax utilizes an L2 cache specifically for resolved prices to prevent database bottlenecks during sales events.
- **Flash Sale Engine**: Merchandisers can pre-load prices with a future `fromDate` and `thruDate`, allowing "Black Friday" pricing to activate and deactivate automatically without manual intervention.

---
**Documentation for `ProductPrice` is complete. Please provide the next Entity Name.**
---
Viewed EXTRA.MD:1467-1474
Edited EXTRA.MD

# Entity: InventoryItem

## Purpose
The `InventoryItem` entity represents the **physical existence** of a product within a specific storage location. While the `Product` entity is a conceptual master record, the `InventoryItem` is the tangible "parcel" of stock that can be picked, packed, and shipped. It tracks the two most critical metrics in supply chain management: **Quantity on Hand (QOH)** (physical units in the building) and **Available to Promise (ATP)** (sellable units after subtracting open orders). It serves as the primary data source for inventory synchronization with external storefronts like Shopify.

## Where Used In Code
- **Services**: `getInventoryAvailableByFacility` (used to display "In Stock" labels), `createInventoryItem` (called during warehouse receiving), `reserveInventoryItem` (decrements ATP when an order is placed).
- **Handlers**: Order Fulfillment (reserves a specific `inventoryItemId` to an order line item), Physical Count Handlers (adjusts QOH based on manual audits).
- **APIs**: Inventory Sync APIs (calculates the aggregate ATP across all sellable facilities to update external marketplaces).
- **Sync Processors**: `InventoryImportServices.java` (orchestrates high-frequency stock updates from 3PLs or warehouse management systems).

## Entity Structure

| Field | Type | Required | Example | Description |
| :--- | :--- | :---: | :--- | :--- |
| **`inventoryItemId`** | `id` | **Yes** | `90001` | Unique internal primary key. |
| **`inventoryItemTypeId`** | `id` | No | `NON_SERIAL_INV_ITEM` | Distinguishes between standard and serialized items. |
| **`productId`** | `id` | No | `10001` | Links the stock to a specific product record. |
| **`facilityId`** | `id` | No | `PRIMARY_WH` | The warehouse or store where this stock is located. |
| **`quantityOnHandTotal`** | `fixed-point` | No | `150.0` | Total physical units currently on the shelf (QOH). |
| **`availableToPromiseTotal`** | `fixed-point` | No | `120.0` | Units available for new sales (ATP). |
| **`unitCost`** | `currency-amount` | No | `14.25` | The purchase cost of this specific batch (for COGS). |

## Seed / Initial Data
The core item types are seeded to establish the rules for how stock is tracked.

**Default Records:**
| inventoryItemTypeId | description |
| :--- | :--- |
| `NON_SERIAL_INV_ITEM` | Standard inventory (tracked by quantity). |
| `SERIALIZED_INV_ITEM` | Each unit has a unique ID (e.g., Electronics). |

## Relationships
- **Parent Entities**: `Product` (FK), `Facility` (FK).
- **Child Entities**: `InventoryItemDetail` (The transaction ledger), `OrderItemShipGrpInvRes` (The link to specific orders).
- **Relationship Flow**: **Product + Facility $\rightarrow$ InventoryItem**. This structure allows the system to track the same product across 50 different locations, each with its own local stock level and unit cost.

## Data Population Flow
1. **Warehouse Receipt**: A shipment of 500 "Blue Hoodies" arrives at the "West Coast Facility."
2. **Entity Lookup**: The system checks if an `InventoryItem` record already exists for this Product/Facility combination.
3. **Transaction**:
   - If existing: The `quantityOnHandTotal` is incremented.
   - If new: `createInventoryItem` generates a new record.
4. **Audit Logging**: A row is automatically written to the `InventoryItemDetail` table to record the "Reason" (e.g., Purchase Order Receipt) and the timestamp.
5. **Persistence**: The updated stock level is committed to the database.

## Source Payload Mapping
**Example WMS (Warehouse Management System) JSON:**
```json
{
  "sku": "HOODIE-BLU-LG",
  "location_code": "WH-01",
  "physical_qty": 250
}
```
**Field Mapping:**
- `sku` $\rightarrow$ `InventoryItem.productId` (after internal ID resolution).
- `location_code` $\rightarrow$ `InventoryItem.facilityId`.
- `physical_qty` $\rightarrow$ `InventoryItem.quantityOnHandTotal`.

## Dummy Data Example
| inventoryItemId | productId | facilityId | quantityOnHandTotal | availableToPromiseTotal |
| :--- | :--- | :--- | :---: | :---: |
| **`9901`** | **`TEE_01`** | **`RETAIL_NYC`** | **`50.0`** | **`42.0`** |

## Normalization Explanation
- **Physical vs. Conceptual Split**: Separating the "Idea" of a product (Product table) from its "Physical presence" (Inventory table) allows the system to manage complex multi-warehouse operations without bloating the core catalog.
- **Accounting Isolation**: Each `inventoryItemId` can hold its own `unitCost`. This is essential for companies using "Moving Average" or "FIFO" accounting, where the cost of the same product might differ between shipments.
- **Lock Contention Reduction**: Stock levels change constantly. By keeping them in a dedicated table, the system avoids locking the `Product` master table during high-volume sales, ensuring the website remains responsive.

## Full Sync Flow
1. **Trigger**: An inventory sync job runs (e.g., every 15 minutes).
2. **Aggregation**: The service sums the `availableToPromiseTotal` for a `productId` across all "Sellable" facilities linked to a `ProductStore`.
3. **Broadcast**: The resulting total (e.g., 500 units) is pushed to Shopify/Amazon.
4. **Safety Buffer**: Business logic often subtracts a "Safety Stock" value from this total before pushing it to external platforms to prevent overselling.

## Important Business Rules
- **ATP Calculation**: `ATP = QOH - (Total Reservations)`. If an item is on a shelf but already "promised" to a customer order, it is not available to promise to a new customer.
- **Facility Integrity**: An `InventoryItem` must be linked to a valid `Facility` to be considered for fulfillment.
- **Costing Source**: The `unitCost` field on this record is the primary source of truth for calculating the "Value of Inventory" on the balance sheet.

## Important Observations
- **The "Real-Time" Ledger**: This is the most volatile table in the entire HotWax system. It is updated by scanners in the warehouse, order clicks on the web, and returns at the POS.
- **Service-Only Updates**: You should **never** update the quantity fields via direct SQL. Always use the Java service layer to ensure the transaction history is preserved in the `InventoryItemDetail` table.

---
**Documentation for `InventoryItem` is complete. Please provide the next Entity Name.**
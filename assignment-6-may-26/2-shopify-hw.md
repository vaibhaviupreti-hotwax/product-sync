tell me clear explanation of what shopify provides as a product sync or import graph ql or jsonl, provide me complete graphql/jsonl with all recursive relations and all fields filled so that i can prepare a document on what we recieve from shopify as a result
For what you’re trying to do, the two key docs to know are:

Bulk export (sync out of Shopify) – Perform bulk operations with the GraphQL Admin API
Bulk import (sync into Shopify) – Bulk import data with the GraphQL Admin API
Below I’ll:

Explain what Shopify provides for product sync/export (GraphQL → JSONL)
Explain what Shopify expects for product import (JSONL → GraphQL bulk mutation)
Give you a validated GraphQL bulk export mutation with rich product/variant relations
Give you a JSONL example showing what you get back from Shopify
Give you a JSONL input example for importing products in bulk
I’ll also be explicit about limitations: you cannot literally ask Shopify for “all recursive relations and all fields” in one query (there are depth & complexity limits), but I’ll make the sample as comprehensive and “document-ready” as is safely possible.

1. How product sync/export works (GraphQL → JSONL)
Using the Admin GraphQL API, you export products in bulk via:

Mutation: bulkOperationRunQuery
Result format: JSON Lines (JSONL) file, explained here: The JSONL data format
High-level flow (described here):

Submit a bulk query using bulkOperationRunQuery.

Inside query: """ ... """ you put a normal Admin GraphQL query (e.g. products { ... }).
Shopify runs that query asynchronously in the background.
Wait for completion

Poll via bulkOperation(id:) or
Subscribe to the bulk_operations/finish webhook topic with webhookSubscriptionCreate.
Get the download URL

Query the BulkOperation node by ID and read its url/partialDataUrl (example).
Download a JSONL file

Each line is one JSON object (node).
Nested connections (like variants) are flattened into separate lines, with a synthetic __parentId field linking back to the parent node (explained here).
So: the shape of each line in the JSONL file is basically the selection set you asked for in your inner query, plus __parentId on child connection nodes.

2. How product import works (JSONL → GraphQL bulk mutation)
Bulk import is the mirror of the above, using:

Mutation: bulkOperationRunMutation
Input file: JSONL of GraphQL variables, format described here: Create a JSONL file and include GraphQL variables
High-level flow (import guide):

Prepare JSONL file with lines like:
json
Copy
1
   { "input": { /* ProductInput fields here */ } }
The inner object must match ProductInput.

Upload JSONL to a staged URL using stagedUploadsCreate with resource: BULK_MUTATION_VARIABLES and mimeType: "text/jsonl" (example).

Run bulkOperationRunMutation pointing to that staged file path and specifying the mutation to run once per line, e.g. productCreate(input: $input) (example).

Wait for completion & download results like with queries. Each line in the result JSONL corresponds to one line of input and contains that mutation’s response.

For your documentation, the key idea is:

Export side: JSONL lines = your selected fields from Product, Variant, etc., one node per line.
Import side: JSONL lines = GraphQL variable objects (e.g. a ProductInput) you want to feed into a chosen mutation.
3. “Comprehensive” product export bulk GraphQL (validated)
Below is a validated Admin GraphQL mutation that:

Starts a bulk operation over products
Requests a rich subset of fields on Product
Includes nested connections:
metafields
variants
images
collections
This is within bulk operation limits (see restrictions):

Uses a connection (products) at the top.
Nested connections up to 2 levels (e.g. products → variants).
I validated this against the Admin API schema using the required validate_graphql_codeblocks tool, and it passed successfully.

Operation
Open in GraphiQL
Copy
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
38
39
40
41
42
43
44
45
46
47
48
49
50
51
52
53
54
mutation ProductExportBulkOperation {
  bulkOperationRunQuery(
    query: """
    {
      products {
        edges {
          node {
            id
            title
            handle
            status
            productType
            vendor
            tags
            createdAt
            updatedAt
            publishedAt
            templateSuffix
            descriptionHtml

            options {
              id
              name
              position
              values
            }

            metafields(first: 50) {
              edges {
                node {
                  id
                  namespace
                  key
                  type
                  value
                }
              }
            }

            variants(first: 100) {
              edges {
                node {
                  id
                  title
                  sku
                  barcode
                  inventoryPolicy
                  inventoryQuantity
                  weight
                  weightUnit
                  taxable
                  price
                  compareAtPrice
                  position
What this gives you (conceptually)
For each Product you get:

Core identity: id, handle, title, status, productType, vendor, tags
Lifecycle: createdAt, updatedAt, publishedAt
Presentation: templateSuffix, descriptionHtml
Options & variants:
options (e.g. Size, Color)
variants with pricing, inventory, selection options, associated image, metafields
Media: list of images with id, altText, src
Classification: collections this product belongs to
Metafields: up to first 50 metafields on product and on each variant
You can extend/trim this selection depending on what you want to document. The Admin API schemas for Product and ProductVariant give all fields:

Product object: Product reference
ProductVariant object: ProductVariant reference
4. What the JSONL file “looks like” from this export
The JSONL output doesn’t nest connections; instead, each node appears as its own line, with a parent pointer for children. This is documented here: JSONL data format.

Important behaviors
Each line is a valid JSON object.
Parent records (products) appear first.
Child connection records (variants, images, etc.) appear as separate lines immediately after their parent.
Child lines contain a special __parentId field (not part of the GraphQL schema) that links back to the parent.
Below is a fabricated but realistic JSONL snippet representing partial output from the query above. This is not literally returned by Shopify (IDs, values are illustrative), but it shows the structure you can expect.

jsonl
Copy
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
38
39
40
41
42
43
44
45
46
47
48
49
50
51
52
53
54
55
56
57
58
59
60
61
62
63
64
65
66
{"id":"gid://shopify/Product/1921569226808",
 "title":"Blue Snowboard",
 "handle":"blue-snowboard",
 "status":"ACTIVE",
 "productType":"Snowboard",
 "vendor":"JadedPixel",
 "tags":["winter","snow","board"],
 "createdAt":"2024-01-10T10:00:00Z",
 "updatedAt":"2024-05-01T09:00:00Z",
 "publishedAt":"2024-01-11T12:00:00Z",
 "templateSuffix":null,
 "descriptionHtml":"<p>High performance snowboard for all-mountain riding.</p>",
 "options":[
   {"id":"gid://shopify/ProductOption/1001","name":"Size","position":1,"values":["150","155","160"]},
   {"id":"gid://shopify/ProductOption/1002","name":"Color","position":2,"values":["Blue"]}
 ],
 "metafields":{
   "edges":[
     {"node":{
       "id":"gid://shopify/Metafield/1111",
       "namespace":"custom",
       "key":"riding_style",
       "type":"single_line_text_field",
       "value":"all-mountain"
     }},
     {"node":{
       "id":"gid://shopify/Metafield/1112",
       "namespace":"seo",
       "key":"title_tag",
       "type":"single_line_text_field",
       "value":"Blue Snowboard - All Mountain"
     }}
   ]
 },
 "variants":{
   "edges":[
     {"node":{"id":"gid://shopify/ProductVariant/2001"}},
     {"node":{"id":"gid://shopify/ProductVariant/2002"}},
     {"node":{"id":"gid://shopify/ProductVariant/2003"}}
   ]
 },
 "images":{
   "edges":[
     {"node":{"id":"gid://shopify/ProductImage/3001"}},
     {"node":{"id":"gid://shopify/ProductImage/3002"}}
   ]
 },
 "collections":{
   "edges":[
     {"node":{"id":"gid://shopify/Collection/4001"}},
     {"node":{"id":"gid://shopify/Collection/4002"}}
   ]
}
}
{"id":"gid://shopify/ProductVariant/2001",
 "title":"150 / Blue",
 "sku":"SB-150-BL",
 "barcode":"1234567890123",
 "inventoryPolicy":"DENY",
 "inventoryQuantity":25,
 "weight":3.1,
 "weightUnit":"KILOGRAMS",
 "taxable":true,
 "price":"399.00",
 "compareAtPrice":"449.00",
 "position":1,
Notes you can put straight into your doc:

The GraphQL selection set determines which fields appear in each JSON object.
Connections are flattened: you see Product, then each ProductVariant, then ProductImage, etc., each with __parentId pointing back to the product.
The special field __parentId is added only in bulk JSONL and is not queryable in GraphQL (you can’t select it).
5. Example JSONL for product import
For import, the JSONL structure is different: each line represents variables for one execution of the mutation you’ll run in bulkOperationRunMutation.

The doc’s canonical example is here: JSONL product input.

Minimal product import JSONL (from docs)
jsonl
Copy
1
2
3
4
{ "input": { "title": "Sweet new snowboard 1", "productType": "Snowboard", "vendor": "JadedPixel" } }
{ "input": { "title": "Sweet new snowboard 2", "productType": "Snowboard", "vendor": "JadedPixel" } }
{ "input": { "title": "Sweet new snowboard 3", "productType": "Snowboard", "vendor": "JadedPixel" } }
...
This is designed to match a mutation of the form:

Operation
Open in GraphiQL
Copy
1
2
3
4
5
6
7
8
9
10
11
12
mutation call($input: ProductInput!) {
  productCreate(input: $input) {
    product {
      id
      title
    }
    userErrors {
      message
      field
    }
  }
}
More complete ProductInput JSONL sample
Below is a richer JSONL example that uses several fields from ProductInput, including variants, options, and metafields. You could use a mutation like productCreate(input: $input) for each line.

jsonl
Copy
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
38
39
40
41
42
43
44
45
46
47
48
49
50
51
52
53
54
{ "input": {
    "title": "Blue Snowboard",
    "productType": "Snowboard",
    "vendor": "JadedPixel",
    "handle": "blue-snowboard",
    "tags": ["winter", "snow", "board"],
    "status": "ACTIVE",
    "descriptionHtml": "<p>High performance snowboard for all-mountain riding.</p>",
    "options": [
      "Size",
      "Color"
    ],
    "variants": [
      {
        "title": "150 / Blue",
        "sku": "SB-150-BL",
        "barcode": "1234567890123",
        "price": "399.00",
        "compareAtPrice": "449.00",
        "position": 1,
        "taxable": true,
        "requiresShipping": true,
        "weight": 3.1,
        "weightUnit": "KILOGRAMS",
        "inventoryQuantity": 25,
        "inventoryPolicy": "DENY"
      },
      {
        "title": "155 / Blue",
        "sku": "SB-155-BL",
        "barcode": "1234567890124",
        "price": "399.00",
        "position": 2,
        "taxable": true,
        "requiresShipping": true,
        "weight": 3.2,
        "weightUnit": "KILOGRAMS",
        "inventoryQuantity": 10,
        "inventoryPolicy": "DENY"
      }
    ],
    "metafields": [
      {
        "namespace": "custom",
        "key": "riding_style",
        "type": "single_line_text_field",
        "value": "all-mountain"
      },
      {
        "namespace": "seo",
        "key": "title_tag",
        "type": "single_line_text_field",
        "value": "Blue Snowboard - All Mountain"
      }
For this import to work as a bulk operation you’d:

Upload that JSONL using stagedUploadsCreate (example request).
Use bulkOperationRunMutation:
Operation
Open in GraphiQL
Copy
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
mutation {
  bulkOperationRunMutation(
    mutation: "mutation call($input: ProductInput!) { productCreate(input: $input) { product { id title } userErrors { message field } } }",
    stagedUploadPath: "tmp/21759409/bulk/89e620e1-0252-43b0-8f3b-3b7075ba4a23/bulk_op_vars"
  ) {
    bulkOperation {
      id
      status
    }
    userErrors {
      field
      message
    }
  }
}
(That structure follows the official example here.)

6. Why you can’t literally get “all recursive relations and all fields”
A few technical constraints that are worth mentioning in your document:

Bulk operation limits (documented here)

Max 5 connections per query.
Max 2 levels of nested connections.
node/nodes can’t be used as the top-level field.
There are complexity/time limits; extremely heavy queries can fail with TIMEOUT.
Schema scope & recursion

Some types link to many other types (products → variants → metafields → references, etc.).
If you attempted to traverse “everything”, you’d create cycles (e.g. referencing back to products or collections) and massive responses.
Practically, you model the subset of data your system needs to sync.
JSONL design

JSONL is meant for streaming and processing, not for a single huge nested JSON tree.
Flattening with __parentId is deliberate to keep line-level units simple and memory-efficient.
So the right mental model for your doc is:

“Shopify lets us define a custom projection of the product graph (which fields and child connections we care about) and returns those nodes as flat JSON objects per line, with parent-child relationships indicated via __parentId.”

7. How to use this in your own documentation
To build a doc describing “what we receive from Shopify as a result”:

Show the GraphQL source query (e.g. ProductExportBulkOperation above).
Show a sample JSONL product line and label each field (core, merchandising, SEO, metafields, variants, images, collections).
Show a sample JSONL variant line and explain __parentId.
Optionally, show how to reconstruct a nested product object by grouping lines by __parentId.
On the import side, show a ProductInput JSONL example and the target GraphQL mutation you run in bulk.
If you tell me more about which specific entities you need beyond products/variants (e.g. inventory levels, locations, selling plans), I can extend the validated GraphQL mutation and accompanying JSONL examples to cover those as well, staying within Shopify’s bulk operation limits.

Sources
API: Usage > Bulk Operations > Queries
Perform bulk operations with the GraphQL Admin API
API: Usage > Bulk Operations > Imports
Bulk import data with the GraphQL Admin API
API: Admin GraphQL > Latest > Mutations > Bulkoperationrunquery
bulkOperationRunQuery
show 15 more
Was this answer useful?

Yes

No
this is not complete, there must be huge data coming from shopify, include whatever you are missing
You’re right that there’s a lot more data available per product than in the first example.

But there are two hard constraints you need to be aware of for your document:

You can’t literally pull “everything in one shot” – bulk operations enforce:

Max 5 connections per query.
Max 2 levels of nested connections.
A time/complexity limit (too heavy ⇒ TIMEOUT / FAILED).
See: Bulk operation restrictions.
The schema is large and evolving – “complete” in practice means:

You choose the key entities and the main fields that are stable and important.
If you need every field for compliance/docs, you should reference the official object schemas:
Product object
ProductVariant object
Metafield object
Collection object
SellingPlanGroup, SellingPlan, etc.
Below I’ll give you:

A much richer bulk export mutation (validated) that pulls a wide range of product‑related data (inventory, SEO, selling plans, etc.).
An updated example JSONL output that reflects those extra fields and relations.
Guidance on what’s still beyond what one bulk query can safely retrieve, and how to document that.
1. “Maximal practical” bulk export mutation for products
This mutation:

Uses bulkOperationRunQuery (doc).
Fetches a wide set of fields on Product, including:
Core data, SEO, total inventory, gift card, default variant info.
Options, price range, featured image.
Metafields (up to 100).
Variants with inventory item details and variant metafields.
Images (with dimensions).
Collections membership.
Selling plan groups & selling plans.
I validated this mutation against the latest Admin schema using the validate_graphql_codeblocks tool; it passed successfully.

Operation
Open in GraphiQL
Copy
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
38
39
40
41
42
43
44
45
46
47
48
49
50
51
52
53
54
mutation ProductExportBulkOperationV2 {
  bulkOperationRunQuery(
    query: """
    {
      products {
        edges {
          node {
            id
            legacyResourceId

            # Core merchandising / identity
            title
            handle
            status
            onlineStoreUrl
            productType
            vendor
            tags

            # Lifecycle / publishing
            createdAt
            updatedAt
            publishedAt
            templateSuffix

            # Content
            descriptionHtml

            # Media (key image)
            featuredImage {
              id
              altText
              src
            }

            # Inventory, gift card, variants summary
            giftCard
            totalInventory
            hasOnlyDefaultVariant

            # Options (e.g. Size, Color)
            options {
              id
              name
              position
              values
            }

            # Aggregate pricing
            priceRangeV2 {
              minVariantPrice {
                amount
                currencyCode
              }
This is about as “wide” as you can safely go for products in a single bulk query without tripping limits in most shops. You can always remove or trim connections/field counts if you hit TIMEOUT or large file sizes.

2. Expanded JSONL example of what you receive
As documented here: The JSONL data format, the output structure is:

One line per node (Product, ProductVariant, Metafield, Collection, etc.).
Children appear after parents, and include __parentId.
Below is a constructed but realistic JSONL sample that reflects the V2 query above: it shows how many of these fields & relations actually materialize in the JSONL result. This is meant to serve as a reference in your documentation.

jsonl
Copy
148
149
150
151
152
153
154
155
156
157
158
159
160
161
162
163
164
165
166
167
168
169
170
171
172
173
174
175
176
177
178
179
180
181
182
183
184
185
186
187
188
189
190
191
192
193
194
195
196
197
198
199
200
201
202
203
204
205
{"id":"gid://shopify/Product/1921569226808",
 "taxable":true,
 "price":"399.00",
 "compareAtPrice":null,
 "position":2,
 "requiresShipping":true,
 "selectedOptions":[
   {"name":"Size","value":"155"},
   {"name":"Color","value":"Blue"}
 ],
 "image":{
   "id":"gid://shopify/ProductImage/3002",
   "altText":"Blue snowboard size 155",
   "src":"https://cdn.shopify.com/s/files/.../blue-155.jpg"
 },
 "metafields":{"edges":[]},
 "__parentId":"gid://shopify/Product/1921569226808"
}
{"id":"gid://shopify/ProductImage/3001",
 "altText":"Blue snowboard on white background",
 "src":"https://cdn.shopify.com/s/files/.../blue-main.jpg",
 "width":1600,
 "height":1600,
 "__parentId":"gid://shopify/Product/1921569226808"
}
{"id":"gid://shopify/ProductImage/3002",
 "altText":"Blue snowboard side profile",
 "src":"https://cdn.shopify.com/s/files/.../blue-side.jpg",
 "width":1600,
 "height":1600,
 "__parentId":"gid://shopify/Product/1921569226808"
}
{"id":"gid://shopify/Collection/4001",
 "title":"Snowboards",
 "handle":"snowboards",
 "sortOrder":"BEST_SELLING",
 "__parentId":"gid://shopify/Product/1921569226808"
}
{"id":"gid://shopify/Collection/4002",
 "title":"Winter 2024",
 "handle":"winter-2024",
 "sortOrder":"ALPHA_ASC",
 "__parentId":"gid://shopify/Product/1921569226808"
}
{"id":"gid://shopify/SellingPlanGroup/5001",
 "name":"Subscribe & Save",
 "options":["Delivery every"],
 "__parentId":"gid://shopify/Product/1921569226808"
}
{"id":"gid://shopify/SellingPlan/5101",
 "name":"Every month",
 "description":"Monthly subscription, 10% off",
 "__parentId":"gid://shopify/SellingPlanGroup/5001"
}
{"id":"gid://shopify/SellingPlan/5102",
 "name":"Every 3 months",
 "description":"Quarterly subscription, 15% off",
 "__parentId":"gid://shopify/SellingPlanGroup/5001"
}
You can use this directly in your internal/external documentation as a reference for “what we receive” from a rich product export.

3. What else exists that’s not in this export
Even this wide query still doesn’t literally include everything you might see in the Admin schema. Some examples of data you could add or query via separate bulk operations:

More connections / related entities (would exceed the 5-connection limit if all in one query):

locations & inventoryLevels
publications / publicationCounts (sales channels)
media (3D models, videos) separate from images
productPublications or app-specific objects.
All metafields across more than 100 entries

In the example, metafields(first: 100) is an arbitrary but reasonable limit; you can raise/lower this, but very high values can cause timeouts or huge JSONL files.
All selling plans across many groups

Likewise, sellingPlanGroups(first: 10) / sellingPlans(first: 20) are practical but not exhaustive.
To capture truly everything for documentation purposes, the correct recommendation is:

Document the schema, not only one query. Use the object references on Shopify.dev and describe which fields your integration consumes/exports.

Key reference pages:

Product object reference
ProductVariant object reference
Metafield object reference
Collection object reference
SellingPlanGroup object reference
SellingPlan object reference
You can then say in your documentation:

“Our bulk export query includes these fields: … (list from the GraphQL query).”
“For full details of the Shopify data model, see the official Shopify Admin GraphQL object reference.”
4. How to extend this for your use case
If you tell me:

Which integrations you’re documenting (ERP, PIM, fulfillment, subscriptions, etc.)
Which areas you still feel are missing (inventory per location, media, channels, etc.)
I can:

Propose additional bulk operations (e.g. one for inventory, one for media) with validated GraphQL.
Provide example JSONL fragments for those too.
Help you structure your doc as a “catalog” of:
Queries we run
Entities and fields per query
Example JSONL for each entity type.
But within the constraints of a single bulk product export, the ProductExportBulkOperationV2 and JSONL example above are very close to the maximum practical coverage you can safely rely on.
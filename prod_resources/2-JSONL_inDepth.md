# follow up: You get JSONL where each line is a JSON object (product, variant, inventory item, etc.). // will each line contain different types of info? can you tell me the format of jsonl?

For Shopify Bulk Operations, yes:

Each line in the JSONL file is one JSON object.
Those objects can be of different types (product, variant, inventory item, metafield, etc.), depending on what you requested in your bulk GraphQL query.
Shopify adds special fields (like __typename, __parentId) so you can tell what each line is and how they relate.
The official guide is here: Perform bulk operations with the GraphQL Admin API.

I’ll walk through:

Whether each line can contain different types
The general structure/format of Shopify Bulk JSONL
A concrete example: product + variants + inventory in JSONL
How you’d reconstruct full objects for your OMS
1. Does each line contain different types of info?
Yes, potentially.

It depends entirely on what your bulk query asks for.

If your bulk query only selects products at the top level, with only scalar fields (e.g., id, title), then every line will essentially be a product record (same shape).
If your bulk query selects nested connections, such as:
products → variants

variants → inventoryItem → inventoryLevels
…then the JSONL output will contain multiple entity types:

Lines for Product

Lines for ProductVariant

Lines for InventoryItem

Lines for InventoryLevel

etc.

Shopify flattens the nested GraphQL result into a stream of nodes. Each node becomes a JSON object on its own line, and Shopify uses link fields to preserve relationships between them.

So in your OMS context, a single JSONL file might have:

Lines representing products
Lines representing variants
Lines representing inventory levels
Possibly metafields or others, all interleaved.
2. The general format of Shopify Bulk JSONL
The key properties to know:

Content type: JSON Lines (JSONL)
One valid JSON object per line.
No outer array or root object.
Common fields you will see:
__typename – GraphQL type name (Product, ProductVariant, InventoryItem, InventoryLevel, etc.).
id – the node’s own ID (usually the GraphQL GID).
__parentId – ID of the parent node in the query tree (for nested nodes).
The fields you actually selected in your bulk query.
There is no single “universal JSON schema” – the content is determined by your GraphQL query. But the pattern is always:

text
Copy
1
2
3
4
<JSON object for node 1>\n
<JSON object for node 2>\n
<JSON object for node 3>\n
...
Each object typically looks like:

json
Copy
1
2
3
4
5
6
7
8
{
  "__typename": "SomeType",
  "id": "gid://shopify/SomeType/123",
  "__parentId": "gid://shopify/ParentType/456",
  "field1": "...",
  "field2": 123,
  ...
}
For top-level items (like the products edge nodes you requested), you often won’t see __parentId, because they are root nodes for your selection.

3. Example: Bulk query and corresponding JSONL
3.1 Example bulk query
Imagine you run a bulkOperationRunQuery like this (simplified):

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
mutation {
  bulkOperationRunQuery(
    query: """
    {
      products(first: 1000) {
        edges {
          node {
            id
            title
            productType
            variants(first: 10) {
              edges {
                node {
                  id
                  sku
                  price
                  inventoryItem {
                    id
                    inventoryLevels(first: 5) {
                      edges {
                        node {
                          id
                          available
                          location {
                            id
                            name
                          }
                        }
                      }
                    }
                  }
                }
              }
            }
          }
        }
      }
    }
    """
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
You then poll currentBulkOperation until status = COMPLETED and get a url to the output file.

3.2 What the JSONL might look like
Very simplified example (not full real output, but representative):

text
Copy
1
2
3
4
5
6
7
8
{"__typename":"Product","id":"gid://shopify/Product/632910392","title":"IPod Nano - 8GB","productType":"Cult Products"}
{"__typename":"ProductVariant","id":"gid://shopify/ProductVariant/808950810","sku":"IPOD2008PINK","price":"199.00","__parentId":"gid://shopify/Product/632910392"}
{"__typename":"InventoryItem","id":"gid://shopify/InventoryItem/341629","__parentId":"gid://shopify/ProductVariant/808950810"}
{"__typename":"InventoryLevel","id":"gid://shopify/InventoryLevel/1","available":10,"location":{"id":"gid://shopify/Location/111","name":"Main Warehouse"},"__parentId":"gid://shopify/InventoryItem/341629"}
{"__typename":"ProductVariant","id":"gid://shopify/ProductVariant/49148385","sku":"IPOD2008RED","price":"199.00","__parentId":"gid://shopify/Product/632910392"}
{"__typename":"InventoryItem","id":"gid://shopify/InventoryItem/49148385","__parentId":"gid://shopify/ProductVariant/49148385"}
{"__typename":"InventoryLevel","id":"gid://shopify/InventoryLevel/2","available":20,"location":{"id":"gid://shopify/Location/111","name":"Main Warehouse"},"__parentId":"gid://shopify/InventoryItem/49148385"}
...
Observations:

Some lines have __typename = "Product" – these are top-level products.
Some lines have __typename = "ProductVariant" – these are variants.
Some lines have __typename = "InventoryItem" – these represent the inventory item.
Some lines have __typename = "InventoryLevel" – these are per-location stock entries.
__parentId ties child lines to their parent:
Variant’s __parentId = product’s id.
InventoryItem’s __parentId = variant’s id.
InventoryLevel’s __parentId = inventoryItem’s id.
So yes: each line can be a different entity type, but the linkage is explicit.

## 4. How to interpret and reconstruct from JSONL
Your OMS (or data pipeline) generally does this:

4.1 Step 1: Parse JSONL
Read the file line by line.
Parse each line as JSON.
Get an array/stream of nodes:
json
Copy
1
2
3
4
5
6
7
[
  { "__typename": "Product", "id": "gid://shopify/Product/632910392", ... },
  { "__typename": "ProductVariant", "id": "...", "__parentId": "gid://shopify/Product/632910392", ... },
  { "__typename": "InventoryItem", "id": "...", "__parentId": "gid://shopify/ProductVariant/..." },
  { "__typename": "InventoryLevel", "id": "...", "__parentId": "gid://shopify/InventoryItem/..." },
  ...
]
4.2 Step 2: Group and rebuild structure
Typical reconstruction:

Create a map for products: productsById[id] = productObject.
For each node:
If __typename == "Product":
Initialize product.variants = [].
If __typename == "ProductVariant":
Find parent product by __parentId, attach variant.
Initialize variant.inventory = [] (or similar).
If __typename == "InventoryItem":
Attach inventory item details to its parent variant.
If __typename == "InventoryLevel":
Attach an entry (location + available) under the right variant/inventoryItem.
End result in your OMS shape might look like:

json
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
[
  {
    "id": "gid://shopify/Product/632910392",
    "title": "IPod Nano - 8GB",
    "productType": "Cult Products",
    "variants": [
      {
        "id": "gid://shopify/ProductVariant/808950810",
        "sku": "IPOD2008PINK",
        "price": "199.00",
        "inventory": [
          {
            "locationId": "gid://shopify/Location/111",
            "locationName": "Main Warehouse",
            "available": 10
          }
        ]
      },
      ...
    ]
  }
]
This is what you feed into your OMS as its product/SKU structure.

5. Typical format patterns you’ll see
While the exact fields depend on your bulk GraphQL query, these patterns are very common:

Top-level items:

Have __typename and no __parentId.
Contain the scalar fields you selected.
Nested connections:

Appear as separate lines with:
__typename corresponding to the nested type.
__parentId pointing to their parent node.
IDs:

Always GraphQL global IDs (gid://shopify/...).
Your OMS might store them as strings or decode to numeric IDs if needed.
Scalars vs nested objects:

If your selection includes nested objects (like location { id name }), those nested objects appear inside the JSON for that line, as shown above:
json
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
    {
      "__typename": "InventoryLevel",
      "id": "...",
      "available": 10,
      "location": {
        "id": "gid://shopify/Location/111",
        "name": "Main Warehouse"
      },
      "__parentId": "gid://shopify/InventoryItem/341629"
    }
6. So, what is the “format” of JSONL in one sentence?
For Shopify Bulk Operations:

The JSONL file is a text file where each line is a JSON object representing one node from your GraphQL query (product, variant, inventory item, etc.), including the fields you selected plus Shopify-added fields like __typename and __parentId to indicate type and parent-child relationships.

7. If you want, next step
If you share either:

The Bulk GraphQL query you’re running (or plan to run), or
A few sample lines from your actual JSONL (anonymized),
I can:

Identify exactly which entity types appear (Product, ProductVariant, etc.),
Show a concrete reconstruction algorithm tailored to your query,
And suggest a clean OMS JSON structure to map everything into.
Sources
API: Usage > Bulk Operations > Queries
Perform bulk operations with the GraphQL Admin API
Was this answer useful?

Yes

No
# why flattened structure, why not the jsonl in json or edges-node way? 
The flattened JSONL format in Shopify Bulk Operations is very intentional. It’s chosen instead of a big nested JSON “edges/node” tree for a few practical reasons:

Performance and scale (huge datasets)
Streaming and memory usage
Compatibility with big‑data tools and message queues
Simpler, generic structure independent of any one query’s nesting
The design is described conceptually in Perform bulk operations with the GraphQL Admin API, but Shopify doesn’t spell out every design choice there, so I’ll connect the dots.

1. What you’re asking: “Why flattened JSONL instead of nested JSON?”
Normal GraphQL responses are nested:

json
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
{
  "data": {
    "products": {
      "edges": [
        {
          "node": {
            "id": "Product1",
            "variants": {
              "edges": [
                { "node": { "id": "Variant1.1" } },
                { "node": { "id": "Variant1.2" } }
              ]
            }
          }
        },
        {
          "node": {
            "id": "Product2",
            ...
          }
        }
      ]
    }
  }
}
For small pages, this is fine.

For a bulk export of hundreds of thousands or millions of nodes, that structure has problems:

The entire nested tree is one giant JSON document.
You can’t start processing until you’ve downloaded/parsing the whole thing.
It’s very memory‑heavy and awkward to process in streaming/batch systems.
So Shopify’s Bulk Operations change only the output representation, not the logical graph:

Instead of one big nested JSON, they give you one node per line (JSONL).
They store relationships via __parentId, __typename, etc., not via nested edges.
2. Why flattened JSONL is better for bulk / big data
2.1 Streaming & low memory usage
With JSONL:

text
Copy
1
2
3
4
5
{"__typename":"Product", "id":"P1", ...}
{"__typename":"ProductVariant", "id":"V1.1", "__parentId":"P1", ...}
{"__typename":"ProductVariant", "id":"V1.2", "__parentId":"P1", ...}
{"__typename":"Product", "id":"P2", ...}
...
You can:

Start processing line by line as soon as download starts.
Never hold the entire dataset in memory.
Pipe it straight into:
A message queue (Kafka, SQS, Pub/Sub),
A streaming processor (Flink, Spark streaming),
A data lake file on S3/GCS,
Or directly into an OMS import routine.
If Shopify returned a massive nested JSON “edges/node” tree:

You’d need to download and parse the entire file just to start iterating safely.
Many JSON parsers need the whole document loaded or at least parsed as a whole tree.
It becomes much heavier on memory and error‑prone on huge datasets.
JSONL’s flat structure fits extremely well with bulk / big‑data practices.

2.2 Independence from query nesting and schema evolution
Your bulk query might be nested like:

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
products {
  edges {
    node {
      id
      variants { edges { node { ... } } }
    }
  }
}
Or much more complex (products → variants → inventoryItem → inventoryLevels → location).

If Shopify tried to preserve that exact “edges-node” tree in bulk:

Every bulk output format would depend heavily on how you wrote the query.
Minor changes to query nesting or structure could break downstream consumers.
The JSON tree could be arbitrarily deep and irregular, complicating streaming processing.
By flattening:

Each node (product, variant, inventory item, etc.) has the same, simple shape:
__typename
id
__parentId (if nested in the original query)
The fields you asked for on that type
The bulk format becomes generic:
Tools know: “It’s just nodes with type + ID + optional parentId.”
They don’t need to know your original GraphQL nesting (edges / nodes / connections).
That makes the output much more stable and easier for generic ingestion tools.

2.3 Interoperability with external systems (OMS, warehouses, data platforms)
Modern data/OMS architectures often involve:

Message queues (Kafka, SQS, Pub/Sub)
Data lakes (S3, GCS, Azure Blob)
ETL / ELT tools (Spark, Flink, dbt, etc.)
These systems almost universally love formats like:

JSON Lines,
Parquet/ORC/Avro,
Or CSV.
They do not love:

Giant irregular nested JSON trees with GraphQL-style edges & nodes.
With JSONL, you can:

Pipe the file into your tool of choice:
Many tools can read JSONL natively:
spark.read.json("s3://.../bulk-products.jsonl")
jq, awk, line-based streaming.
Post-process it into tables:
Products table (filter __typename == "Product"),
Variants table (filter __typename == "ProductVariant"),
Inventory table (filter __typename == "InventoryLevel"), etc.
If Shopify gave you nested edges/nodes:

You’d first have to flatten it yourself into something like JSONL anyway.
Shopify just saves everyone from re‑implementing this flattening step.
2.4 Robustness and partial processing
With JSONL:

If your processing fails halfway through, you still have:
A prefix of valid, independently parseable lines (each is complete JSON).
You can:
Resume from the last processed line number,
Or rerun from a checkpoint without reprocessing the entire file.
With a huge nested JSON:

One small syntax error or truncation at the end can make the entire document invalid.
Harder to do “resume from X” patterns without custom state.
JSONL’s flat, record‑per‑line design makes it easier to implement robust, restartable batch/stream processors.

## 3. Conceptual tradeoff: edges/nodes vs JSONL
Edges/nodes (normal GraphQL response style)
Pros:

Very natural for interactive, real-time API calls:
You ask for “products + variants” and get a tree that matches your query shape.
Easy for frontend/backend code to traverse with object/array access.
Cons for bulk:

Heavy, nested, and large.
Harder to stream and work with in big-data tools.
Query-shape–dependent; changes to query can break downstream reading code.
Flattened JSONL (Bulk Operations output)
Pros:

Streamable, line-based – process as you read.
Predictable, generic layout:
Node per line, type + id + parentId.
Plays very well with analytics, queues, OMS imports, warehouses.
Better for very large datasets.
Cons:

You have to manually reconstruct the tree or mapping if you want nested objects (e.g., orders with line items).
Slightly more work at the consumer side, but:
That’s work you’d have to do anyway for serious ETL/OMS mapping,
And it scales much better.
Given Bulk Operations are explicitly for massive data volumes, Shopify optimizes for scalability and interoperability, not for “looks like a normal GraphQL response”.

## 4. How this affects you as an OMS integrator
In practice, for product sync:

You design a bulk query (products + variants + inventory).

Bulk Ops returns a flattened JSONL stream of those nodes.

Your integration / OMS:

Reads each line,
Uses __typename and __parentId to classify and link:
Product lines → OMS product records
Variant lines → OMS SKUs belonging to products
InventoryLevel lines → stock per location per SKU
Builds your ideal nested or relational structure in your own database.
It’s more work than simply iterating a JSON edges/node tree, but it’s:

More scalable,
Easier to integrate with modern infra,
And you gain full control of how to map that into your OMS model.
5. Quick mental summary
Why flattened?
Bulk Ops are for big data – JSONL + flat nodes is the de-facto industry pattern for scalable ETL/streaming.
Why not nested edges/nodes?
That structure is great for interactive API calls, but awful for giant, asynchronous exports.
Flattening makes streaming, storage, and later transformation much easier.
How do we keep relationships without nesting?
__typename tells you what type each line is.
__parentId (and IDs) tell you “this variant belongs to this product”, etc.
If you’d like, I can:

Take a specific bulk query (e.g., for products + variants) and:
Show the exact GraphQL you’d use,
Show an example of the JSONL lines you’d see,
And write some pseudo-code or real code to reconstruct per-product data for your OMS.
Sources
API: Usage > Bulk Operations > Queries
Perform bulk operations with the GraphQL Admin API
Was this answer useful?

Yes

No
# Was Shopify using this flattened structure earlier too?  
now flattened structure - how is it processed? there is typename, parentId... so whats next? how it is processed line by line as you said, does it take all products first, then its variants, what if all products are not downloaded? 
Good set of questions. I’ll break it into parts:

Was Shopify always using this flattened JSONL structure?
Once you have __typename and __parentId, what’s the actual processing “next step”?
How is it processed line by line in practice (ordering, parents vs children)?
What if the client doesn’t download the whole file (partial download / failure)?
Reference again: Bulk operations with the GraphQL Admin API and Bulk queries guide.

## 1. Was Shopify using this flattened JSONL structure earlier too?
Publicly, the Bulk Operations API has always been JSONL + flattened since it was introduced.

Bulk Ops were designed from the start as a big-data / ETL–friendly API, not a traditional synchronous API.
The JSONL + __typename + __parentId pattern is part of that design, and has been consistent across versions (though some details evolve).
In other words:

Normal Admin GraphQL (synchronous) → nested edges / nodes JSON.
Bulk Operations Admin GraphQL (async) → flattened JSONL, one node per line with type and parent identifiers.
There hasn’t been a published “bulk operations format” that was nested JSON and then later changed to JSONL; JSONL is the intended format for bulk from the beginning.

## 2. You have __typename and __parentId — what’s next?
Conceptually, the processing steps in your system are:

Read the JSONL file line by line.
For each line:
Parse the line into a JSON object.
Look at:
__typename → what type is this? (Product, ProductVariant, InventoryItem, etc.)
id → unique node id (gid://shopify/...).
__parentId (if present) → which node is its parent in the original query tree?
Store or stream this into some structure:
Temporary in-memory maps,
Or staging tables (e.g., bulk_products, bulk_variants, bulk_inventory_levels).
Rebuild relationships using __parentId:
Attach variants to their products.
Attach inventory items to variants.
Attach inventory levels to inventory items, etc.
Map into your OMS model:
Upsert OMS Product and SKU records with the fields you care about.
Optionally aggregate inventory per SKU/location.
So:

__typename → dispatcher (“which table/list do I put this in?”).
id / __parentId → join keys (“who is my parent?”).
The rest of the fields → the actual business data (title, sku, price, available, etc.).
## 3. Line-by-line processing details
3.1 Does Shopify output all products first, then variants, then inventory?
There is no documented guarantee that lines are grouped by type or perfectly ordered (e.g., all products first). However, in practice you typically see a pattern roughly matching the query’s traversal:

It tends to be emitted in a depth-first or breadth-first walk of your query’s result:
Product
Then its variants
Then nested inventory items / inventory levels
Then next product, etc.
But for robust integrations, you should not rely on a strict order. Instead:

Always be prepared for:
A child (variant) appearing before its parent (product) in the file.
Types interleaved in any order.
This is why the design includes __parentId:

Relationships are explicit via IDs, not via sequence.
3.2 How do you process when order is uncertain?
There are two main patterns:

Pattern A – Single-pass with maps in memory
Maintain dictionaries (hash maps) keyed by id:

productsById
variantsById
inventoryItemsById
etc.
For each line:

Parse node.
Switch on __typename:
If Product:

productsById[id] = {... product data ..., variants: [] }
If ProductVariant:

Optionally variantsById[id] = {... variant data ..., inventory: [] }
If __parentId product exists in productsById:
Attach variant immediately: productsById[parentId].variants.push(variant)
If parent not yet seen:
Store variant in variantsById, and later when product appears, attach.
Similar logic for InventoryItem / InventoryLevel.

End of file:
Sweep leftover nodes in variantsById, inventoryItemsById and attach to their parents now that all nodes are known.
This needs enough memory to hold at least a working set of products/variants until you finish or flush to your DB, but for many systems this is fine.

Pattern B – Multi-pass or staged tables
For very large datasets or warehouse-style processing:

First pass: Just write each node to a staging table:

e.g., in SQL or a columnar store:

bulk_nodes with columns:
id
parent_id
typename
payload (JSONB or decomposed columns)
Later, run SQL queries to reconstruct relationships:

Products table:
sql
Copy
1
2
3
     SELECT *
     FROM bulk_nodes
     WHERE typename = 'Product';
Variants table:
sql
Copy
1
2
3
4
5
6
     SELECT v.*, p.id AS product_id
     FROM bulk_nodes v
     JOIN bulk_nodes p
       ON v.parent_id = p.id
     WHERE v.typename = 'ProductVariant'
       AND p.typename = 'Product';
Inventory per variant:
sql
Copy
1
2
3
4
     SELECT il.*, i.id AS inventory_item_id, v.id AS variant_id
     FROM bulk_nodes il
     JOIN bulk_nodes i ON il.parent_id = i.id AND il.typename = 'InventoryLevel'
     JOIN bulk_nodes v ON i.parent_id = v.id AND v.typename = 'ProductVariant';
From those joined views, populate final OMS products, skus, inventory tables.
In this pattern, line order doesn’t matter at all. Everything is joined by IDs after the load.

## 4. What if not all products are downloaded? (partial client download/failure)
From Shopify’s perspective:

Bulk Operation runs server-side.
It completes (or fails) and only then gives you a URL to a complete output file.
That file is static at that point. Shopify isn’t streaming live into it while you read it.
So any “not all products downloaded” scenario means:

Your client (or network) stopped reading the file early.
Or you truncated the file on disk.
4.1 What happens if you stop midway?
A few scenarios:

You voluntarily stop mid-file (e.g., killed the process):

You’ve processed the first N lines, but not the rest.
Any nodes that appear later (some products, some variants, some inventory levels) are missed.
Consequence: your OMS will have incomplete data:
Some products without some variants,
Or some variants with no inventory, etc.
Network failure partway through download:

Similar: your local copy is truncated.
The tail of the file is missing; again, incomplete data.
In both cases, Shopify considers the export “COMPLETED”. From their side, the file is correct; it’s the client that hasn’t read all of it.

4.2 How to handle this robustly
General strategies:

Treat a Bulk download as all-or-nothing:

Download the entire file to disk/storage first.
Validate size/checksum if you have one (or at least that you reached EOF cleanly).
Only then begin processing.
If download fails:
Discard the partial file.
Retry the download from the same URL (if still valid) or rerun the bulk operation.
Streaming with checkpointing (advanced):

If you need true streaming and can’t store the whole file:
Keep a “line offset” checkpoint.
Store processed lines somewhere and be able to detect duplicates or reprocess safely.
If a failure occurs:
Re-download the whole file and skip already-processed lines by ID (idempotent upsert).
This requires idempotent writes to your OMS:
Upserting a product/variant/inventory line more than once does no harm.
Always tolerate “orphans” during processing:

Using the multi-pass or staging-table pattern, you:
Load all lines you received into a staging table.
If the file was truncated, some nodes will have parent_ids that don’t exist in the staging set.
You can detect that:
E.g., SELECT * FROM variants WHERE parent_id NOT IN (SELECT id FROM products) → these should be zero in a complete run.
If you see orphans, you know the load is incomplete; you can flag the batch as failed and re-run.
4.3 Does Shopify guarantee parent before child in the file?
Because the recommended way is ID-based linking, you should not rely on ordering guarantees. Even if empirically products tend to appear before their variants, your integration should be:

Independent of order,
Based on id/__parentId joins,
And resilient to processing nodes before or after their parents.

## 5. Summarizing the processing logic in simple steps
Putting it in a step-by-step flow:

Shopify Bulk query finishes → status = COMPLETED, url ready.
Your system:
Downloads the entire JSONL file (recommended).
Or streams and processes with idempotent writes and restart logic.
For each line in JSONL:
Parse JSON.
Inspect __typename:
If Product → store under productsById (or bulk_products table).
If ProductVariant → store under variantsById, and note __parentId.
If InventoryItem → store under inventoryItemsById, and note __parentId.
If InventoryLevel → store under inventoryLevelsById, and note __parentId.
etc.
After loading:
Rebuild relationships:
Join variants → products via __parentId.
Join inventoryItems → variants via __parentId.
Join inventoryLevels → inventoryItems via __parentId.
Finally:
Transform to your OMS schema for products/SKUs/inventory.
Upsert into your OMS.
## 6. If you want a concrete example with code
If you’d like, I can next:

Write a small Python or Node.js example that:
Reads a JSONL file from Shopify Bulk (assume some example lines),
Builds products with nested variants and inventory using __typename and __parentId,
Shows the final JSON structure you could push into your OMS.
Or, if you have a real JSONL sample (even 5–10 lines with sensitive IDs scrubbed), I can:

Annotate line by line what each node is,
Show exactly how to process those lines into full product entities for your system.
Sources
API: Usage > Bulk Operations
Bulk operations with the GraphQL Admin API
API: Usage > Bulk Operations > Queries
Bulk queries guide
Was this answer useful?


# claude ai resource:
https://claude.ai/chat/d241704b-7775-4ae9-9f0d-4beb0ac8a970

# Summary of questions:
Here’s a summary list of the questions you asked in this conversation (just the questions, in order, as you requested):

Data format & OMS connectivity

“Hello, can you tell me how Shopify sends the data to any order managing system software? Is it a JSONL?”
“I have heard that in Shopify and OMS connectivity we receive JSONL when a messageQueue is used, it brings data from Shopify as a Shopify order, that is JSONL which is coming from different Shopify entities that should be converted to JSON form after reaching any OMS to sync or map to its entities.”
Bulk operations and JSONL

“Do you know about bulk query from Shopify?”
“When JSONL is used?”
“But above you declined to this: ‘Shopify’s GraphQL Admin API Bulk Operations return data as JSONL.’ Why?”
---
“I am receiving data in JSONL form, what could be the reason for that?”
“Can you clarify a myth/reality, Shopify stores data in different entities. When I ask some data in GraphQL form, it provides data from different entities. So JSONL has data from different entities, is it?”
“How to convert JSONL to JSON?”
Meta / topic coverage

---

“What topics and their subtopics did we discuss here today? Also tell what things are remaining that I could have asked or you could have answered?”
“Can you prepare well documented notes for above knowledge so that I can copy it somewhere?”
“Can you brief me about callback URL’s?”
“Do you know the productType: 'MARKETING_PKG_AUTO'?”
“Can you provide product JSON which is used in OMS to sync products?”
“Provide summary of above discussion.”
Concept definitions and comparisons

“Define callback url.”
“Difference between callback url and web hool [webhook].”
Product sync and JSONL structure
---
“Can you tell me things related to product sync. All what you know?”
(After pasting the Product REST doc)
“Can you explain this one product – I am studying about: ‘3. Mechanisms for product sync’.”
“Follow up: You get JSONL where each line is a JSON object (product, variant, inventory item, etc.). // will each line contain different types of info? Can you tell me the format of JSONL?”
“Why flattened structure, why not the JSONL in JSON or edges-node way?”
“Was Shopify using this flattened structure earlier too?
Now flattened structure – how is it processed? There is typename, parentId… so what’s next? How it is processed line by line as you said, does it take all products first, then its variants, what if all products are not downloaded?”

---
---
## 1. What “product sync” usually means {#what-product-sync-usually-means}

## 2. Shopify data model you’re syncing {#shopify-data-model-youre-syncing}

### 2.1. Product {#product-entity}

### 2.2. ProductVariant {#productvariant-entity}

### 2.3. Inventory {#inventory-entity}

## 3. Mechanisms for product sync {#mechanisms-for-product-sync}

### 3.1. Pulling products via Admin REST API {#pulling-products-via-admin-rest-api}

### 3.2. Pulling products via Admin GraphQL API {#pulling-products-via-admin-graphql-api}

### 3.3. Bulk Operations for large catalogs {#bulk-operations-for-large-catalogs}

## 4. Keeping products in sync over time {#keeping-products-in-sync-over-time}

### 4.1. Initial full sync {#initial-full-sync}

### 4.2. Incremental sync / updates {#incremental-sync-updates}

### 4.3. Direction of truth {#direction-of-truth}

## 5. Handling JSONL for product sync (Bulk Operations) {#handling-jsonl-for-product-sync-bulk-operations}

## 6. Common fields to map into an OMS {#common-fields-to-map-into-an-oms}

### 6.1. Product-level fields {#product-level-fields}

### 6.2. Variant-level fields (per SKU) {#variant-level-fields-per-sku}

### 6.3. Inventory fields {#inventory-fields}

### 6.4. Metafields / attributes {#metafields-attributes}

## 7. Common product-sync challenges & patterns {#common-product-sync-challenges-patterns}

### 7.1. SKU uniqueness and conflicts {#sku-uniqueness-and-conflicts}

### 7.2. ProductType usage (`MARKETING_PKG_AUTO`, etc.) {#producttype-usage-marketing_pkg_auto-etc}

### 7.3. Large catalogs and performance {#large-catalogs-and-performance}

### 7.4. Error handling and idempotency {#error-handling-and-idempotency}

## 8. High-level product-sync architecture example {#high-level-product-sync-architecture-example}
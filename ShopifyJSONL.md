# Shopify → OMS Data Integration: 
# JSON vs JSONL, Bulk Operations, and Entities

1. Core Concepts and Definitions
1.1 JSON vs JSONL
JSON (JavaScript Object Notation)
A single JSON value (object or array), e.g.:
json

    {
      "orders": [
        { "id": 1, "name": "Order #1" },
        { "id": 2, "name": "Order #2" }
      ]
    }
Used in:

Shopify webhooks
Shopify REST Admin API
Shopify (non‑bulk) GraphQL Admin API responses
JSONL / JSON Lines / NDJSON

Text file/stream with one JSON object per line:

    {"id":1,"name":"Order #1"}
    {"id":2,"name":"Order #2"}
    {"id":3,"name":"Order #3"}
Convenient for:
Streaming / message queues
Large append-only logs
Batch processing (Spark, Flink, data lakes)
Shopify uses JSONL natively only in GraphQL Bulk Operations (see below).
2. How Shopify Sends Data to External Systems / OMS
2.1 Main integration mechanisms
Webhooks (push model)

Shopify sends HTTP POST requests to your endpoint on certain events.
Examples: orders/create, orders/updated, fulfillments/create, refunds/create, etc.
Payloads are single JSON objects, not JSONL.
Docs:
Webhooks overview:
https://shopify.dev/docs/apps/webhooks
Webhook resource & order topics:
https://shopify.dev/docs/api/admin-rest/latest/resources/webhook#event-topics-Order
Admin APIs (pull model)

Your app/OMS pulls data from Shopify:
REST Admin API: https://shopify.dev/docs/api/admin-rest
GraphQL Admin API: https://shopify.dev/docs/api/admin-graphql
Responses are JSON, typically:
REST: JSON objects/arrays (e.g. { "orders": [ ... ] })
GraphQL: { "data": { ... } } plus optional errors.
2.2 Data format in typical integrations
Normal webhooks → JSON
REST Admin API responses → JSON
Standard GraphQL Admin API (non-bulk) responses → JSON
Only Bulk Operations → JSONL (see Section 3)
So, in a typical Shopify → OMS integration:

Real-time or near real-time updates (per order, per event) are JSON.
Large, asynchronous exports (full syncs) via Bulk Operations use JSONL.
3. Shopify GraphQL Admin Bulk Operations and JSONL
3.1 Overview
Shopify’s Bulk Operations API (Admin GraphQL) is a special mechanism for high-volume, asynchronous reads and writes.

Key docs:

Bulk overview:
https://shopify.dev/docs/api/usage/bulk-operations
Bulk queries (bulk reads/exports):
https://shopify.dev/docs/api/usage/bulk-operations/queries
Bulk mutations (bulk writes/imports):
https://shopify.dev/docs/api/usage/bulk-operations/imports
3.2 What Bulk Operations are used for
Use Bulk Operations when you need to:

Export large datasets:
All orders
All customers
All products/variants
Import large batches of changes:
Bulk create/update products
Bulk update product metafields, etc.
Instead of manual pagination and client-side throttling, you:

Start a bulk operation (query or mutation).
Shopify runs it asynchronously.
You poll the operation status.
When complete, Shopify gives you a URL to a JSONL file with all results.
3.3 Key GraphQL operations
Start a bulk read (export):
bulkOperationRunQuery
Start a bulk write (import):
bulkOperationRunMutation
Check status of the current bulk operation:
currentBulkOperation
All of these are GraphQL Admin API mutations/queries.

3.4 Output format: JSONL
From Shopify docs (bulk queries / imports):

“When the operation is complete, the results are delivered in the form of a JSONL file that Shopify makes available at a URL.”

Characteristics of this JSONL:

One JSON object per line.
Each line corresponds to a node (entity) from your query or mutation result.
Lines may represent different entity types if your query includes multiple types (e.g. orders + line items + customers).
They often include special linkage fields, such as:
__typename – the GraphQL type name (Order, LineItem, Customer, etc.).
__parentId or __parentIds – referencing parent objects to reconstruct nested relationships.
Example (simplified):

{"__typename":"Order","id":"gid://shopify/Order/1","name":"#1001"}
{"__typename":"Customer","id":"gid://shopify/Customer/10","email":"a@example.com","__parentId":"gid://shopify/Order/1"}
{"__typename":"LineItem","id":"gid://shopify/LineItem/100","name":"Product A","__parentId":"gid://shopify/Order/1"}
{"__typename":"LineItem","id":"gid://shopify/LineItem/101","name":"Product B","__parentId":"gid://shopify/Order/1"}
This is native JSONL from Shopify, but only for Bulk Operations.

4. Shopify Entities and How They Appear in Responses
4.1 Entities in Shopify
Shopify models many different entities (object types), for example:

Order
Customer
Product
ProductVariant
LineItem
InventoryItem
Location
FulfillmentOrder
Fulfillment
etc.
These appear as different types in the Admin GraphQL API schema.

4.2 Non-bulk GraphQL responses
In a standard GraphQL query (non-bulk):

You can query multiple entities in one request.
The response is still one JSON object, with nested fields.
Example query (standard GraphQL):

Operation
Open in GraphiQL

query {
  order(id: "gid://shopify/Order/123") {
    id
    name
    customer {
      id
      email
    }
    lineItems(first: 5) {
      edges {
        node {
          id
          name
        }
      }
    }
  }
}
Example response (simplified):

JSON
{
  "data": {
    "order": {
      "id": "gid://shopify/Order/123",
      "name": "#1001",
      "customer": {
        "id": "gid://shopify/Customer/456",
        "email": "customer@example.com"
      },
      "lineItems": {
        "edges": [
          {
            "node": {
              "id": "gid://shopify/LineItem/1",
              "name": "Product A"
            }
          }
        ]
      }
    }
  }
}
Different “entities” are present (Order, Customer, LineItem), but they are nested within one JSON document.
This is not JSONL.
4.3 Bulk Operations JSONL and multiple entity types
When you run a bulk query that selects multiple entity types (e.g., orders with nested line items and customers), the JSONL output:

Contains one object per entity per line.
Includes multiple entity types in the same file—this is true.
Uses __parentId or similar fields to record the parent-child relationship.
Therefore:

It is correct to say that Bulk Operations JSONL can hold data from different Shopify entities.
It is incorrect to generalize that all GraphQL or all Shopify responses are JSONL; this only applies to Bulk Ops.
5. JSONL in Message Queues and Middleware
5.1 Where JSONL appears outside of Bulk Operations
If you see JSONL but you’re not directly using Bulk Operations, the JSONL is almost certainly introduced by:

A message queue / event bus (Kafka, SQS, Pub/Sub, RabbitMQ, etc.).
An integration platform (Mulesoft, Boomi, Workato, etc.).
A data sync/ETL tool (Fivetran, Airbyte, Stitch, custom scripts).
A data warehouse / analytics pipeline export.
Typical flows:

Webhooks → Integration → JSONL

Shopify webhooks (JSON) → Integration receives JSON → Integration emits JSONL to queues or files.
REST/GraphQL → Integration → JSONL

Integration pulls JSON from Shopify → writes/streams it as JSONL for easier processing.
Clues that the JSONL is not direct Shopify Bulk Ops:

Each object is wrapped with extra metadata:
source, tenant_id, event_type, received_at, payload, etc.
The inner payload contains the raw Shopify object.
File naming may reflect the integration (e.g., oms_events_2024-05-01.jsonl stored in S3).
In that case:

Shopify → JSON (webhooks/API)
Middleware → JSONL (for internal architecture)
6. Converting JSONL to JSON
6.1 Basic conversion: JSONL → JSON array
Goal: turn a .jsonl file into a standard JSON array of objects.

Conceptual algorithm:

Read the file line by line.
Ignore empty lines.
Parse each line as JSON.
Append each parsed object to a list/array.
Serialize the list/array as JSON.
Result example:

json

[
  {"id":"1","name":"Order #1","total":10.0},
  {"id":"2","name":"Order #2","total":15.5},
  {"id":"3","name":"Order #3","total":7.25}
]
6.2 Using tools (example: jq)
With jq (a common JSON CLI tool):


jq -s '.' data.jsonl > data.json
-s (“slurp”) turns all lines into a JSON array.
6.3 Shopify Bulk nuance: reconstructing nested entities
For Shopify Bulk Operations, you often want to:

First: Convert JSONL → array of objects (as above).
Then: Rebuild relationships using:
__typename (to know the entity type),
__parentId (to attach children to parents).
Typical reconstruction pattern for orders:

For each object:
If __typename == "Order", create an order entry.
If __typename == "LineItem", find its parent order by __parentId and append to that order’s lineItems.
If __typename == "Customer", assign it to the parent order via __parentId.
End goal: for each Order, produce a nested structure:

json

{
  "id": "gid://shopify/Order/1",
  "name": "#1001",
  "customer": { ... },
  "lineItems": [ { ... }, { ... } ]
}
This is the typical structure an OMS expects.

7. Typical OMS Sync Strategy with Shopify
7.1 Recommended pattern
Initial full sync (historical data)

Use Bulk Operations to export large volumes (e.g., all orders).
Process the JSONL:
Convert JSONL → objects.
Map into OMS entities.
Good for:
New OMS integrations.
Rebuilding states in data warehouses.
Incremental / ongoing sync

Use webhooks for real-time changes:
orders/create
orders/updated
fulfillments/create
refunds/create
Optionally use incremental bulk queries for:
Periodic “catch-ups” (e.g., orders updated in the last X hours) to avoid missed webhook events.
Error handling & idempotency

Ensure that:
Applying the same event twice does not corrupt data (idempotent operations).
Failed bulk operations are retried/resubmitted with smaller scope or reduced complexity.
8. Myth vs Reality Summary
Myth:

“Shopify sends data to OMS as JSONL by default.”

Reality:
Normal Shopify integrations (webhooks, REST, standard GraphQL) return JSON, not JSONL.
JSONL is used natively only in GraphQL Bulk Operations.
Myth:

“Whenever I query Shopify via GraphQL, I get JSONL with different entities.”

Reality:
Standard GraphQL responses are JSON, sometimes with many entities nested.
Bulk GraphQL operations produce JSONL where each line is an entity, and multiple entity types can appear in the same file.
Reality:

“A JSONL file from Shopify Bulk Operations can contain rows for different entities (e.g., orders, line items, customers).”

Correct. That’s due to:
How you write the bulk query (which entities you include).
Shopify flattening these into separate JSONL lines linked by __parentId/__typename.
Reality:

“Sometimes I receive JSONL because of a message queue / integration layer.”

Correct. Many architectures convert Shopify JSON (webhooks/API) to JSONL internally for stream or batch processing.
9. Questions You Could Explore Next
If you want to go deeper, good follow-up questions are:

Concrete Bulk Operations examples

“Show a full GraphQL bulk query to export all orders after a date, with specific fields.”
“Show code (Python/Node/etc.) to:
Trigger a bulk operation,
Poll status,
Download the JSONL,
Convert it into structured JSON orders for my OMS.”
OMS data modeling

“Which Shopify fields should I map to my OMS’s Order / LineItem / Customer entities?”
“How do Shopify Fulfillment and FulfillmentOrder map to OMS shipment/fulfillment concepts?”
Sync design

“How should I combine Bulk Ops and webhooks to maintain a consistent OMS sync?”
“What are recommended webhook topics and minimal field sets for an OMS integration?”
Bulk Operations limits and tuning

“What are the concurrency and time limits for Bulk Operations?”
“How do I design bulk queries that avoid timeouts for very large stores?”

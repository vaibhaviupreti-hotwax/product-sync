
## Q3 Prepare a documentation 
requirement is meta information
next step will be how to model it ? - for better data model - later to run a logic on it.

order validate -  chk fraud - adhar|ID|y - country wise personal identity
ProductCategoryAttribute
better model for the real data..

order approve 
c_id + id type(which id used) | unicpo | phone | ship to address ..|etc.. 

not a separate table | cannnot alter table | table size | normalized form

- Write example use cases of ProductCategoryAttribute table, it is not currently implemented but is included in the enterprise data model? 

Shopify          →  knows what products LOOK like
                    (images, descriptions, prices)

HotWax OMS       →  knows how products BEHAVE
                    (fulfillment, returns, tax, inventory)
---

1. Fulfillment Routing
attrName  = "fulfillment_facility"
attrValue = "WAREHOUSE_EAST"

attrName  = "ship_from_store_eligible"
attrValue = "Y"

2. Return & Refund Rules
attrName  = "returnable"
attrValue = "N"

attrName  = "return_window_days"
attrValue = "30"

3. Tax Calculation
attrName  = "tax_category"
attrValue = "CLOTHING_EXEMPT"

attrName  = "promotion_eligible"
attrValue = "Y"

4. Promotion & Discount Eligibility
attrName  = "max_discount_percent"
attrValue = "20"

attrName  = "reserve_immediately"
attrValue = "Y"

5. Inventory Reservation Rules
attrName  = "safety_stock_buffer"
attrValue = "5"

attrName  = "exclude_from_sync"
attrValue = "Y"

6. Shopify/External Sync Control
attrName  = "shopify_collection_id"
attrValue = "gid://shopify/Collection/123"

Customer places order for "Luxury Handbag" (CAT_LUXURY)
              ↓
OMS checks ProductCategoryAttribute for CAT_LUXURY
              ↓
returnable        = N   → no return option shown
tax_category      = LUXURY_TAX → apply 15% luxury tax
fulfillment_type  = SHIP_ONLY  → no store pickup allowed
reserve_immediate = Y          → lock inventory right away
promotion_eligible = N         → no coupons applicable
              ↓
Order processed with all these rules applied
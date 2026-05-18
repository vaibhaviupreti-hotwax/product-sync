# Question: There are “products” for services/jobs in OMS. is it, do oms have a product, means services ka data product entity me save hota hai kya?

Edited 3-seed-data

Yes, absolutely! **Data for services and jobs is saved in the exact same `Product` entity.** 

In HotWax/OFBiz (and most enterprise systems), a "Product" does not just mean a physical box on a shelf. A Product is **anything you can sell, price, or add to an order**.

### How does the system know it is a Service and not a Physical Item?
This is exactly where the **`productTypeId`** comes in (which we discussed earlier). 

When you save a service in the `Product` table, you change the type:

*   **Physical Item**: `productTypeId="FINISHED_GOOD"`
*   **Service/Job**: `productTypeId="SERVICE"`

### Why do we save Services in the Product Entity?
Because by saving a Service as a "Product", you can use all the existing commerce features for it:
1.  **Pricing**: You can give the service a price in the `ProductPrice` table (e.g., $50/hour for Consulting).
2.  **Ordering**: A customer can add the service to their cart, and it will be saved as an `OrderItem` alongside their physical items.
3.  **Categories**: You can group services into a `ProductCategory` (e.g., "Installation Services").
4.  **Taxes**: You can apply specific tax rules to it.

### The Key Difference in the OMS:
When the Order Management System processes an order, it looks at the `ProductType` of each line item.
*   If it sees a `FINISHED_GOOD`, it sends a request to the warehouse to **reserve inventory** and **create a shipping label**.
*   If it sees a `SERVICE`, the OMS **skips the warehouse steps completely**. It knows there is no physical box to ship, so it just marks that line item as "Completed" (or assigns it to a technician) without looking for inventory. 

**Example of a Service "Product" in XML:**
```xml
<Product productId="INSTALLATION_FEE" 
         productTypeId="SERVICE" 
         isVirtual="N" 
         isVariant="N" 
         productName="Standard Home Installation" 
         description="2 hours of technician labor for setup."/>
```
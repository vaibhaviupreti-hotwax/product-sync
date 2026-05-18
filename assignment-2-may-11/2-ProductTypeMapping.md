Edited VirtualVariant.md

Here is how the **`ProductType`** entity maps to real-world business scenarios. 

The `ProductType` dictates *how the system behaves*—whether it should track inventory, generate a shipping label, or trigger a manufacturing run.

### 1. Toy Manufacturing (Your Example)
In a factory that makes wooden toys, different stages of the product lifecycle require different types:
*   **`RAW_MATERIAL`**: The bulk blocks of pine wood, the buckets of red/blue paint, and the metal axles. (These occupy warehouse space but are *never* sold to a customer).
*   **`SUBASSEMBLY`**: The unpainted wooden toy car. (It has been manufactured from wood and axles, but it is not yet a final product).
*   **`FINISHED_GOOD`**: The "Red Wooden Toy Car" in its retail packaging. (This is tracked in inventory and sold on the website).

### 2. The Coffee Shop (Hybrid Business)
A modern coffee shop sells physical items, digital items, and labor.
*   **`RAW_MATERIAL`**: 50lb bags of coffee beans, gallons of milk, bottles of caramel syrup. (Used in production, not sold directly).
*   **`FINISHED_GOOD`**: A pre-packaged bag of chips or a branded coffee mug. (Bought from a supplier and sold as-is; requires inventory tracking).
*   **`SERVICE`**: "Barista Training Masterclass" ticket. (There is no physical inventory to ship; fulfillment means adding a name to a guest list).
*   **`ASSET_USAGE`**: Renting the back room of the cafe for a 2-hour private event. (Tracked by a calendar schedule, not by a quantity-on-hand).

### 3. Software & Electronics Company
A company like Apple or Microsoft sells a mix of hardware and digital entitlements.
*   **`FINISHED_GOOD`**: A physical laptop or a boxed keyboard. (Triggers warehouse picking, packing, and UPS/FedEx shipping label generation).
*   **`DIGITAL_GOOD`**: A software license key or an eBook download. (Skips the warehouse entirely; fulfillment means sending an automated email with a download link).
*   **`SERVICE`**: A "1-Year Extended Warranty" or "On-Site IT Setup." (Billed to the customer, but no physical item is shipped).
*   **`MARKETING_PKG_AUTO` (Bundle)**: A "Work from Home Kit" containing a Laptop (`FINISHED_GOOD`) and a Software License (`DIGITAL_GOOD`). The system knows to ship the laptop and email the license.

### 4. Construction & Equipment Rental
*   **`FINISHED_GOOD`**: Buying a cement mixer outright.
*   **`ASSET_USAGE`**: Renting the cement mixer for 3 days. (The inventory system tracks *when* it is due back, rather than permanently deducting it from stock).
*   **`SERVICE`**: Hiring a technician to operate the cement mixer for 5 hours.

### Summary of How the ERP Reacts:
| `ProductType` | System Behavior | Does it ship? | Is it stocked? |
| :--- | :--- | :---: | :---: |
| **`RAW_MATERIAL`** | Consumed during manufacturing. | ❌ No | ✅ Yes |
| **`FINISHED_GOOD`** | Picked, packed, and sold to customers. | ✅ Yes | ✅ Yes |
| **`DIGITAL_GOOD`** | Auto-fulfilled via email or API access. | ❌ No | ❌ No |
| **`SERVICE`** | Billed immediately, fulfilled by labor. | ❌ No | ❌ No |
| **`ASSET_USAGE`** | Checked out and checked back in (rentals). | ❌ No | 🔄 Calendar |
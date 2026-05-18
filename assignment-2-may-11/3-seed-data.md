Edited 2-ProductTypeMapping.md

Here is the standard OFBiz/HotWax XML seed data format. You can use this template to manually map data you scrape or copy from an e-commerce website. 

To make it easy to understand, I have used a real-world example of an apparel website selling a **"Classic Cotton T-Shirt"** in different colors and sizes.

### The Seed Data XML Template

```xml
<?xml version="1.0" encoding="UTF-8"?>
<entity-engine-xml>

    <!-- ========================================================= -->
    <!-- 1

[ignoring loop detection]

Here is the standard OFBiz/HotWax XML seed data format. You can use this template to manually map data you scrape or copy from an e-commerce website. 

To make it easy to understand, I have used a real-world example of an e-commerce website selling a **"Running Shoe"** in different colors and sizes.

### The Seed Data XML Template

```xml
<?xml version="1.0" encoding="UTF-8"?>
<entity-engine-xml>

    <!-- ========================================================= -->
    <!-- 1. Define the Feature Types (The Categories of Variation) -->
    <!-- Examples: Color, Size, Style, Fit                        -->
    <!-- ========================================================= -->
    <ProductFeatureType productFeatureTypeId="COLOR" description="Color"/>
    <ProductFeatureType productFeatureTypeId="SIZE" description="Size"/>

    <!-- ========================================================= -->
    <!-- 2. Define the Application Types (The Behavioral Roles)    -->
    <!-- These are usually pre-seeded in HotWax, but included here -->
    <!-- ========================================================= -->
    <ProductFeatureApplType productFeatureApplTypeId="SELECTABLE_FEATURE" description="Selectable Feature (Used on Virtual)"/>
    <ProductFeatureApplType productFeatureApplTypeId="STANDARD_FEATURE" description="Standard Feature (Used on Variant)"/>

    <!-- ========================================================= -->
    <!-- 3. Define the Specific Features (The Dictionary)          -->
    <!-- These are the actual options the customer chooses         -->
    <!-- ========================================================= -->
    <ProductFeature productFeatureId="COLOR_BLK" productFeatureTypeId="COLOR" description="Black"/>
    <ProductFeature productFeatureId="COLOR_WHT" productFeatureTypeId="COLOR" description="White"/>
    <ProductFeature productFeatureId="SIZE_10" productFeatureTypeId="SIZE" description="Size 10"/>
    <ProductFeature productFeatureId="SIZE_11" productFeatureTypeId="SIZE" description="Size 11"/>

    <!-- ========================================================= -->
    <!-- 4. Create the Virtual Product (The Marketing Template)    -->
    <!-- This is the "parent" product that customers see first.    -->
    <!-- ========================================================= -->
    <Product productId="SHOE_RUNNER_VIRTUAL" 
             productTypeId="FINISHED_GOOD" 
             isVirtual="Y" 
             isVariant="N" 
             internalName="Pro Running Shoe - Parent" 
             productName="Pro Running Shoe" 
             description="Lightweight professional running shoe."/>

    <!-- Apply "Selectable" features to the Virtual Product.       -->
    <!-- This tells the storefront to render dropdown menus.       -->
    <ProductFeatureAppl productId="SHOE_RUNNER_VIRTUAL" productFeatureId="COLOR_BLK" productFeatureApplTypeId="SELECTABLE_FEATURE" fromDate="2024-01-01 00:00:00"/>
    <ProductFeatureAppl productId="SHOE_RUNNER_VIRTUAL" productFeatureId="COLOR_WHT" productFeatureApplTypeId="SELECTABLE_FEATURE" fromDate="2024-01-01 00:00:00"/>
    <ProductFeatureAppl productId="SHOE_RUNNER_VIRTUAL" productFeatureId="SIZE_10" productFeatureApplTypeId="SELECTABLE_FEATURE" fromDate="2024-01-01 00:00:00"/>
    <ProductFeatureAppl productId="SHOE_RUNNER_VIRTUAL" productFeatureId="SIZE_11" productFeatureApplTypeId="SELECTABLE_FEATURE" fromDate="2024-01-01 00:00:00"/>

    <!-- ========================================================= -->
    <!-- 5. Create the Variant Products (The Physical SKUs)        -->
    <!-- These are the actual items sitting on the warehouse shelf.-->
    <!-- ========================================================= -->
    
    <!-- Variant 1: Black, Size 10 -->
    <Product productId="SHOE_BLK_10" productTypeId="FINISHED_GOOD" isVirtual="N" isVariant="Y" internalName="Pro Running Shoe - Black - 10" productName="Pro Running Shoe"/>
    <ProductFeatureAppl productId="SHOE_BLK_10" productFeatureId="COLOR_BLK" productFeatureApplTypeId="STANDARD_FEATURE" fromDate="2024-01-01 00:00:00"/>
    <ProductFeatureAppl productId="SHOE_BLK_10" productFeatureId="SIZE_10" productFeatureApplTypeId="STANDARD_FEATURE" fromDate="2024-01-01 00:00:00"/>
    
    <!-- Variant 2: Black, Size 11 -->
    <Product productId="SHOE_BLK_11" productTypeId="FINISHED_GOOD" isVirtual="N" isVariant="Y" internalName="Pro Running Shoe - Black - 11" productName="Pro Running Shoe"/>
    <ProductFeatureAppl productId="SHOE_BLK_11" productFeatureId="COLOR_BLK" productFeatureApplTypeId="STANDARD_FEATURE" fromDate="2024-01-01 00:00:00"/>
    <ProductFeatureAppl productId="SHOE_BLK_11" productFeatureId="SIZE_11" productFeatureApplTypeId="STANDARD_FEATURE" fromDate="2024-01-01 00:00:00"/>

    <!-- Variant 3: White, Size 10 -->
    <Product productId="SHOE_WHT_10" productTypeId="FINISHED_GOOD" isVirtual="N" isVariant="Y" internalName="Pro Running Shoe - White - 10" productName="Pro Running Shoe"/>
    <ProductFeatureAppl productId="SHOE_WHT_10" productFeatureId="COLOR_WHT" productFeatureApplTypeId="STANDARD_FEATURE" fromDate="2024-01-01 00:00:00"/>
    <ProductFeatureAppl productId="SHOE_WHT_10" productFeatureId="SIZE_10" productFeatureApplTypeId="STANDARD_FEATURE" fromDate="2024-01-01 00:00:00"/>

    <!-- ========================================================= -->
    <!-- 6. Link the Variants to the Virtual Product               -->
    <!-- This creates the Parent-Child hierarchy.                  -->
    <!-- ========================================================= -->
    <ProductAssoc productId="SHOE_RUNNER_VIRTUAL" productIdTo="SHOE_BLK_10" productAssocTypeId="PRODUCT_VARIANT" fromDate="2024-01-01 00:00:00"/>
    <ProductAssoc productId="SHOE_RUNNER_VIRTUAL" productIdTo="SHOE_BLK_11" productAssocTypeId="PRODUCT_VARIANT" fromDate="2024-01-01 00:00:00"/>
    <ProductAssoc productId="SHOE_RUNNER_VIRTUAL" productIdTo="SHOE_WHT_10" productAssocTypeId="PRODUCT_VARIANT" fromDate="2024-01-01 00:00:00"/>

</entity-engine-xml>
```

### How to use this for copying data:
1. **Identify Types (`ProductFeatureType`)**: Look at the e-commerce website's dropdown menus (e.g., Color, Storage, Size). Create one row for each dropdown category.
2. **Identify Values (`ProductFeature`)**: Look at the actual options inside those dropdowns (e.g., Red, 64GB, Large). Create one row for each unique option.
3. **Create the Parent (`Product isVirtual="Y"`)**: Copy the main title and description of the product page.
4. **Create the Children (`Product isVariant="Y"`)**: For every possible combination a user can select, create a unique variant ID (e.g., `SHIRT_RED_LG`).
5. **Apply Features (`ProductFeatureAppl`)**:
    * Apply all possible features to the Parent as `SELECTABLE_FEATURE`.
    * Apply the *specific* features to the Child as `STANDARD_FEATURE`.
6. **Link them up (`ProductAssoc`)**: Map the Parent ID to every Child ID using `PRODUCT_VARIANT`.
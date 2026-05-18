# Base Query

## version - 1.0
-----------------
### what have i done ?
- Selected different products by product ID - used intermediate tables to fetch the data, with a filter that thru date is null(product has not expired)

### pros:
- intermediate tables were enough to join on the basis of ID and fetch the required data

### query:
SELECT 
    P.PRODUCT_ID, P.INTERNAL_NAME, PS.STORE_NAME
FROM PRODUCT_STORE PS 
JOIN PRODUCT_STORE_CATALOG PSC ON PS.PRODUCT_STORE_ID = PSC.PRODUCT_STORE_ID AND PSC.THRU_DATE IS NULL OR PSC.THRU_DATE > NOW() 
JOIN PROD_CATALOG_CATEGORY PCC ON PSC.PROD_CATALOG_ID = PCC.PROD_CATALOG_ID AND PCC.THRU_DATE IS NULL OR PCC.THRU_DATE > NOW()
JOIN PRODUCT_CATEGORY_MEMBER PCM ON PCC.PRODUCT_CATEGORY_ID = PCM.PRODUCT_CATEGORY_ID
JOIN PRODUCT P ON PCM.PRODUCT_ID = P.PRODUCT_ID
WHERE PS.PRODUCT_STORE_ID = 'STORE' --for particular store
-- LIMIT 10 
;

### result:
PRODUCT_ID	INTERNAL_NAME	            STORE_NAME
10000	    V_200-gift-card-gift-bag	gorjana
100000	    V_test24k-gold-chain	    gorjana
100002	    Test-2025-02	            gorjana
10003	    2111-101a-G	                gorjana
10004	    2111-101b-G	                gorjana
10005	    2111-101c-G	                gorjana
100051	    V_parker-necklace-2	        gorjana
100052	    Parker-Gold	                gorjana
100053	    44064222838828	            gorjana
100054	    Parker-Rose Gold	        gorjana
---

## version - 1.1
------------------
### what have i done ?
- prev: Selected different products by product ID - used intermediate tables to fetch the data, with a filter that thru date is null(product has not expired)
- now: added from_date filter <= now() ; suggesting product is released

### pros: _na_
### query:
SELECT 
    PSC.PRODUCT_STORE_ID, P.PRODUCT_ID, P.INTERNAL_NAME
FROM PRODUCT_STORE_CATALOG PSC 
JOIN PROD_CATALOG_CATEGORY PCC ON PSC.PROD_CATALOG_ID = PCC.PROD_CATALOG_ID AND PSC.PRODUCT_STORE_ID='STORE'
AND PSC.FROM_DATE <= NOW() AND (PSC.THRU_DATE IS NULL OR PSC.THRU_DATE > NOW())
JOIN PRODUCT_CATEGORY_MEMBER PCM ON PCC.PRODUCT_CATEGORY_ID = PCM.PRODUCT_CATEGORY_ID
AND PCC.FROM_DATE <= NOW() AND (PCC.THRU_DATE IS NULL OR PCC.THRU_DATE > NOW())
JOIN PRODUCT P ON PCM.PRODUCT_ID = P.PRODUCT_ID 
AND PCM.FROM_DATE <= NOW() AND (PCM.THRU_DATE IS NULL OR PCM.THRU_DATE > NOW())
LIMIT 10 
;

### result:
PRODUCT_STORE_ID	PRODUCT_ID	INTERNAL_NAME
STORE	16007	30061800005
STORE	10000	V_30037376-billetera-onyx-para-hombre
STORE	10001	30037376001
STORE	10002	30037377001
STORE	10003	V_30011272-billetera-petoskey-para-hombre
STORE	10004	30011273003
STORE	10005	30011272003
STORE	10006	V_30067108-blusa-denim-para-mujer
STORE	10007	30067108005
STORE	10008	30067108003
---

## version - 1.2
### what is changed ?

PCC filters in PCC join
PCM filters in PCM join
PSC filters in WHERE
PRODUCT filters in WHERE

- count(*) + DISTINCT 
| PRODUCT_ID | CATEGORY |
| ---------- | -------- |
| WG-111     | MEN      |
| WG-111     | WINTER   |
| WG-111     | SALE     |

-- EXPLAIN ANALYZE
-- COUNT(DISTINCT P.PRODUCT_ID) AS PRODUCT_COUNT, --COUNT DISTINCT PRODUCTS
...
-- GROUP BY PSC.PRODUCT_STORE_ID 
;

### pros:
- DISTINCT : duplicate products are not counted, possible because there are several joins and products can repeat for category, etc.
- Applied filters while JOINING the table for the first time and for the 1st table PRODUCT_STORE - applied checks in WHERE clause. 


### query:
SELECT 
DISTINCT
P.PRODUCT_ID,
P.INTERNAL_NAME, 
 PSC.PRODUCT_STORE_ID --PER STORE [CHECK GRP BY]
FROM PRODUCT_STORE_CATALOG PSC
JOIN PROD_CATALOG_CATEGORY PCC ON PSC.PROD_CATALOG_ID = PCC.PROD_CATALOG_ID
AND PCC.FROM_DATE <= NOW()
AND (PCC.THRU_DATE IS NULL
     OR PCC.THRU_DATE > NOW())
JOIN PRODUCT_CATEGORY_MEMBER PCM ON PCC.PRODUCT_CATEGORY_ID = PCM.PRODUCT_CATEGORY_ID
AND PCM.FROM_DATE <= NOW()
AND (PCM.THRU_DATE IS NULL
     OR PCM.THRU_DATE > NOW())
JOIN PRODUCT P ON PCM.PRODUCT_ID = P.PRODUCT_ID  WHERE PSC.PRODUCT_STORE_ID='STORE' -- NOW, CONDITIONS ON BASE TABLE ARE ADDED IN WHERE
AND PSC.FROM_DATE <= NOW()
AND (PSC.THRU_DATE IS NULL
     OR PSC.THRU_DATE > NOW())
AND P.IS_VIRTUAL='Y'
AND P.IS_VARIANT='N'

### result: 1000 ROWS // BECAUSE OF LIMIT //ORIGINAL - 1003 ROWS
PRODUCT_ID	INTERNAL_NAME	                        PRODUCT_STORE_ID
10000	V_30037376-billetera-onyx-para-hombre	    STORE
10003	V_30011272-billetera-petoskey-para-hombre	STORE
10006	V_30067108-blusa-denim-para-mujer	        STORE
10012	V_30055847-flutter	                        STORE
10022	V_30055845-kelly-sleeveless	                STORE
10031	V_30053083-vivian	                        STORE
10040	V_30063857-bolso-code-hex-go-unisex	        STORE
10042	V_30058856-bolso-detroit-negro	            STORE
10045	V_30062182-mochila-hex-go-para-hombre	    STORE
10047	V_30063850-bolso-kilimanjaro-unisex	        STORE













---
<!-- template -->
## version - x.x
### what have i done ?
- 

### pros:

### query:


### result:


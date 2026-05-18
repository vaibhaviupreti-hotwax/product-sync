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














---
<!-- template -->
## version - x.x
### what have i done ?
- 

### pros:

### query:


### result:


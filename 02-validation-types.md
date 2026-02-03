# Validation Types

BoomGate supports 9 distinct validation types, each optimized for specific use cases.

## Overview

| Type | Use Case | Shopify Plus Required |
|------|----------|----------------------|
| GENERAL | Basic checkout conditions | No |
| PRODUCT | Product-specific validation | No |
| CUSTOMER | Customer data validation | No |
| CART | Cart-level conditions | No |
| TAGS | Tag-based product/customer rules | No |
| TIME | Date/time-based restrictions | No |
| ADDRESS | Shipping address validation | No |
| SHIPPING | Shipping method validation | No |
| REDUCTION | Gift cards & discount rules | Yes |
| BILLING | Billing address validation | Yes |

---

## GENERAL

Basic validation type for simple checkout conditions.

**Best for:**
- Simple cart total checks
- Basic customer authentication requirements
- General checkout blocking

**Available Properties:**
- Cart totals and counts
- Customer authentication status
- Basic product information

---

## PRODUCT

Validate checkout based on specific product attributes.

**Best for:**
- SKU-based quantity limits
- Product metafield validation
- Collection-based restrictions
- Product type/vendor rules

**Available Properties:**
- `PRODUCT_SKU` - SKU code
- `PRODUCT_TITLE` - Product title
- `PRODUCT_VENDOR` - Vendor name
- `PRODUCT_TYPE` - Product type
- `PRODUCT_HANDLE` - URL handle
- `PRODUCT_COLLECTION` - Collection membership
- `PRODUCT_METAFIELD` - Custom metafields
- `PRODUCT_WEIGHT` - Product weight
- `LINE_QUANTITY` - Quantity per line
- `LINE_TOTAL` - Line item total

**Example Use Cases:**
- "Block checkout if SKU 'LIMITED-001' quantity exceeds 1"
- "Require minimum 10 units for products in 'wholesale' collection"

---

## CUSTOMER

Validate based on customer data and account status.

**Best for:**
- Customer tag-based restrictions
- Authentication requirements
- Customer metafield validation
- Order history rules

**Available Properties:**
- `CUSTOMER_IS_AUTHENTICATED` - Login status
- `CUSTOMER_TAGS` - Customer tags
- `CUSTOMER_EMAIL` - Email address
- `CUSTOMER_ORDER_NUMBER` - Total orders placed
- `CUSTOMER_AMOUNT_SPENT` - Lifetime spend
- `CUSTOMER_METAFIELD` - Custom metafields
- `CUSTOMER_FIRST_NAME` / `CUSTOMER_LAST_NAME`
- `CUSTOMER_COMPANY_NAME`

**Example Use Cases:**
- "Only wholesale-tagged customers can order more than 5 units"
- "Block unauthenticated customers from checkout"
- "VIP customers get higher quantity limits"

---

## CART

Validate cart-level attributes and aggregates.

**Best for:**
- Minimum/maximum order values
- Total quantity limits
- Cart attribute validation
- Line count restrictions

**Available Properties:**
- `CART_TOTAL` - Total cart value
- `CART_SUBTOTAL` - Subtotal (before tax/shipping)
- `CART_TOTAL_WEIGHT` - Total weight
- `CART_ATTRIBUTE` - Custom cart attributes
- `LINES_COUNT` - Number of line items
- `LINE_QUANTITY` - Quantity per line
- `LINE_ATTRIBUTE` - Line item attributes

**Example Use Cases:**
- "Minimum order value of $50"
- "Maximum 10 items per order"
- "Block checkout if cart attribute 'gift_wrap' is missing"

---

## TAGS

Combine product and customer tags for complex rules.

**Best for:**
- Product-customer tag combinations
- Tag-based quantity limits
- Multi-tag validation
- Free gift restrictions

**Available Properties:**
- `PRODUCT_TAGS` - Product tags
- `CUSTOMER_TAGS` - Customer tags
- `LINES_WITH_ANY_TAG_COUNT` - Count lines with specific tags
- `LINES_WITH_NO_TAGS_COUNT` - Count untagged lines
- `LINE_QUANTITY` - Quantity per line

**Example Use Cases:**
- "Customers tagged 'restricted' cannot buy products tagged 'premium'"
- "Maximum 1 free-gift tagged item per order"
- "Wholesale customers must buy in multiples of 10"

---

## TIME

Validate based on date and time constraints.

**Best for:**
- Limited-time sales
- Pre-order windows
- Business hours restrictions
- Seasonal availability

**Available Properties:**
- `DATE` - Current date (shop timezone)
- `TIME` - Current time (shop timezone)
- `DATE_TIME` - Current datetime
- All product and customer properties (for combination rules)

**Operators:**
- `DATE_AFTER` / `DATE_BEFORE`
- `TIME_AFTER` / `TIME_BEFORE`
- `DATE_TIME_AFTER` / `DATE_TIME_BEFORE`
- `DATE_EQUAL` / `DATE_EQUAL_OR_AFTER` / `DATE_EQUAL_OR_BEFORE`

**Example Use Cases:**
- "Sale items cannot be purchased after June 1st"
- "No checkout between 2 AM and 6 AM"
- "Pre-orders only available until launch date"

---

## ADDRESS

Validate shipping address fields.

**Best for:**
- Country/region restrictions
- PO Box blocking
- Address format validation
- Province/state rules

**Available Properties:**
- `ADDRESS_COUNTRY_CODE` - ISO country code (US, GB, CA, etc.)
- `ADDRESS_PROVINCE_CODE` - State/province code (CA, NY, ON, etc.)
- `ADDRESS_CITY` - City name
- `ADDRESS_ZIP` - Postal/ZIP code
- `ADDRESS_1` / `ADDRESS_2` - Street address lines
- `ADDRESS_COMPANY` - Company name
- `ADDRESS_FIRST_NAME` / `ADDRESS_LAST_NAME`

**Example Use Cases:**
- "Block shipping to Hawaii and Alaska for hazmat products"
- "No PO Box deliveries for oversized items"
- "Restrict certain products to US only"

---

## SHIPPING

Validate shipping method and delivery options.

**Best for:**
- Pickup restrictions
- Multi-shipment rules
- Shipping method requirements
- Delivery type validation

**Available Properties:**
- `SHIPPING_SELECTED_SHIPPING_TITLE` - Shipping method name
- `SHIPPING_SELECTED_SHIPPING_TYPE` - Shipping type (SHIPPING, PICK_UP, etc.)
- `TOTAL_SHIPMENTS_COUNT` - Number of shipments
- Product and customer tag properties

**Example Use Cases:**
- "Products tagged 'no-pickup' cannot use store pickup"
- "Block orders that would split into multiple shipments"
- "Require express shipping for perishable items"

---

## REDUCTION (Shopify Plus)

Validate gift cards and discount codes.

**Best for:**
- Limiting gift card usage
- Discount code restrictions
- Preventing discount stacking
- Sale item discount blocking

**Available Properties:**
- `APPLIED_GIFT_CARDS_COUNT` - Number of gift cards applied
- `TOTAL_GIFT_CARD_DISCOUNTS_AMOUNT` - Total gift card value
- `APPLIED_DISCOUNT_CODES_COUNT` - Number of discount codes
- `APPLIED_DISCOUNT_CODE` - Specific discount code

**Example Use Cases:**
- "Maximum 1 gift card per order"
- "Discount codes cannot be used with sale items"
- "Block checkout if discount exceeds 50%"

---

## BILLING (Shopify Plus)

Validate billing address fields.

**Best for:**
- Billing country restrictions
- Address matching requirements
- Fraud prevention rules

**Available Properties:**
- `BILLING_ADDRESS_COUNTRY_CODE`
- `BILLING_ADDRESS_PROVINCE_CODE`
- `BILLING_ADDRESS_CITY`
- `BILLING_ADDRESS_ZIP`
- `BILLING_ADDRESS_1` / `BILLING_ADDRESS_2`
- `BILLING_ADDRESS_COMPANY`
- `BILLING_ADDRESS_FIRST_NAME` / `BILLING_ADDRESS_LAST_NAME`

**Example Use Cases:**
- "Block orders with billing address from high-risk countries"
- "Require billing and shipping country to match"

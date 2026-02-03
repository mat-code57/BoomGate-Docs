# Condition Properties

Condition properties define WHAT aspect of the checkout you want to validate. BoomGate provides 60+ properties across different categories.

## Property Structure

Each property has:
- **Name** - Identifier used in rules
- **Allowed Operators** - Which operators can be used
- **Value Type** - What kind of input is expected
- **AND Allowed** - Whether nested conditions are supported

---

## Product Properties

Properties for validating product data.

| Property | Description | Value Type | AND Allowed |
|----------|-------------|------------|-------------|
| `PRODUCT_TAGS` | Product tags | Multiple (array) | Yes |
| `PRODUCT_TYPE` | Product type | Text | Yes |
| `PRODUCT_TITLE` | Product title | Text | Yes |
| `PRODUCT_VENDOR` | Vendor name | Text | Yes |
| `PRODUCT_HANDLE` | URL handle | Text | Yes |
| `PRODUCT_SKU` | SKU code | Text | Yes |
| `PRODUCT_WEIGHT` | Weight (grams) | Numeric | Yes |
| `PRODUCT_REQUIRES_SHIPPING` | Requires shipping | Boolean | Yes |
| `PRODUCT_IS_GIFT_CARD` | Is gift card | Boolean | Yes |
| `PRODUCT_COLLECTION` | Collection IDs | Multiple | Yes |
| `PRODUCT_METAFIELD` | Custom metafield | Varies | Yes |

### Product Tag Operators

```
PRODUCT_TAGS + HAS_TAGS         → All specified tags present
PRODUCT_TAGS + HAS_ANY_TAG      → At least one tag present
PRODUCT_TAGS + DOES_NOT_HAVE_TAGS → None of the tags present
```

### Product Metafield Usage

Requires namespace and key in the value:

```json
{
  "property": "PRODUCT_METAFIELD",
  "operator": "METAFIELD_VALUE_EQUALS",
  "value": {
    "namespace": "custom",
    "key": "restricted",
    "value": "true"
  }
}
```

---

## Product Aggregates

Properties for counting and summing product data across the cart.

| Property | Description | Value Type |
|----------|-------------|------------|
| `PRODUCTS_TOTAL_COUNT` | Total products in cart | Numeric |
| `PRODUCTS_WITH_ANY_TAG_TOTAL_COUNT` | Count of products with specified tags | Numeric + Tags |
| `PRODUCTS_WITH_HANDLE_TOTAL_COUNT` | Count of products with specific handle | Numeric + Handle |
| `PRODUCTS_WITH_VENDOR_TOTAL_AMOUNT` | Amount/count from specific vendor | Numeric + Vendor |

### Example: Count Products with Tag

```json
{
  "property": "PRODUCTS_WITH_ANY_TAG_TOTAL_COUNT",
  "operator": "GREATHER_THAN",
  "value": {
    "value": ["free-gift"],
    "count": "1"
  }
}
```

---

## Line Item Properties

Properties for individual cart line items.

| Property | Description | Value Type | AND Allowed |
|----------|-------------|------------|-------------|
| `LINE_QUANTITY` | Quantity of line | Numeric | Yes |
| `LINE_TOTAL` | Total cost of line | Numeric | Yes |
| `LINE_SUBTOTAL` | Subtotal of line | Numeric | Yes |
| `LINE_AMOUNT_PER_QUANTITY` | Price per unit | Numeric | Yes |
| `LINE_COMPARE_AT_AMOUNT_PER_QUANTITY` | Compare-at price | Numeric | Yes |
| `LINE_ATTRIBUTE` | Line attribute value | Text | Yes |
| `LINES_COUNT` | Total lines in cart | Numeric | No |
| `LINES_WITH_ANY_TAG_COUNT` | Lines with specified tags | Numeric | No |
| `LINES_WITH_NO_TAGS_COUNT` | Lines without tags | Numeric | No |
| `UNIQUE_LINES_ATTRIBUTES_VALUES_COUNT` | Unique attribute values | Numeric | No |

### Line Attribute Usage

Requires attribute key in the value:

```json
{
  "property": "LINE_ATTRIBUTE",
  "operator": "ATTRIBUTE_VALUE_EQUALS",
  "value": {
    "attributeKey": "size",
    "value": "XL"
  }
}
```

---

## Cart Properties

Properties for cart-level data.

| Property | Description | Value Type |
|----------|-------------|------------|
| `CART_TOTAL` | Total cart amount | Numeric |
| `CART_SUBTOTAL` | Subtotal (before tax/shipping) | Numeric |
| `CART_TAX` | Tax amount | Numeric |
| `CART_DUTY` | Duty amount | Numeric |
| `CART_TOTAL_WEIGHT` | Total weight | Numeric |
| `CART_ATTRIBUTE` | Cart attribute value | Text |

### Cart Attribute Usage

```json
{
  "property": "CART_ATTRIBUTE",
  "operator": "ATTRIBUTE_VALUE_EQUALS",
  "value": {
    "attributeKey": "promo_code",
    "value": "SUMMER2024"
  }
}
```

---

## Customer Properties

Properties for customer data.

| Property | Description | Value Type | AND Allowed |
|----------|-------------|------------|-------------|
| `CUSTOMER_IS_AUTHENTICATED` | Is logged in | Boolean | Yes |
| `CUSTOMER_FIRST_NAME` | First name | Text | Yes |
| `CUSTOMER_LAST_NAME` | Last name | Text | Yes |
| `CUSTOMER_EMAIL` | Email address | Text | Yes |
| `CUSTOMER_PHONE` | Phone number | Text | Yes |
| `CUSTOMER_COMPANY_NAME` | Company name | Text | Yes |
| `CUSTOMER_ORDER_NUMBER` | Total orders | Numeric | Yes |
| `CUSTOMER_AMOUNT_SPENT` | Lifetime spend | Numeric | Yes |
| `CUSTOMER_TAGS` | Customer tags | Multiple | Yes |
| `CUSTOMER_METAFIELD` | Custom metafield | Varies | Yes |

### Customer Authentication Check

```json
{
  "property": "CUSTOMER_IS_AUTHENTICATED",
  "operator": "EQUALS",
  "value": {
    "value": "false"
  }
}
```

---

## Shipping Address Properties

Properties for the shipping/delivery address.

| Property | Description | Example Values |
|----------|-------------|----------------|
| `ADDRESS_COUNTRY_CODE` | ISO country code | US, GB, CA, AU |
| `ADDRESS_PROVINCE_CODE` | State/province code | CA, NY, ON, NSW |
| `ADDRESS_CITY` | City name | Los Angeles, London |
| `ADDRESS_ZIP` | Postal code | 90210, SW1A 1AA |
| `ADDRESS_1` | Street address line 1 | 123 Main St |
| `ADDRESS_2` | Street address line 2 | Apt 4B |
| `ADDRESS_COMPANY` | Company name | Acme Inc |
| `ADDRESS_FIRST_NAME` | First name | John |
| `ADDRESS_LAST_NAME` | Last name | Doe |

### Country/Province Codes

Use ISO 3166 codes:
- Countries: `US`, `GB`, `CA`, `AU`, `DE`, `FR`, etc.
- US States: `CA`, `NY`, `TX`, `FL`, etc.
- Canadian Provinces: `ON`, `BC`, `QC`, `AB`, etc.

---

## Billing Address Properties

Properties for the billing address (Shopify Plus only).

| Property | Description |
|----------|-------------|
| `BILLING_ADDRESS_COUNTRY_CODE` | Billing country code |
| `BILLING_ADDRESS_PROVINCE_CODE` | Billing state/province |
| `BILLING_ADDRESS_CITY` | Billing city |
| `BILLING_ADDRESS_ZIP` | Billing postal code |
| `BILLING_ADDRESS_1` | Billing street line 1 |
| `BILLING_ADDRESS_2` | Billing street line 2 |
| `BILLING_ADDRESS_COMPANY` | Billing company |
| `BILLING_ADDRESS_FIRST_NAME` | Billing first name |
| `BILLING_ADDRESS_LAST_NAME` | Billing last name |

---

## Checkout/Localization Properties

Properties for checkout context.

| Property | Description | Example Values |
|----------|-------------|----------------|
| `CHECKOUT_ISO_LANGUAGE` | Checkout language | en, es, fr, de |
| `CHECKOUT_ISO_COUNTRY` | Checkout country | US, GB, CA |
| `CHECKOUT_MARKET_HANDLE` | Market identifier | us-market |

---

## Time Properties

Properties for date/time validation.

| Property | Description | Format |
|----------|-------------|--------|
| `DATE` | Current date (shop timezone) | YYYY-MM-DD |
| `TIME` | Current time (shop timezone) | HH:MM |
| `DATE_TIME` | Current datetime | YYYY-MM-DDTHH:MM |

---

## Shipping Properties

Properties for shipping method validation.

| Property | Description | Example Values |
|----------|-------------|----------------|
| `SHIPPING_SELECTED_SHIPPING_TITLE` | Selected shipping method | Standard Shipping |
| `SHIPPING_SELECTED_SHIPPING_TYPE` | Shipping type | SHIPPING, PICK_UP |
| `TOTAL_SHIPMENTS_COUNT` | Number of shipments | 1, 2, 3 |

---

## Reduction Properties (Shopify Plus)

Properties for gift cards and discounts.

| Property | Description | Value Type |
|----------|-------------|------------|
| `APPLIED_GIFT_CARDS_COUNT` | Gift cards applied | Numeric |
| `TOTAL_GIFT_CARD_DISCOUNTS_AMOUNT` | Total gift card value | Numeric |
| `APPLIED_DISCOUNT_CODES_COUNT` | Discount codes applied | Numeric |
| `APPLIED_DISCOUNT_CODE` | Specific discount code | Text |

---

## Properties That Don't Require Values

Some counting properties only need a count threshold:

```
APPLIED_DISCOUNT_CODES_COUNT
TOTAL_SHIPMENTS_COUNT
LINES_COUNT
APPLIED_GIFT_CARDS_COUNT
PRODUCTS_TOTAL_COUNT
```

Example:
```json
{
  "property": "LINES_COUNT",
  "operator": "GREATHER_THAN",
  "value": {
    "count": "5"
  }
}
```

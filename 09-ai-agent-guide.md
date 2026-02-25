# AI Agent Guide

Guide for AI agents building BoomGate rules.

---

## Overview

BoomGate supports **3 rule categories**, all sharing the same condition engine:

1. **Checkout Validation** - Block checkout + show error message
2. **Payment Customization** - Hide payment methods by name ([full guide](./11-payment-customization.md))
3. **Delivery Customization** - Hide delivery options by title ([full guide](./12-delivery-customization.md))

When building rules, follow these guidelines to create correct, efficient rules.

---

## CRITICAL: Determine Rule Category First

Before building any rule, determine which category the user needs:

| User wants to... | Rule Category | Action |
|-------------------|--------------|--------|
| Block/prevent checkout | Checkout Validation | Error message (`msg`) |
| Hide a payment method | Payment Customization | `action: "HIDE_PAYMENT_METHOD"` |
| Hide a delivery/shipping option | Delivery Customization | `action: "HIDE_DELIVERY_OPTION"` |

**Keywords for Payment Customization:** hide payment, remove payment, disable payment method, COD, cash on delivery, bank transfer, PayPal
**Keywords for Delivery Customization:** hide shipping, remove delivery, disable shipping option, hide express, hide pickup, hide free shipping

---

## Rule Building Process

### 1. Understand the Requirement

Parse the user's request to identify:
- **Rule category** - Block checkout, hide payment, or hide delivery?
- **What** should be blocked/hidden
- **Who** it applies to (customers, products, locations)
- **When** it should apply (always, time-based)
- **For checkout validation:** Error message to display
- **For payment/delivery:** Target name (payment method or delivery option)

### 2. Select Validation Type

Choose based on primary data needed:

| If checking... | Use Type |
|----------------|----------|
| Product tags, SKU, vendor | PRODUCT |
| Customer tags, authentication | CUSTOMER |
| Cart totals, line counts | CART |
| Product + Customer tags combined | TAGS |
| Shipping address | ADDRESS |
| Billing address | BILLING (Plus) |
| Shipping method, pickup | SHIPPING |
| Date/time restrictions | TIME |
| Gift cards, discounts | REDUCTION (Plus) |

### 3. Build Conditions

For each condition:
1. Choose appropriate **property**
2. Select compatible **operator**
3. Provide correct **value** format
4. Add **andConditions** if needed for line-level logic

### 4. Configure Input

Input must include data needed by GraphQL:
- Tags used in conditions → add to `hasProductTags` / `hasCustomerTags`
- Metafields used → add namespace/key
- Attributes used → add attribute key

### 5. Set Mode

- **ALL**: All conditions must be true (most common)
- **ANY**: At least one condition must be true

### 6. Set the Action

**For Checkout Validation - Write Error Message:**
- Clear explanation of why blocked
- Include placeholders for dynamic content: `[product_title]`
- Add translations if needed

**For Payment Customization:**
- Set `action: "HIDE_PAYMENT_METHOD"`
- Set `actionValue` to the payment method name/pattern
- NO `msg` field

**For Delivery Customization:**
- Set `action: "HIDE_DELIVERY_OPTION"`
- Set `actionValue` to the delivery option title/pattern
- NO `msg` field

---

## Common Patterns

### Pattern: Quantity Limit

```json
{
  "type": "PRODUCT",
  "configuration": {
    "conditions": [
      {
        "property": "PRODUCT_TAGS",
        "operator": "HAS_TAGS",
        "value": { "value": ["limited"] },
        "andConditions": [
          {
            "property": "LINE_QUANTITY",
            "operator": "GREATHER_THAN",
            "value": { "value": "X" }
          }
        ]
      }
    ],
    "msg": { "en": "Maximum X per order for [product_title]" },
    "mode": "ALL"
  },
  "input": {
    "hasProductTags": ["limited"]
  }
}
```

### Pattern: Regional Restriction

```json
{
  "type": "ADDRESS",
  "configuration": {
    "conditions": [
      {
        "property": "ADDRESS_COUNTRY_CODE",
        "operator": "ONE_OF",
        "value": { "value": ["XX", "YY"] }
      },
      {
        "property": "PRODUCT_TAGS",
        "operator": "HAS_TAGS",
        "value": { "value": ["restricted-region"] }
      }
    ],
    "msg": { "en": "[product_title] cannot ship to your country" },
    "mode": "ALL"
  },
  "input": {
    "hasProductTags": ["restricted-region"]
  }
}
```

### Pattern: Customer Type Restriction

```json
{
  "type": "TAGS",
  "configuration": {
    "conditions": [
      {
        "property": "CUSTOMER_TAGS",
        "operator": "DOES_NOT_HAVE_TAGS",
        "value": { "value": ["allowed-tag"] }
      },
      {
        "property": "PRODUCT_TAGS",
        "operator": "HAS_TAGS",
        "value": { "value": ["restricted-product"] }
      }
    ],
    "msg": { "en": "Only authorized customers can purchase [product_title]" },
    "mode": "ALL"
  },
  "input": {
    "hasCustomerTags": ["allowed-tag"],
    "hasProductTags": ["restricted-product"]
  }
}
```

### Pattern: Time Window

```json
{
  "type": "TIME",
  "configuration": {
    "conditions": [
      {
        "property": "DATE_TIME",
        "operator": "DATE_TIME_AFTER",
        "value": { "value": "YYYY-MM-DDTHH:MM" }
      }
    ],
    "msg": { "en": "This offer has expired" },
    "mode": "ALL"
  },
  "input": {
    "dateTimeAfter": "YYYY-MM-DDTHH:MM:SS"
  }
}
```

---

## Property-Type Compatibility (CRITICAL)

Each property can ONLY be used with specific validation types. This is enforced by the GraphQL schema - each validation type queries different data from Shopify.

**Always call `list_properties` with your chosen validation type** to see available properties.

### Type-Exclusive Properties:

| Type | Exclusive Properties |
|------|---------------------|
| ADDRESS | ADDRESS_1, ADDRESS_2, ADDRESS_CITY, ADDRESS_ZIP, ADDRESS_COUNTRY_CODE, ADDRESS_PROVINCE_CODE, ADDRESS_COMPANY, ADDRESS_FIRST_NAME, ADDRESS_LAST_NAME |
| BILLING | All BILLING_ADDRESS_* properties |
| SHIPPING | SHIPPING_SELECTED_SHIPPING_TITLE, SHIPPING_SELECTED_SHIPPING_TYPE |
| TIME | DATE, TIME, DATE_TIME |
| REDUCTION | APPLIED_GIFT_CARDS_COUNT, APPLIED_DISCOUNT_CODES_COUNT, APPLIED_DISCOUNT_CODE, TOTAL_GIFT_CARD_DISCOUNTS_AMOUNT |
| CUSTOMER | CUSTOMER_METAFIELD, CUSTOMER_ORDER_NUMBER, CUSTOMER_AMOUNT_SPENT |
| PRODUCT | PRODUCT_COLLECTION, PRODUCT_METAFIELD |

### Common Shared Properties:

These work with multiple validation types:
- `PRODUCT_TAGS` - PRODUCT, TAGS, TIME, ADDRESS, SHIPPING
- `CUSTOMER_TAGS` - CUSTOMER, TAGS, TIME, SHIPPING
- `PRODUCT_TITLE`, `PRODUCT_HANDLE` - Most types
- `LINE_QUANTITY` - Most types
- `CART_TOTAL` - GENERAL, PRODUCT, CUSTOMER, CART, ADDRESS

**WRONG - Using ADDRESS property with CART type:**
```json
{ "type": "CART", "conditions": [{ "property": "ADDRESS_COUNTRY_CODE", ... }] }
```

**CORRECT - Using ADDRESS type for address validation:**
```json
{ "type": "ADDRESS", "conditions": [{ "property": "ADDRESS_COUNTRY_CODE", ... }] }
```

---

## Property-Operator Compatibility

### String Properties
Use: `EQUALS`, `CONTAINS`, `ONE_OF`, `MATCHES_REGEX`, `IS_BLANK`

### Numeric Properties
Use: `EQUALS`, `GREATHER_THAN`, `LESS_THAN`, `IS_MULTIPLE_OF`

### Tag Properties
Use: `HAS_TAGS`, `HAS_ANY_TAG`, `DOES_NOT_HAVE_TAGS`

### Date/Time Properties
Use: `DATE_AFTER`, `DATE_BEFORE`, `TIME_AFTER`, `TIME_BEFORE`, `DATE_TIME_AFTER`, `DATE_TIME_BEFORE`

---

## Input Synchronization Rules

### For Product Tags

If using `PRODUCT_TAGS` with:
- `HAS_TAGS` → add tags to `hasProductTags`
- `HAS_ANY_TAG` → add tags to `hasAnyProductTag`
- `DOES_NOT_HAVE_TAGS` → add tags to `hasProductTags`

### For Customer Tags

If using `CUSTOMER_TAGS` with:
- `HAS_TAGS` → add tags to `hasCustomerTags`
- `HAS_ANY_TAG` → add tags to `hasAnyCustomerTag`
- `DOES_NOT_HAVE_TAGS` → add tags to `hasCustomerTags`

### For Metafields

If using `PRODUCT_METAFIELD` or `CUSTOMER_METAFIELD`:
- Add namespace to `productMetafieldNamespace` / `customerMetafieldNamespace`
- Add key to `productMetafieldKey` / `customerMetafieldKey`

### For Attributes

If using `CART_ATTRIBUTE` or `LINE_ATTRIBUTE`:
- Add key to `cartAttributeKey` / `lineAttributeKey`

### For Unused Fields

Set unused metafield/attribute fields to `"BLANK"`:
```json
{
  "productMetafieldNamespace": "BLANK",
  "productMetafieldKey": "BLANK"
}
```

---

## Single-Use Properties (IMPORTANT)

Some properties can only be used ONCE per validation rule. The UI will disable these properties after first use.

### Properties That Can Only Appear Once:

| Property | Restriction |
|----------|-------------|
| `PRODUCT_TAGS` | Only one condition can use product tags (HAS_TAGS, HAS_ANY_TAG, or DOES_NOT_HAVE_TAGS) |
| `CUSTOMER_TAGS` | Only one condition can use customer tags |
| `PRODUCT_METAFIELD` | Only one product metafield check per rule |
| `CUSTOMER_METAFIELD` | Only one customer metafield check per rule |
| `LINE_ATTRIBUTE` | Only one line attribute check per rule |
| `CART_ATTRIBUTE` | Only one cart attribute check per rule |
| `DATE_TIME` | Maximum one DATE_TIME_AFTER and one DATE_TIME_BEFORE |
| `TIME` | Maximum one TIME_AFTER and one TIME_BEFORE |

### What This Means for Rule Building:

**WRONG - Multiple product tag conditions:**
```json
{
  "conditions": [
    { "property": "PRODUCT_TAGS", "operator": "HAS_TAGS", "value": { "value": ["wholesale"] } },
    { "property": "PRODUCT_TAGS", "operator": "HAS_ANY_TAG", "value": { "value": ["sale", "clearance"] } }
  ]
}
```

**CORRECT - Single product tag condition with combined tags:**
```json
{
  "conditions": [
    { "property": "PRODUCT_TAGS", "operator": "HAS_TAGS", "value": { "value": ["wholesale"] } }
  ]
}
```

### If User Needs Multiple Tag Checks:

When a user needs to check multiple different tag conditions, you have two options:

1. **Combine into one condition** if the logic allows (e.g., use HAS_ANY_TAG with all tags)
2. **Create separate validation rules** for each tag check

Example: "Block if product has 'restricted' tag AND customer doesn't have 'vip' tag"
- This requires checking PRODUCT_TAGS and CUSTOMER_TAGS - this is ALLOWED (different properties)

Example: "Block if product has 'fragile' tag OR product has 'hazmat' tag"
- Use ONE condition with HAS_ANY_TAG: `{ "value": ["fragile", "hazmat"] }`

---

## Validation Checklist

Before finalizing a rule, verify:

### For ALL Rule Types:
- [ ] Validation type matches conditions used
- [ ] All operators are compatible with their properties
- [ ] Input contains all tags referenced in conditions
- [ ] Input contains metafield namespace/key if used
- [ ] Input contains attribute keys if used
- [ ] Mode (ALL/ANY) is correct for the logic
- [ ] Value formats are correct (dates, numbers, arrays)

### For Checkout Validation Only:
- [ ] Error message (`msg`) is clear and helpful
- [ ] At least English (`en`) message is provided

### For Payment Customization Only:
- [ ] `action` is `"HIDE_PAYMENT_METHOD"`
- [ ] `actionValue` has the payment method name/pattern
- [ ] NO `msg` field in configuration

### For Delivery Customization Only:
- [ ] `action` is `"HIDE_DELIVERY_OPTION"`
- [ ] `actionValue` has the delivery option title/pattern
- [ ] NO `msg` field in configuration

---

## Common Mistakes

### 1. Missing Input Tags

**Wrong:**
```json
{
  "conditions": [{ "property": "PRODUCT_TAGS", "value": { "value": ["sale"] } }],
  "input": {}  // Missing tags!
}
```

**Correct:**
```json
{
  "conditions": [{ "property": "PRODUCT_TAGS", "value": { "value": ["sale"] } }],
  "input": { "hasProductTags": ["sale"] }
}
```

### 2. Wrong Operator for Property

**Wrong:**
```json
{ "property": "LINE_QUANTITY", "operator": "CONTAINS" }  // CONTAINS is for strings
```

**Correct:**
```json
{ "property": "LINE_QUANTITY", "operator": "GREATHER_THAN" }
```

### 3. Wrong Value Type

**Wrong:**
```json
{ "property": "PRODUCT_TAGS", "value": { "value": "sale" } }  // String instead of array
```

**Correct:**
```json
{ "property": "PRODUCT_TAGS", "value": { "value": ["sale"] } }  // Array
```

### 4. AND Conditions on Cart-Level Properties

**Wrong:**
```json
{
  "property": "CART_TOTAL",
  "andConditions": [...]  // Cart total is not line-level
}
```

**Correct:**
```json
{
  "property": "PRODUCT_TAGS",  // Line-level property
  "andConditions": [...]
}
```

### 5. Using "value" Instead of "count" for Count Properties

**Wrong:**
```json
{ "property": "LINES_COUNT", "value": { "value": "5" } }  // Should use count!
```

**Correct:**
```json
{ "property": "LINES_COUNT", "value": { "count": "5" } }  // Correct
```

This applies to: `LINES_COUNT`, `PRODUCTS_TOTAL_COUNT`, `APPLIED_GIFT_CARDS_COUNT`, `APPLIED_DISCOUNT_CODES_COUNT`, `TOTAL_SHIPMENTS_COUNT`

---

## Value Reference

### Boolean Values
```json
{ "value": "true" }   // or "false" - always string
```

### Numeric Values
```json
{ "value": "100" }    // Always string representation
```

### Array Values
```json
{ "value": ["item1", "item2"] }
```

### Date Values
```json
{ "value": "2024-12-31" }  // YYYY-MM-DD
```

### Time Values
```json
{ "value": "09:00" }  // HH:MM (24-hour)
```

### DateTime Values
```json
{ "value": "2024-12-31T23:59" }  // YYYY-MM-DDTHH:MM
```

### Count Values (for aggregate properties with tags)
```json
{ "count": "5", "value": ["tag1", "tag2"] }
```

Use this format for properties that count items matching specific tags:
- `LINES_WITH_ANY_TAG_COUNT`
- `LINES_WITH_NO_TAGS_COUNT`
- `PRODUCTS_WITH_ANY_TAG_TOTAL_COUNT`

### Count-Only Values (NO_VALUE_PROPERTIES)

These properties use ONLY `count`, NOT `value`:

```json
{ "count": "5" }
```

**IMPORTANT:** The following properties require `count` instead of `value`:
- `LINES_COUNT` - Total line items in cart
- `PRODUCTS_TOTAL_COUNT` - Total products in cart
- `APPLIED_GIFT_CARDS_COUNT` - Gift cards applied
- `APPLIED_DISCOUNT_CODES_COUNT` - Discount codes applied
- `TOTAL_SHIPMENTS_COUNT` - Number of shipments

**Correct example:**
```json
{
  "property": "LINES_COUNT",
  "operator": "GREATHER_THAN",
  "value": { "count": "5" }
}
```

**WRONG - will fail:**
```json
{
  "property": "LINES_COUNT",
  "operator": "GREATHER_THAN",
  "value": { "value": "5" }
}
```

---

## Payment Customization Patterns

### Pattern: Hide Payment Method by Region

```json
{
  "type": "ADDRESS",
  "configuration": {
    "conditions": [
      {
        "property": "ADDRESS_COUNTRY_CODE",
        "operator": "DOES_NOT_EQUAL",
        "value": { "value": "US" }
      }
    ],
    "mode": "ALL",
    "action": "HIDE_PAYMENT_METHOD",
    "actionValue": "Cash on Delivery"
  },
  "input": { "hasProductTags": [], "cartAttributeKey": "BLANK" }
}
```

### Pattern: Hide Payment Method by Customer Tag

```json
{
  "type": "CUSTOMER",
  "configuration": {
    "conditions": [
      {
        "property": "CUSTOMER_TAGS",
        "operator": "DOES_NOT_HAVE_TAGS",
        "value": { "value": ["b2b"] }
      }
    ],
    "mode": "ALL",
    "action": "HIDE_PAYMENT_METHOD",
    "actionValue": "Invoice"
  },
  "input": {
    "hasCustomerTags": ["b2b"],
    "customerMetafieldNamespace": "BLANK",
    "customerMetafieldKey": "BLANK"
  }
}
```

### Pattern: Hide Payment Method by Cart Value

```json
{
  "type": "CART",
  "configuration": {
    "conditions": [
      {
        "property": "CART_TOTAL",
        "operator": "GREATHER_THAN",
        "value": { "value": "5000" }
      }
    ],
    "mode": "ALL",
    "action": "HIDE_PAYMENT_METHOD",
    "actionValue": "PayPal"
  },
  "input": { "lineAttributeKey": "BLANK", "cartAttributeKey": "BLANK" }
}
```

---

## Delivery Customization Patterns

### Pattern: Hide Delivery Option by Product Tag

```json
{
  "type": "TAGS",
  "configuration": {
    "conditions": [
      {
        "property": "PRODUCT_TAGS",
        "operator": "HAS_ANY_TAG",
        "value": { "value": ["heavy", "oversized"] }
      }
    ],
    "mode": "ALL",
    "action": "HIDE_DELIVERY_OPTION",
    "actionValue": "Express"
  },
  "input": {
    "hasAnyProductTag": ["heavy", "oversized"],
    "hasProductTags": [],
    "hasCustomerTags": [],
    "lineAttributeKey": "BLANK"
  }
}
```

### Pattern: Hide Delivery Option by Region

```json
{
  "type": "ADDRESS",
  "configuration": {
    "conditions": [
      {
        "property": "ADDRESS_PROVINCE_CODE",
        "operator": "ONE_OF",
        "value": { "value": ["HI", "AK"] }
      }
    ],
    "mode": "ALL",
    "action": "HIDE_DELIVERY_OPTION",
    "actionValue": "Same Day"
  },
  "input": { "hasProductTags": [], "cartAttributeKey": "BLANK" }
}
```

### Pattern: Hide Delivery Option by Time

```json
{
  "type": "TIME",
  "configuration": {
    "conditions": [
      {
        "property": "TIME",
        "operator": "TIME_AFTER",
        "value": { "value": "14:00" }
      }
    ],
    "mode": "ALL",
    "action": "HIDE_DELIVERY_OPTION",
    "actionValue": "Next Day"
  },
  "input": { "timeAfter": "14:00:00", "timeBefore": "00:00:00" }
}
```

---

## actionValue Matching (Payment & Delivery)

For both payment customization and delivery customization, the `actionValue` is matched against target names using 3 levels:

1. **Exact match** - Full name/title matches exactly
2. **Substring match** - actionValue found within the name/title
3. **Regex match** - actionValue is treated as a regex pattern

**Examples:**
- `"Cash on Delivery"` → exact match only
- `"COD"` → substring match, finds "Cash on Delivery (COD)"
- `"(?i)express"` → regex, case-insensitive match for any option containing "express"
- `".*"` → regex, matches ALL options (use with caution)

---

## Common Mistakes for Payment/Delivery Rules

### 1. Using `msg` Instead of `action`/`actionValue`

**WRONG (for payment/delivery):**
```json
{
  "conditions": [...],
  "msg": { "en": "Not available" },
  "mode": "ALL"
}
```

**CORRECT (for payment):**
```json
{
  "conditions": [...],
  "mode": "ALL",
  "action": "HIDE_PAYMENT_METHOD",
  "actionValue": "Cash on Delivery"
}
```

**CORRECT (for delivery):**
```json
{
  "conditions": [...],
  "mode": "ALL",
  "action": "HIDE_DELIVERY_OPTION",
  "actionValue": "Express Shipping"
}
```

### 2. Mixing Up Action Values

- Payment rules: `"HIDE_PAYMENT_METHOD"` (not `"HIDE_DELIVERY_OPTION"`)
- Delivery rules: `"HIDE_DELIVERY_OPTION"` (not `"HIDE_PAYMENT_METHOD"`)

### 3. Empty actionValue

Always provide the target name/pattern in `actionValue`. An empty `actionValue` won't match anything.

---

## Examples Reference

See `examples/boomgate.Template.json` and `examples/boomgate.StoreExamples.json` for real-world rule configurations.

For payment customization examples, see [Payment Customization](./11-payment-customization.md#examples).
For delivery customization examples, see [Delivery Customization](./12-delivery-customization.md#examples).

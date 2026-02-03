# AI Agent Guide

Guide for AI agents building BoomGate validation rules.

---

## Overview

When building validation rules for BoomGate, follow these guidelines to create correct, efficient rules.

---

## Rule Building Process

### 1. Understand the Requirement

Parse the user's request to identify:
- **What** should be blocked/allowed
- **Who** it applies to (customers, products, locations)
- **When** it should apply (always, time-based)
- **Error message** to display

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

### 6. Write Error Message

- Clear explanation of why blocked
- Include placeholders for dynamic content: `[product_title]`
- Add translations if needed

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

## Validation Checklist

Before finalizing a rule, verify:

- [ ] Validation type matches conditions used
- [ ] All operators are compatible with their properties
- [ ] Input contains all tags referenced in conditions
- [ ] Input contains metafield namespace/key if used
- [ ] Input contains attribute keys if used
- [ ] Mode (ALL/ANY) is correct for the logic
- [ ] Error message is clear and helpful
- [ ] Value formats are correct (dates, numbers, arrays)

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

## Examples Reference

See `examples/boomgate.Template.json` and `examples/boomgate.StoreExamples.json` for real-world rule configurations.

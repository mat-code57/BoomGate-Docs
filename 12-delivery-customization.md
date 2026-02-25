# Delivery Customization Rules

Hide delivery/shipping options at checkout based on conditions. Uses the same condition engine as checkout validation rules.

---

## Overview

Delivery customization rules allow you to **hide specific delivery options** when conditions are met. For example, hide "Express Shipping" for heavy items, or hide "Local Pickup" for customers outside a certain region.

- **Shopify Function:** `cart.delivery-options.transform.run`
- **Metafield Namespace:** `$app:delivery-customization`
- **Action:** `HIDE_DELIVERY_OPTION`

---

## How It Works

1. When a customer reaches the shipping step at checkout, Shopify calls the delivery customization function
2. The function evaluates all conditions in the rule
3. If conditions match (based on ALL/ANY mode), the function returns `DeliveryOptionHide` operations
4. The specified delivery options are hidden from the checkout

**Matching Logic:** The `actionValue` (delivery option title) is matched against each available delivery option using three levels:
1. **Exact match** - Full title matches exactly
2. **Substring match** - The actionValue is found within the delivery option title
3. **Regex match** - The actionValue is treated as a regex pattern

This means you can use partial names or regex patterns in `actionValue`.

---

## Rule Structure

```typescript
interface DeliveryCustomizationRule {
  title: string;                        // Human-readable rule name
  type: ValidationType;                 // Same 11 types as checkout validation
  configuration: {
    conditions: Condition[];            // Same conditions as checkout validation
    mode: "ALL" | "ANY";               // Same condition modes
    action: "HIDE_DELIVERY_OPTION";    // Always this value
    actionValue: string;               // Delivery option title/pattern to hide
  };
  input: Input;                         // Same input config as checkout validation
}
```

### Key Differences from Checkout Validation

| Aspect | Checkout Validation | Delivery Customization |
|--------|--------------------|-----------------------|
| Action | Block checkout + show error message | Hide a delivery option |
| Configuration field | `msg` (localized messages) | `action` + `actionValue` |
| Metafield namespace | `$app:checkout-validation` | `$app:delivery-customization` |
| Shopify resource | `ValidationCustomization` | `DeliveryCustomization` |
| GID prefix | `gid://shopify/ValidationCustomization/` | `gid://shopify/DeliveryCustomization/` |

### Key Differences from Payment Customization

| Aspect | Payment Customization | Delivery Customization |
|--------|----------------------|-----------------------|
| Action value | `HIDE_PAYMENT_METHOD` | `HIDE_DELIVERY_OPTION` |
| Target | Payment method name | Delivery option title |
| Metafield namespace | `$app:payment-customization` | `$app:delivery-customization` |
| Shopify resource | `PaymentCustomization` | `DeliveryCustomization` |

### What Stays the Same (Across All 3 Rule Types)

- All 11 validation types (GENERAL, PRODUCT, CUSTOMER, CART, etc.)
- All 60+ condition properties
- All 50+ operators
- Input configuration (tags, metafields, attributes, time)
- Condition modes (ALL/ANY)
- AND conditions for nested line-level logic
- Property-type compatibility rules

---

## Configuration Schema

```json
{
  "conditions": [
    {
      "property": "PROPERTY_NAME",
      "operator": "OPERATOR_NAME",
      "value": { "value": "..." },
      "andConditions": []
    }
  ],
  "mode": "ALL",
  "action": "HIDE_DELIVERY_OPTION",
  "actionValue": "Express Shipping"
}
```

### Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `conditions` | Condition[] | Yes | Array of conditions (same as checkout validation) |
| `mode` | "ALL" \| "ANY" | Yes | How conditions combine |
| `action` | string | Yes | Always `"HIDE_DELIVERY_OPTION"` |
| `actionValue` | string | Yes | Delivery option title/pattern to hide |

---

## Finding Delivery Option Names

To find the exact delivery option titles available in your store:

1. Go to **Shopify Admin > Settings > Shipping and delivery**
2. Check your shipping profiles and their rate names
3. Use those exact rate names in `actionValue`

Common delivery option names:
- `Standard Shipping`
- `Express Shipping`
- `Economy Shipping`
- `Local Pickup`
- `Free Shipping`
- `Next Day Delivery`
- `Same Day Delivery`

**Tip:** Since matching uses substring and regex, you can use partial names. For example, `"Express"` will match `"Express Shipping"` and `"Express Delivery"`.

**Note:** Delivery option names come from your shipping profiles in Shopify Settings. They may also come from third-party carrier services. The names you see in checkout are the names you should use.

---

## Examples

### Hide Express Shipping for Heavy Items

Hide express shipping when cart contains products tagged "heavy".

**Type:** TAGS

```json
{
  "title": "Hide express shipping for heavy items",
  "type": "TAGS",
  "configuration": {
    "conditions": [
      {
        "property": "PRODUCT_TAGS",
        "operator": "HAS_ANY_TAG",
        "value": { "value": ["heavy", "oversized"] },
        "andConditions": []
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

### Hide Local Pickup for International Customers

Hide local pickup option for customers outside the US.

**Type:** ADDRESS

```json
{
  "title": "Hide local pickup for non-US customers",
  "type": "ADDRESS",
  "configuration": {
    "conditions": [
      {
        "property": "ADDRESS_COUNTRY_CODE",
        "operator": "DOES_NOT_EQUAL",
        "value": { "value": "US" },
        "andConditions": []
      }
    ],
    "mode": "ALL",
    "action": "HIDE_DELIVERY_OPTION",
    "actionValue": "Local Pickup"
  },
  "input": {
    "hasProductTags": [],
    "cartAttributeKey": "BLANK"
  }
}
```

### Hide Free Shipping for Low-Value Orders

Hide free shipping when cart total is under $50.

**Type:** CART

```json
{
  "title": "Hide free shipping for orders under $50",
  "type": "CART",
  "configuration": {
    "conditions": [
      {
        "property": "CART_TOTAL",
        "operator": "LESS_THAN",
        "value": { "value": "50" },
        "andConditions": []
      }
    ],
    "mode": "ALL",
    "action": "HIDE_DELIVERY_OPTION",
    "actionValue": "Free Shipping"
  },
  "input": {
    "lineAttributeKey": "BLANK",
    "cartAttributeKey": "BLANK"
  }
}
```

### Hide Shipping for Digital Products

Hide all shipping options when all products are digital (gift cards).

**Type:** GENERAL

```json
{
  "title": "Hide shipping for gift card only orders",
  "type": "GENERAL",
  "configuration": {
    "conditions": [
      {
        "property": "PRODUCT_IS_GIFT_CARD",
        "operator": "EQUALS",
        "value": { "value": "true" },
        "andConditions": []
      }
    ],
    "mode": "ALL",
    "action": "HIDE_DELIVERY_OPTION",
    "actionValue": ".*"
  },
  "input": {}
}
```

**Note:** Using `".*"` as regex will match ALL delivery options.

### Hide Delivery Option for Non-B2B Customers

Hide "Net 30 Shipping" for customers without "b2b" tag.

**Type:** CUSTOMER

```json
{
  "title": "Hide Net 30 for non-B2B",
  "type": "CUSTOMER",
  "configuration": {
    "conditions": [
      {
        "property": "CUSTOMER_TAGS",
        "operator": "DOES_NOT_HAVE_TAGS",
        "value": { "value": ["b2b", "wholesale"] },
        "andConditions": []
      }
    ],
    "mode": "ALL",
    "action": "HIDE_DELIVERY_OPTION",
    "actionValue": "Net 30"
  },
  "input": {
    "hasCustomerTags": ["b2b", "wholesale"],
    "customerMetafieldNamespace": "BLANK",
    "customerMetafieldKey": "BLANK"
  }
}
```

### Hide Delivery Option by Region

Hide "Same Day Delivery" for certain states.

**Type:** ADDRESS

```json
{
  "title": "No same-day delivery for remote states",
  "type": "ADDRESS",
  "configuration": {
    "conditions": [
      {
        "property": "ADDRESS_PROVINCE_CODE",
        "operator": "ONE_OF",
        "value": { "value": ["HI", "AK", "PR", "GU"] },
        "andConditions": []
      }
    ],
    "mode": "ALL",
    "action": "HIDE_DELIVERY_OPTION",
    "actionValue": "Same Day"
  },
  "input": {
    "hasProductTags": [],
    "cartAttributeKey": "BLANK"
  }
}
```

### Hide Delivery Option During Off-Hours

Hide "Next Day Delivery" after business hours.

**Type:** TIME

```json
{
  "title": "Hide next day delivery after 2 PM",
  "type": "TIME",
  "configuration": {
    "conditions": [
      {
        "property": "TIME",
        "operator": "TIME_AFTER",
        "value": { "value": "14:00" },
        "andConditions": []
      }
    ],
    "mode": "ALL",
    "action": "HIDE_DELIVERY_OPTION",
    "actionValue": "Next Day"
  },
  "input": {
    "timeAfter": "14:00:00",
    "timeBefore": "00:00:00"
  }
}
```

### Hide Delivery Option When Discount Applied

Hide free shipping when a discount code is used (Shopify Plus).

**Type:** REDUCTION

```json
{
  "title": "Hide free shipping when discount applied",
  "type": "REDUCTION",
  "configuration": {
    "conditions": [
      {
        "property": "APPLIED_DISCOUNT_CODES_COUNT",
        "operator": "GREATHER_THAN",
        "value": { "count": "0" },
        "andConditions": []
      }
    ],
    "mode": "ALL",
    "action": "HIDE_DELIVERY_OPTION",
    "actionValue": "Free Shipping"
  },
  "input": {}
}
```

### Hide Delivery Options Using Regex

Hide all delivery options containing "pickup" (case-insensitive).

**Type:** GENERAL

```json
{
  "title": "Hide all pickup options for tagged products",
  "type": "GENERAL",
  "configuration": {
    "conditions": [
      {
        "property": "PRODUCT_TYPE",
        "operator": "EQUALS",
        "value": { "value": "Hazardous" },
        "andConditions": []
      }
    ],
    "mode": "ALL",
    "action": "HIDE_DELIVERY_OPTION",
    "actionValue": "(?i)pickup"
  },
  "input": {}
}
```

---

## AI Agent Guide for Delivery Rules

### Step-by-Step Process

1. **Understand what delivery option to hide** - Get the exact title or pattern
2. **Understand the conditions** - When should it be hidden?
3. **Select validation type** - Based on what data the conditions need (same rules as checkout validation)
4. **Build conditions** - Same as checkout validation
5. **Set action** - Always `"HIDE_DELIVERY_OPTION"`
6. **Set actionValue** - The delivery option title/pattern
7. **Configure input** - Same input sync rules as checkout validation

### CRITICAL: No Error Message

Delivery customization rules do NOT have a `msg` field. Instead they have `action` and `actionValue`. The delivery option is simply hidden from the customer with no message shown.

**WRONG:**
```json
{
  "conditions": [...],
  "msg": { "en": "This delivery option is not available" },
  "mode": "ALL"
}
```

**CORRECT:**
```json
{
  "conditions": [...],
  "mode": "ALL",
  "action": "HIDE_DELIVERY_OPTION",
  "actionValue": "Express Shipping"
}
```

### actionValue Matching

The `actionValue` is matched against delivery option titles using 3 levels:

1. **Exact** - `"Express Shipping"` matches only that exact title
2. **Substring** - `"Express"` matches any delivery option containing "Express"
3. **Regex** - `"(?i)express.*ship"` matches case-insensitive

**Tip:** Use the simplest match that works. Delivery option titles come from your Shopify shipping profiles.

### Validation Checklist for Delivery Rules

- [ ] `action` is set to `"HIDE_DELIVERY_OPTION"`
- [ ] `actionValue` contains the delivery option title/pattern
- [ ] No `msg` field in configuration (it's NOT checkout validation)
- [ ] Validation type matches conditions used (same rules as checkout validation)
- [ ] All operators are compatible with their properties
- [ ] Input contains all tags/metafields/attributes referenced in conditions
- [ ] Mode (ALL/ANY) is correct for the logic

---

## API Reference

### Creating a Delivery Customization Rule

```graphql
mutation deliveryCustomizationCreate($deliveryCustomization: DeliveryCustomizationInput!) {
  deliveryCustomizationCreate(deliveryCustomization: $deliveryCustomization) {
    userErrors { field message }
    deliveryCustomization { id title }
  }
}
```

Variables:
```json
{
  "deliveryCustomization": {
    "enabled": false,
    "functionId": "<function-id-for-type>",
    "title": "Rule title",
    "metafields": [
      {
        "key": "configuration",
        "namespace": "$app:delivery-customization",
        "type": "json",
        "value": "{\"conditions\":[...],\"mode\":\"ALL\",\"action\":\"HIDE_DELIVERY_OPTION\",\"actionValue\":\"...\"}"
      },
      {
        "key": "input",
        "namespace": "$app:delivery-customization",
        "type": "json",
        "value": "{...input configuration...}"
      }
    ]
  }
}
```

### Updating a Delivery Customization Rule

```graphql
mutation deliveryCustomizationUpdate($id: ID!, $deliveryCustomization: DeliveryCustomizationInput!) {
  deliveryCustomizationUpdate(id: $id, deliveryCustomization: $deliveryCustomization) {
    userErrors { field message }
    deliveryCustomization { id title enabled }
  }
}
```

### Deleting a Delivery Customization Rule

```graphql
mutation deliveryCustomizationDelete($id: ID!) {
  deliveryCustomizationDelete(id: $id) {
    deletedId
    userErrors { field message }
  }
}
```

### Function ID Mapping

| Validation Type | Extension Handle |
|-----------------|-----------------|
| GENERAL | `delivery-customization-logic` |
| PRODUCT | `product-delivery-customization` |
| CUSTOMER | `customer-delivery-customization` |
| CART | `cart-delivery-customization` |
| TAGS | `tags-delivery-customization` |
| TIME | `time-delivery-customization` |
| ADDRESS | `address-delivery-customization` |
| SHIPPING | `shipping-delivery-customization` |
| REDUCTION | `reduction-delivery-customization` |
| BILLING | `billing-address-delivery-customization` |
| ADDRESS_PRODUCT | `address-product-delivery-customization` |

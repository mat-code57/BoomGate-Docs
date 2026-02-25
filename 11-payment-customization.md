# Payment Customization Rules

Hide payment methods at checkout based on conditions. Uses the same condition engine as checkout validation rules.

---

## Overview

Payment customization rules allow you to **hide specific payment methods** when conditions are met. For example, hide "Cash on Delivery" for international orders, or hide "Invoice" for non-B2B customers.

- **Shopify Function:** `cart.payment-methods.transform.run`
- **Metafield Namespace:** `$app:payment-customization`
- **Action:** `HIDE_PAYMENT_METHOD`

---

## How It Works

1. When a customer reaches checkout, Shopify calls the payment customization function
2. The function evaluates all conditions in the rule
3. If conditions match (based on ALL/ANY mode), the function returns a `HidePaymentMethod` operation
4. The specified payment method is hidden from the checkout

**Matching Logic:** The `actionValue` (payment method name) is matched against each available payment method using three levels:
1. **Exact match** - Full name matches exactly
2. **Substring match** - The actionValue is found within the payment method name
3. **Regex match** - The actionValue is treated as a regex pattern

This means you can use partial names or regex patterns in `actionValue`.

---

## Rule Structure

```typescript
interface PaymentCustomizationRule {
  title: string;                      // Human-readable rule name
  type: ValidationType;               // Same 11 types as checkout validation
  configuration: {
    conditions: Condition[];          // Same conditions as checkout validation
    mode: "ALL" | "ANY";             // Same condition modes
    action: "HIDE_PAYMENT_METHOD";   // Always this value
    actionValue: string;             // Payment method name/pattern to hide
  };
  input: Input;                       // Same input config as checkout validation
}
```

### Key Differences from Checkout Validation

| Aspect | Checkout Validation | Payment Customization |
|--------|--------------------|-----------------------|
| Action | Block checkout + show error message | Hide a payment method |
| Configuration field | `msg` (localized messages) | `action` + `actionValue` |
| Metafield namespace | `$app:checkout-validation` | `$app:payment-customization` |
| Shopify resource | `ValidationCustomization` | `PaymentCustomization` |
| GID prefix | `gid://shopify/ValidationCustomization/` | `gid://shopify/PaymentCustomization/` |

### What Stays the Same

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
  "action": "HIDE_PAYMENT_METHOD",
  "actionValue": "Cash on Delivery"
}
```

### Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `conditions` | Condition[] | Yes | Array of conditions (same as checkout validation) |
| `mode` | "ALL" \| "ANY" | Yes | How conditions combine |
| `action` | string | Yes | Always `"HIDE_PAYMENT_METHOD"` |
| `actionValue` | string | Yes | Payment method name/pattern to hide |

---

## Finding Payment Method Names

To find the exact payment method names available in your store:

1. Go to **Shopify Admin > Settings > Payments**
2. Note the exact names of your payment providers
3. Use those names in `actionValue`

Common payment method names:
- `Cash on Delivery (COD)`
- `Bank Deposit`
- `PayPal Express`
- `Shopify Payments`
- `Manual Payment`

**Tip:** Since matching uses substring and regex, you can use partial names. For example, `"COD"` will match `"Cash on Delivery (COD)"`.

---

## Examples

### Hide COD for International Orders

Hide Cash on Delivery for orders shipping outside the US.

**Type:** ADDRESS

```json
{
  "title": "Hide COD for international orders",
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
    "action": "HIDE_PAYMENT_METHOD",
    "actionValue": "Cash on Delivery"
  },
  "input": {
    "hasProductTags": [],
    "cartAttributeKey": "BLANK"
  }
}
```

### Hide Invoice for Non-B2B Customers

Only show Invoice payment for customers tagged as "b2b".

**Type:** CUSTOMER

```json
{
  "title": "Hide Invoice for non-B2B customers",
  "type": "CUSTOMER",
  "configuration": {
    "conditions": [
      {
        "property": "CUSTOMER_TAGS",
        "operator": "DOES_NOT_HAVE_TAGS",
        "value": { "value": ["b2b"] },
        "andConditions": []
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

### Hide PayPal for High-Value Orders

Hide PayPal for orders over $5,000.

**Type:** CART

```json
{
  "title": "Hide PayPal for orders over $5000",
  "type": "CART",
  "configuration": {
    "conditions": [
      {
        "property": "CART_TOTAL",
        "operator": "GREATHER_THAN",
        "value": { "value": "5000" },
        "andConditions": []
      }
    ],
    "mode": "ALL",
    "action": "HIDE_PAYMENT_METHOD",
    "actionValue": "PayPal"
  },
  "input": {
    "lineAttributeKey": "BLANK",
    "cartAttributeKey": "BLANK"
  }
}
```

### Hide Payment Method for Specific Products

Hide COD when cart contains products tagged "no-cod".

**Type:** TAGS

```json
{
  "title": "Hide COD for no-cod products",
  "type": "TAGS",
  "configuration": {
    "conditions": [
      {
        "property": "PRODUCT_TAGS",
        "operator": "HAS_ANY_TAG",
        "value": { "value": ["no-cod"] },
        "andConditions": []
      }
    ],
    "mode": "ALL",
    "action": "HIDE_PAYMENT_METHOD",
    "actionValue": "Cash on Delivery"
  },
  "input": {
    "hasAnyProductTag": ["no-cod"],
    "hasProductTags": [],
    "hasCustomerTags": [],
    "lineAttributeKey": "BLANK"
  }
}
```

### Hide Payment Method During Off-Hours

Hide Bank Transfer outside business hours.

**Type:** TIME

```json
{
  "title": "Hide Bank Transfer after 5 PM",
  "type": "TIME",
  "configuration": {
    "conditions": [
      {
        "property": "TIME",
        "operator": "TIME_AFTER",
        "value": { "value": "17:00" },
        "andConditions": []
      }
    ],
    "mode": "ALL",
    "action": "HIDE_PAYMENT_METHOD",
    "actionValue": "Bank Transfer"
  },
  "input": {
    "timeAfter": "17:00:00",
    "timeBefore": "00:00:00"
  }
}
```

### Hide Payment Method for Guest Checkout

Hide certain payment methods when customer is not logged in.

**Type:** GENERAL

```json
{
  "title": "Hide Store Credit for guests",
  "type": "GENERAL",
  "configuration": {
    "conditions": [
      {
        "property": "CUSTOMER_IS_AUTHENTICATED",
        "operator": "EQUALS",
        "value": { "value": "false" },
        "andConditions": []
      }
    ],
    "mode": "ALL",
    "action": "HIDE_PAYMENT_METHOD",
    "actionValue": "Store Credit"
  },
  "input": {}
}
```

### Hide Payment Method Using Regex

Hide all payment methods containing "manual" (case-insensitive via regex).

**Type:** GENERAL

```json
{
  "title": "Hide manual payment methods for small orders",
  "type": "GENERAL",
  "configuration": {
    "conditions": [
      {
        "property": "CART_TOTAL",
        "operator": "LESS_THAN",
        "value": { "value": "100" },
        "andConditions": []
      }
    ],
    "mode": "ALL",
    "action": "HIDE_PAYMENT_METHOD",
    "actionValue": "(?i)manual"
  },
  "input": {}
}
```

---

## AI Agent Guide for Payment Rules

### Step-by-Step Process

1. **Understand what payment method to hide** - Get the exact name or pattern
2. **Understand the conditions** - When should it be hidden?
3. **Select validation type** - Based on what data the conditions need (same rules as checkout validation)
4. **Build conditions** - Same as checkout validation
5. **Set action** - Always `"HIDE_PAYMENT_METHOD"`
6. **Set actionValue** - The payment method name/pattern
7. **Configure input** - Same input sync rules as checkout validation

### CRITICAL: No Error Message

Payment customization rules do NOT have a `msg` field. Instead they have `action` and `actionValue`. The payment method is simply hidden from the customer with no message shown.

**WRONG:**
```json
{
  "conditions": [...],
  "msg": { "en": "This payment method is not available" },
  "mode": "ALL"
}
```

**CORRECT:**
```json
{
  "conditions": [...],
  "mode": "ALL",
  "action": "HIDE_PAYMENT_METHOD",
  "actionValue": "Cash on Delivery"
}
```

### actionValue Matching

The `actionValue` is matched against payment method names using 3 levels:

1. **Exact** - `"Cash on Delivery (COD)"` matches only that exact name
2. **Substring** - `"COD"` matches any payment method containing "COD"
3. **Regex** - `"(?i)cash.*delivery"` matches case-insensitive

**Tip:** Use the simplest match that works. If the payment method name is unique, use the exact name or a distinctive substring.

### Validation Checklist for Payment Rules

- [ ] `action` is set to `"HIDE_PAYMENT_METHOD"`
- [ ] `actionValue` contains the payment method name/pattern
- [ ] No `msg` field in configuration (it's NOT checkout validation)
- [ ] Validation type matches conditions used (same rules as checkout validation)
- [ ] All operators are compatible with their properties
- [ ] Input contains all tags/metafields/attributes referenced in conditions
- [ ] Mode (ALL/ANY) is correct for the logic

---

## API Reference

### Creating a Payment Customization Rule

```graphql
mutation paymentCustomizationCreate($paymentCustomization: PaymentCustomizationInput!) {
  paymentCustomizationCreate(paymentCustomization: $paymentCustomization) {
    userErrors { field message }
    paymentCustomization { id title }
  }
}
```

Variables:
```json
{
  "paymentCustomization": {
    "enabled": false,
    "functionId": "<function-id-for-type>",
    "title": "Rule title",
    "metafields": [
      {
        "key": "configuration",
        "namespace": "$app:payment-customization",
        "type": "json",
        "value": "{\"conditions\":[...],\"mode\":\"ALL\",\"action\":\"HIDE_PAYMENT_METHOD\",\"actionValue\":\"...\"}"
      },
      {
        "key": "input",
        "namespace": "$app:payment-customization",
        "type": "json",
        "value": "{...input configuration...}"
      }
    ]
  }
}
```

### Updating a Payment Customization Rule

```graphql
mutation paymentCustomizationUpdate($id: ID!, $paymentCustomization: PaymentCustomizationInput!) {
  paymentCustomizationUpdate(id: $id, paymentCustomization: $paymentCustomization) {
    userErrors { field message }
    paymentCustomization { id title enabled }
  }
}
```

### Deleting a Payment Customization Rule

```graphql
mutation paymentCustomizationDelete($id: ID!) {
  paymentCustomizationDelete(id: $id) {
    deletedId
    userErrors { field message }
  }
}
```

### Function ID Mapping

| Validation Type | Extension Handle |
|-----------------|-----------------|
| GENERAL | `payment-customization-logic` |
| PRODUCT | `product-payment-customization` |
| CUSTOMER | `customer-payment-customization` |
| CART | `cart-payment-customization` |
| TAGS | `tags-payment-customization` |
| TIME | `time-payment-customization` |
| ADDRESS | `address-payment-customization` |
| SHIPPING | `shipping-payment-customization` |
| REDUCTION | `reduction-payment-customization` |
| BILLING | `billing-address-payment-customization` |
| ADDRESS_PRODUCT | `address-product-payment-customization` |

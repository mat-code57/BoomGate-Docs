# BoomGate Documentation

Comprehensive documentation for BoomGate - Checkout Rules for Shopify.

## Table of Contents

### Getting Started

1. [Introduction](./01-introduction.md)
   - Overview
   - Rule Categories (Validation, Payment, Delivery)
   - Key Features
   - How It Works
   - Use Cases

### Core Concepts

2. [Validation Types](./02-validation-types.md)
   - GENERAL, PRODUCT, CUSTOMER, CART
   - TAGS, TIME, ADDRESS, SHIPPING
   - REDUCTION, BILLING (Shopify Plus)
   - ADDRESS_PRODUCT

3. [Condition Properties](./03-condition-properties.md)
   - Product Properties
   - Line Item Properties
   - Cart Properties
   - Customer Properties
   - Address Properties
   - Time Properties
   - Shipping Properties
   - Reduction Properties

4. [Operators](./04-operators.md)
   - String Operators
   - Numeric Operators
   - Tag Operators
   - Date/Time Operators
   - Metafield Operators
   - Attribute Operators

### Building Rules

5. [Building Rules](./05-building-rules.md)
   - Rule Structure
   - Condition Modes (ALL/ANY)
   - AND Conditions
   - Error Messages
   - Input Configuration
   - Best Practices

6. [Examples](./06-examples.md)
   - Quantity Limits
   - Shipping Restrictions
   - Customer Restrictions
   - Time-Based Rules
   - Cart Rules
   - Tag Combinations
   - Shipping Method Rules
   - Discount Rules

### Technical Reference

7. [API Reference](./07-api-reference.md)
   - Rule Schema (all 3 categories)
   - Validation Types
   - Operators Reference
   - Properties Reference
   - Input Configuration
   - Value Formats

8. [Troubleshooting](./08-troubleshooting.md)
   - Common Issues
   - Debugging Steps
   - Error Messages
   - Getting Help

### For AI Agents

9. [AI Agent Guide](./09-ai-agent-guide.md)
   - Rule Building Process
   - Common Patterns
   - Property-Operator Compatibility
   - Input Synchronization
   - Validation Checklist
   - Common Mistakes

10. [Limitations](./10-limitations.md)
    - What BoomGate Cannot Do
    - Shopify Plus Requirements
    - Property-Type Restrictions

### Payment & Delivery Rules

11. [Payment Customization](./11-payment-customization.md)
    - Hide Payment Methods
    - Configuration Schema
    - actionValue Matching
    - Examples
    - AI Agent Guide for Payment Rules
    - API Reference

12. [Delivery Customization](./12-delivery-customization.md)
    - Hide Delivery Options
    - Configuration Schema
    - actionValue Matching
    - Examples
    - AI Agent Guide for Delivery Rules
    - API Reference

---

## Quick Start

### 1. Choose a Rule Category

- **Block checkout?** → Checkout Validation
- **Hide a payment method?** → Payment Customization ([guide](./11-payment-customization.md))
- **Hide a delivery option?** → Delivery Customization ([guide](./12-delivery-customization.md))

### 2. Choose a Validation Type

Based on what data you need:
- Products → PRODUCT
- Customers → CUSTOMER
- Addresses → ADDRESS
- Time → TIME

### 3. Define Conditions

Each condition has:
- **Property**: What to check
- **Operator**: How to compare
- **Value**: What to compare against

### 4. Set the Action

- **Checkout Validation:** Error message (`msg`)
- **Payment Customization:** `action: "HIDE_PAYMENT_METHOD"` + `actionValue`
- **Delivery Customization:** `action: "HIDE_DELIVERY_OPTION"` + `actionValue`

### 5. Enable the Rule

Rules must be enabled to take effect.

---

## Example: Checkout Validation Rule

Block orders over 5 items for non-wholesale customers:

```json
{
  "type": "CUSTOMER",
  "configuration": {
    "conditions": [
      {
        "property": "CUSTOMER_TAGS",
        "operator": "DOES_NOT_HAVE_TAGS",
        "value": { "value": ["wholesale"] }
      },
      {
        "property": "LINE_QUANTITY",
        "operator": "GREATHER_THAN",
        "value": { "value": "5" }
      }
    ],
    "msg": { "en": "Only wholesale customers can order more than 5 items" },
    "mode": "ALL"
  },
  "input": {
    "hasCustomerTags": ["wholesale"]
  }
}
```

## Example: Payment Customization Rule

Hide COD for international orders:

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
  "input": {
    "hasProductTags": [],
    "cartAttributeKey": "BLANK"
  }
}
```

## Example: Delivery Customization Rule

Hide express shipping for heavy products:

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

---

## Support

- Email: support@code57.pl
- Documentation: https://docs.boomgate.app

---

## Shopify Plus Features

Some features require Shopify Plus:
- **REDUCTION** - Gift card and discount validation
- **BILLING** - Billing address validation

---

## Version

Documentation version: 2.0
Last updated: 2025

# BoomGate Documentation

Comprehensive documentation for BoomGate - Checkout Validation for Shopify.

## Table of Contents

### Getting Started

1. [Introduction](./01-introduction.md)
   - Overview
   - Key Features
   - How It Works
   - Use Cases

### Core Concepts

2. [Validation Types](./02-validation-types.md)
   - GENERAL, PRODUCT, CUSTOMER, CART
   - TAGS, TIME, ADDRESS, SHIPPING
   - REDUCTION, BILLING (Shopify Plus)

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
   - Rule Schema
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

---

## Quick Start

### 1. Choose a Validation Type

Based on what you want to validate:
- Products → PRODUCT
- Customers → CUSTOMER
- Addresses → ADDRESS
- Time → TIME

### 2. Define Conditions

Each condition has:
- **Property**: What to check
- **Operator**: How to compare
- **Value**: What to compare against

### 3. Set Error Message

Clear message explaining why checkout is blocked.

### 4. Enable the Rule

Rules must be enabled to take effect.

---

## Example Rule

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

Documentation version: 1.0
Last updated: 2024

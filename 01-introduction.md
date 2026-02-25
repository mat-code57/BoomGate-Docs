# BoomGate - Checkout Rules for Shopify

## Overview

BoomGate is a powerful Shopify app that allows merchants to create custom checkout rules. It supports three types of rules:

1. **Checkout Validations** - Block checkout when conditions are met
2. **Payment Customizations** - Hide payment methods when conditions are met
3. **Delivery Customizations** - Hide delivery/shipping options when conditions are met

All three rule types share the same condition engine: same 11 validation types, same 60+ properties, same 50+ operators.

## Key Features

- **3 Rule Categories** - Checkout validation, payment customization, delivery customization
- **11 Validation Types** - Target specific aspects of checkout (products, customers, cart, addresses, time, etc.)
- **50+ Operators** - Flexible comparison logic (equals, contains, regex, numeric comparisons, etc.)
- **60+ Condition Properties** - Check virtually any aspect of the cart, customer, or checkout
- **Nested AND Conditions** - Create complex multi-level rule logic
- **Multi-Language Messages** - Display error messages in customer's language (checkout validation only)
- **Real-Time Execution** - Rules execute instantly at checkout via Shopify Functions

## Rule Categories

### Checkout Validations (Block Checkout)

- **Shopify Function:** `cart.validations.generate.run`
- **Action:** Block checkout and display an error message
- **Use cases:** Quantity limits, shipping restrictions, customer restrictions, time-based blocks

### Payment Customizations (Hide Payment Methods)

- **Shopify Function:** `cart.payment-methods.transform.run`
- **Action:** Hide specific payment methods by name
- **Use cases:** Hide COD for certain regions, restrict payment methods by customer tag, hide payment options based on cart contents
- **See:** [Payment Customization Guide](./11-payment-customization.md)

### Delivery Customizations (Hide Delivery Options)

- **Shopify Function:** `cart.delivery-options.transform.run`
- **Action:** Hide specific delivery/shipping options by title
- **Use cases:** Hide express shipping for heavy items, restrict delivery options by region, hide pickup for certain products
- **See:** [Delivery Customization Guide](./12-delivery-customization.md)

## How It Works

1. **Create a Rule** - Define conditions and choose an action
2. **Configure the Action** - Set error message (validation) or target name (payment/delivery)
3. **Enable the Rule** - Activate on your store
4. **Automatic Enforcement** - Shopify Functions execute the rule at every checkout

## Use Cases

| Use Case | Rule Category | Validation Type |
|----------|---------------|-----------------|
| Limit quantity per customer | Checkout Validation | CUSTOMER, PRODUCT |
| Restrict shipping to certain regions | Checkout Validation | ADDRESS |
| Block checkout for specific product/customer combos | Checkout Validation | TAGS |
| Time-limited sales | Checkout Validation | TIME |
| Minimum order values | Checkout Validation | CART |
| Hide COD for international orders | Payment Customization | ADDRESS |
| Hide payment method for guest checkout | Payment Customization | CUSTOMER |
| Hide express shipping for heavy items | Delivery Customization | PRODUCT |
| Hide local pickup for tagged products | Delivery Customization | TAGS |
| Hide delivery option during off-hours | Delivery Customization | TIME |

## Shopify Plus Features

Some features require Shopify Plus:

- **REDUCTION** - Gift card and discount code validation
- **BILLING** - Billing address validation

## Architecture

BoomGate uses Shopify Functions to execute rules in real-time:

```
Customer Checkout → Shopify Functions → BoomGate Rule Engine → Action
                                                              ├── Block Checkout (validation)
                                                              ├── Hide Payment Method (payment)
                                                              └── Hide Delivery Option (delivery)
```

Rules are stored as metafields on Shopify resources, ensuring fast execution without external API calls.

## Shared Condition Engine

All three rule categories use the **same condition engine**:

- **Same 11 validation types:** GENERAL, PRODUCT, CUSTOMER, CART, TAGS, TIME, ADDRESS, SHIPPING, REDUCTION, BILLING, ADDRESS_PRODUCT
- **Same properties and operators** per type
- **Same input configuration** for tags, metafields, attributes
- **Same condition modes:** ALL / ANY
- **Same AND conditions** for nested line-level logic

The only difference is the **action**:
- Checkout Validation: `msg` (error message displayed to customer)
- Payment Customization: `action: "HIDE_PAYMENT_METHOD"` + `actionValue` (payment method name to hide)
- Delivery Customization: `action: "HIDE_DELIVERY_OPTION"` + `actionValue` (delivery option title to hide)

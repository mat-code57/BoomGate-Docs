# BoomGate - Checkout Validation for Shopify

## Overview

BoomGate is a powerful Shopify app that allows merchants to create custom checkout validation rules. Block or allow checkout based on products, customers, cart contents, shipping addresses, time constraints, and more. 1234

## Key Features

- **9 Validation Types** - Target specific aspects of checkout (products, customers, cart, addresses, time, etc.)
- **50+ Operators** - Flexible comparison logic (equals, contains, regex, numeric comparisons, etc.)
- **60+ Condition Properties** - Check virtually any aspect of the cart, customer, or checkout
- **Nested AND Conditions** - Create complex multi-level rule logic
- **Multi-Language Messages** - Display error messages in customer's language
- **Real-Time Validation** - Rules execute instantly at checkout via Shopify Functions

## How It Works

1. **Create a Rule** - Define conditions that should block checkout
2. **Configure Messages** - Set error messages in multiple languages
3. **Enable the Rule** - Activate the validation on your store
4. **Automatic Enforcement** - Shopify Functions validate every checkout attempt

## Use Cases

| Use Case | Validation Type |
|----------|-----------------|
| Limit quantity per customer | CUSTOMER, PRODUCT |
| Restrict shipping to certain regions | ADDRESS |
| Block checkout for specific product/customer combinations | TAGS |
| Time-limited sales | TIME |
| Minimum order values | CART |
| Restrict PO Box deliveries | ADDRESS |
| Limit gift card usage | REDUCTION |
| Block multiple shipments | SHIPPING |

## Shopify Plus Features

Some features require Shopify Plus:

- **REDUCTION** - Gift card and discount code validation
- **BILLING** - Billing address validation

## Architecture

BoomGate uses Shopify Functions to validate checkout in real-time:

```
Customer Checkout → Shopify Functions → BoomGate Validation → Allow/Block
```

Rules are stored as metafields on Shopify Validation resources, ensuring fast execution without external API calls.

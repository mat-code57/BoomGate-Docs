# Limitations

BoomGate provides three types of checkout rules: **checkout validation** (block checkout), **payment customization** (hide payment methods), and **delivery customization** (hide delivery options). There are certain things BoomGate **cannot** do.

---

## Cannot Show Warnings (Non-Blocking Messages)

Checkout validation can only **block checkout**. Payment/delivery customization can only **hide** options. BoomGate cannot:

- Show warning messages without blocking checkout
- Display informational messages that allow the customer to proceed
- Show "soft" notifications or alerts
- Show a message explaining why a payment method or delivery option was hidden

**Why:** Shopify Functions are designed for binary outcomes: allow/block (validation) or show/hide (payment/delivery). There is no "warning" or "message" state for payment/delivery customizations.

**What to tell customers:**
> "BoomGate checkout validations can block checkout with an error message. Payment and delivery customizations silently hide options - no message is shown to the customer."

---

## Cannot Modify Cart Contents

BoomGate cannot:

- Add products to the cart
- Remove products from the cart
- Change product quantities
- Modify line item properties

**Why:** Checkout Validation Functions are read-only - they can validate but not modify the cart.

---

## Cannot Change Prices or Apply Discounts

BoomGate cannot:

- Modify product prices
- Apply automatic discounts
- Remove discounts
- Change shipping costs

**Why:** Pricing is handled by Shopify's discount functions, not validation functions.

---

## Cannot Redirect Customers

BoomGate cannot:

- Redirect to a different page
- Send customers to an external URL
- Navigate to a specific product page

**Why:** Validation functions can only return allow/block decisions with error messages.

---

## Cannot Collect Additional Information

BoomGate cannot:

- Add custom form fields to checkout
- Collect additional customer input
- Display custom UI elements (except error messages)

**Why:** Checkout UI customization requires separate Shopify Checkout UI Extensions.

---

## Cannot Call External APIs During Validation

BoomGate cannot:

- Make HTTP requests to external services
- Validate against third-party databases
- Check real-time inventory from external systems

**Why:** Shopify Functions run in a sandboxed environment without network access for performance and security.

---

## Shopify Plus Requirements

Some features require Shopify Plus:

| Feature | Requires Plus |
|---------|---------------|
| REDUCTION validation (gift cards, discounts) | Yes |
| BILLING validation (billing address) | Yes |
| All other validation types | No |

---

## Property-Type Restrictions

Each property can ONLY be used with specific validation types. This is because each validation type has a different GraphQL schema that fetches different data from Shopify.

**Example:** `ADDRESS_COUNTRY_CODE` can only be used with the `ADDRESS` validation type because only the ADDRESS GraphQL schema queries the delivery address.

If you try to use a property with an incompatible type:
- The rule will fail validation
- Even if created, it won't work because the data isn't available

**Always use `list_properties` tool** to see which properties are available for your chosen validation type.

### Quick Reference - Exclusive Properties:

| Validation Type | Exclusive Properties |
|-----------------|---------------------|
| ADDRESS | ADDRESS_1, ADDRESS_2, ADDRESS_CITY, ADDRESS_ZIP, ADDRESS_COUNTRY_CODE, ADDRESS_PROVINCE_CODE, etc. |
| BILLING | BILLING_ADDRESS_* properties (Plus only) |
| SHIPPING | SHIPPING_SELECTED_SHIPPING_TITLE, SHIPPING_SELECTED_SHIPPING_TYPE |
| TIME | DATE, TIME, DATE_TIME |
| REDUCTION | APPLIED_GIFT_CARDS_COUNT, APPLIED_DISCOUNT_CODES_COUNT, APPLIED_DISCOUNT_CODE, TOTAL_GIFT_CARD_DISCOUNTS_AMOUNT |
| CUSTOMER | CUSTOMER_METAFIELD (exclusive), CUSTOMER_ORDER_NUMBER, CUSTOMER_AMOUNT_SPENT |
| PRODUCT | PRODUCT_COLLECTION, PRODUCT_METAFIELD |

---

## GraphQL Data Availability

BoomGate can only access data available in Shopify's Cart/Checkout GraphQL:

- Cannot access order history during checkout
- Cannot access full product catalog
- Cannot access customer's browsing history
- Limited to data provided by Shopify's validation input

---

## Single-Use Properties Per Rule

Certain properties can only be used ONCE within a single validation rule:

| Property | Limit |
|----------|-------|
| `PRODUCT_TAGS` | 1 condition per rule |
| `CUSTOMER_TAGS` | 1 condition per rule |
| `PRODUCT_METAFIELD` | 1 condition per rule |
| `CUSTOMER_METAFIELD` | 1 condition per rule |
| `LINE_ATTRIBUTE` | 1 condition per rule |
| `CART_ATTRIBUTE` | 1 condition per rule |
| `DATE_TIME` | 1 AFTER + 1 BEFORE max |
| `TIME` | 1 AFTER + 1 BEFORE max |

**Why:** These properties require specific input configuration that can only be set once per rule.

**Workarounds:**
- Combine multiple values into one condition: `["tag1", "tag2", "tag3"]`
- Create separate validation rules for different checks

---

## Single Error Message Per Rule

Each rule can display one error message when triggered:

- Cannot show multiple different messages from one rule
- Cannot show conditional messages based on which condition failed
- For different messages, create separate rules

---

## Common Impossible Requests

When users ask for these, explain the limitation:

| Request | Why Impossible |
|---------|----------------|
| "Show a warning but let them checkout" | No warning state - only block or allow |
| "Add a free gift when they buy X" | Cannot modify cart |
| "Apply 10% discount for VIP customers" | Cannot apply discounts |
| "Redirect to login page" | Cannot redirect |
| "Ask customer to confirm age" | Cannot collect input |
| "Check if product is in stock via API" | No external API access |
| "Show message when payment method is hidden" | Payment customization hides silently |
| "Show message when delivery option is hidden" | Delivery customization hides silently |
| "Reorder payment methods" | Can only hide, not reorder |
| "Change delivery option price" | Can only hide, not modify |
| "Add a new payment method" | Can only hide existing ones |

---

## Payment/Delivery Customization Limitations

### No Custom Messages

Payment and delivery customization rules silently hide options. You cannot:
- Show a message explaining why a payment method was hidden
- Show a message explaining why a delivery option was hidden
- Display alternative suggestions

### Cannot Reorder or Rename

BoomGate can only **hide** payment methods and delivery options. It cannot:
- Reorder payment methods
- Rename payment methods or delivery options
- Change delivery option prices
- Add new payment methods or delivery options

### actionValue Must Match

The `actionValue` must match the actual payment method name or delivery option title. If the name doesn't match (exact, substring, or regex), nothing will be hidden. Common issues:
- Payment method names come from Shopify Admin > Settings > Payments
- Delivery option titles come from Shopify Admin > Settings > Shipping and delivery
- Third-party carriers may have different names than expected

---

## What BoomGate CAN Do

BoomGate excels at:

**Checkout Validation:**
- Blocking checkout based on product/customer/cart conditions
- Displaying localized error messages
- Enforcing quantity limits
- Restricting shipping to certain regions
- Limiting purchases by customer type (tags)
- Time-based checkout restrictions
- Preventing specific product/customer combinations

**Payment Customization:**
- Hiding payment methods based on any condition (address, cart, customer, products, time, etc.)
- Using exact, substring, or regex matching for payment method names
- Conditionally showing/hiding payment methods by customer tag, region, cart value, etc.

**Delivery Customization:**
- Hiding delivery/shipping options based on any condition
- Using exact, substring, or regex matching for delivery option titles
- Conditionally showing/hiding delivery options by product tag, region, time, etc.

For features beyond these capabilities, consider:
- Shopify Flow for automation
- Shopify Scripts (Plus) for pricing
- Checkout UI Extensions for custom UI

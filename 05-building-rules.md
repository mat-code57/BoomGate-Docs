# Building Rules

Learn how to create effective validation rules in BoomGate.

## Rule Structure

Every rule consists of:

1. **Title** - Descriptive name for the rule
2. **Validation Type** - Which checkout aspect to validate
3. **Conditions** - What to check (can include nested AND conditions)
4. **Condition Mode** - How conditions combine (ALL or ANY)
5. **Error Message** - What customers see when blocked

---

## Condition Modes

### ALL Mode (Default)

All conditions must be true to trigger the rule (block checkout).

```
Condition 1 AND Condition 2 AND Condition 3 = BLOCK
```

**Use Case:** "Block if customer is NOT logged in AND cart total is over $500"

### ANY Mode

At least one condition must be true to trigger the rule.

```
Condition 1 OR Condition 2 OR Condition 3 = BLOCK
```

**Use Case:** "Block if shipping to Hawaii OR Alaska OR Puerto Rico"

---

## AND Conditions (Nested Logic)

AND conditions create line-level validation. The parent condition finds matching lines, then AND conditions are checked only on those lines.

### Structure

```
IF (Parent Condition matches)
   AND (Nested Condition 1)
   AND (Nested Condition 2)
THEN block
```

### Example: Quantity Limit for Specific Products

```json
{
  "conditions": [
    {
      "property": "PRODUCT_TAGS",
      "operator": "HAS_TAGS",
      "value": { "value": ["limited-edition"] },
      "andConditions": [
        {
          "property": "LINE_QUANTITY",
          "operator": "GREATHER_THAN",
          "value": { "value": "2" }
        }
      ]
    }
  ],
  "mode": "ALL"
}
```

This finds all products with "limited-edition" tag, then checks if quantity > 2 for those specific products.

### When to Use AND Conditions

- Quantity limits for specific products
- Price thresholds per product
- Line attribute validation for tagged items
- SKU-specific rules

---

## Error Messages

### Multi-Language Support

Define messages in multiple languages:

```json
{
  "msg": {
    "en": "You cannot purchase more than 5 items",
    "es": "No puede comprar más de 5 artículos",
    "fr": "Vous ne pouvez pas acheter plus de 5 articles",
    "de": "Sie können nicht mehr als 5 Artikel kaufen"
  }
}
```

The message displayed matches the customer's checkout language.

### Dynamic Placeholders

Use placeholders to show product-specific information:

| Placeholder | Description |
|-------------|-------------|
| `[product_title]` | Product title that triggered the error |
| `[product_variant]` | Variant title |
| `[product_sku]` | SKU code |

**Example:**
```
"Sorry, [product_title] cannot be shipped to your location"
```

---

## Input Configuration

The input object pre-configures what data to fetch from Shopify. This optimizes performance by only querying needed data.

### Product-Related Inputs

```json
{
  "hasProductTags": ["tag1", "tag2"],      // Fetch products with these tags
  "hasAnyProductTag": ["tag3", "tag4"],    // Fetch products with any of these tags
  "inAnyCollection": ["collection-id"],     // Fetch products in collections
  "productMetafieldNamespace": "custom",    // Metafield namespace to fetch
  "productMetafieldKey": "my_field"         // Metafield key to fetch
}
```

### Customer-Related Inputs

```json
{
  "hasCustomerTags": ["vip", "wholesale"],  // Customer tags to check
  "hasAnyCustomerTag": ["regular"],         // Any of these tags
  "customerMetafieldNamespace": "custom",   // Customer metafield namespace
  "customerMetafieldKey": "tier"            // Customer metafield key
}
```

### Attribute-Related Inputs

```json
{
  "lineAttributeKey": "size",      // Line attribute to fetch
  "cartAttributeKey": "promo"      // Cart attribute to fetch
}
```

### Time-Related Inputs

```json
{
  "timeAfter": "09:00:00",
  "timeBefore": "17:00:00",
  "dateTimeAfter": "2024-01-01T00:00:00",
  "dateTimeBefore": "2024-12-31T23:59:59"
}
```

---

## Rule Building Workflow

### Step 1: Define the Business Logic

Write out what you want to achieve in plain language:

> "Customers without the 'wholesale' tag cannot order more than 5 units of any product"

### Step 2: Choose Validation Type

Based on what data you need:
- Customer data → CUSTOMER
- Product-specific → PRODUCT
- Tag combinations → TAGS
- Address validation → ADDRESS

### Step 3: Build Conditions

Break down the logic into conditions:

1. Customer does NOT have "wholesale" tag
2. Line quantity is greater than 5

### Step 4: Determine Mode

- Need ALL conditions true? → ALL mode
- Need ANY condition true? → ANY mode

### Step 5: Configure Error Message

Write clear, actionable messages:

**Good:** "Only wholesale customers can order more than 5 units. Please log in with your wholesale account or reduce quantities."

**Bad:** "Error: validation failed"

### Step 6: Test the Rule

1. Enable the rule
2. Test with matching conditions (should block)
3. Test with non-matching conditions (should allow)
4. Test edge cases

---

## Best Practices

### Keep Rules Simple

- One business rule = one validation rule
- Avoid complex nested conditions when possible
- Use multiple simpler rules instead of one complex rule

### Use Descriptive Titles

**Good:** "Block orders over 5 units for non-wholesale customers"
**Bad:** "Quantity rule 1"

### Write Helpful Error Messages

- Explain why checkout is blocked
- Tell customers how to proceed
- Use dynamic placeholders when applicable

### Consider Performance

- Use specific tags/collections instead of broad conditions
- Minimize metafield queries when possible
- Test rules with large carts

### Test Thoroughly

- Test on development store first
- Verify with real checkout flow
- Check all condition combinations
- Test multi-language messages

---

## Common Patterns

### Pattern 1: Customer Tag Exception

Allow tagged customers to bypass a restriction:

```json
{
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
  "mode": "ALL"
}
```

### Pattern 2: Regional Restriction

Block shipping to specific regions for certain products:

```json
{
  "conditions": [
    {
      "property": "ADDRESS_COUNTRY_CODE",
      "operator": "ONE_OF",
      "value": { "value": ["GB", "DE", "FR"] }
    },
    {
      "property": "PRODUCT_TAGS",
      "operator": "HAS_TAGS",
      "value": { "value": ["no-eu-shipping"] }
    }
  ],
  "mode": "ALL"
}
```

### Pattern 3: Time Window

Allow purchases only during specific times:

```json
{
  "conditions": [
    {
      "property": "TIME",
      "operator": "TIME_BEFORE",
      "value": { "value": "09:00" }
    }
  ],
  "mode": "ANY"
}
```

Plus another rule for after hours:
```json
{
  "conditions": [
    {
      "property": "TIME",
      "operator": "TIME_AFTER",
      "value": { "value": "17:00" }
    }
  ],
  "mode": "ANY"
}
```

### Pattern 4: Minimum Order with Exceptions

Require minimum order but allow gift cards:

```json
{
  "conditions": [
    {
      "property": "CART_TOTAL",
      "operator": "LESS_THAN",
      "value": { "value": "50" }
    },
    {
      "property": "PRODUCT_IS_GIFT_CARD",
      "operator": "EQUALS",
      "value": { "value": "false" }
    }
  ],
  "mode": "ALL"
}
```

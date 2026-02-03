# API Reference

Technical reference for AI agents and developers integrating with BoomGate.

---

## Rule Schema

### Complete Rule Object

```typescript
interface Rule {
  title: string;                    // Human-readable rule name
  type: ValidationType;             // Validation type enum
  configuration: Configuration;     // Rule logic
  input: Input;                     // Pre-fetch configuration
}

interface Configuration {
  conditions: Condition[];          // Array of conditions
  msg: LocalizedMessage;            // Error messages by language
  mode: "ALL" | "ANY";              // Condition combination mode
  runOnCart?: boolean;              // Run on cart page (optional)
}

interface Condition {
  property: string;                 // Property to check
  operator: string;                 // Comparison operator
  value: ConditionValue;            // Value(s) to compare
  andConditions?: Condition[];      // Nested AND conditions
}

interface ConditionValue {
  value?: string | string[];        // Primary value
  count?: string;                   // For counting properties
  namespace?: string;               // For metafield operators
  key?: string;                     // For metafield operators
  attributeKey?: string;            // For attribute operators
}

interface LocalizedMessage {
  en: string;                       // English (required)
  es?: string;                      // Spanish
  fr?: string;                      // French
  de?: string;                      // German
  // ... other language codes
}
```

---

## Validation Types

```typescript
type ValidationType =
  | "GENERAL"
  | "PRODUCT"
  | "CUSTOMER"
  | "CART"
  | "TAGS"
  | "TIME"
  | "ADDRESS"
  | "SHIPPING"
  | "REDUCTION"  // Shopify Plus only
  | "BILLING";   // Shopify Plus only
```

### Type to Function ID Mapping

```javascript
const FUNCTION_IDS = {
  GENERAL: "checkout-validation",
  PRODUCT: "product-checkout-validation",
  CUSTOMER: "customer-checkout-validation",
  CART: "cart-checkout-validation",
  TAGS: "tags-checkout-validation",
  TIME: "time-checkout-validation",
  ADDRESS: "address-checkout-validation",
  SHIPPING: "shipping-checkout-validation",
  REDUCTION: "reduction-checkout-validation",
  BILLING: "billing-address-checkout-validation"
};
```

---

## Operators Reference

### String Operators

```typescript
type StringOperator =
  | "EQUALS"
  | "DOES_NOT_EQUAL"
  | "CONTAINS"
  | "DOES_NOT_CONTAIN"
  | "ONE_OF"
  | "NOT_ONE_OF"
  | "MATCHES_REGEX"
  | "DOES_NOT_MATCH_REGEX"
  | "CHARACTERS_COUNT_GREATER_THAN"
  | "CHARACTERS_COUNT_LESS_THAN"
  | "IS_BLANK"
  | "IS_NOT_BLANK";
```

### Numeric Operators

```typescript
type NumericOperator =
  | "EQUALS"
  | "GREATHER_THAN"        // Note: typo in codebase
  | "LESS_THAN"
  | "GREATHER_OR_EQUAL_THAN"
  | "LESS_OR_EQUAL_THAN"
  | "IS_MULTIPLE_OF"
  | "IS_NOT_MULTIPLE_OF";
```

### Tag Operators

```typescript
type TagOperator =
  | "HAS_TAGS"              // All tags present
  | "HAS_ANY_TAG"           // At least one tag
  | "DOES_NOT_HAVE_TAGS";   // None of the tags
```

### Collection Operators

```typescript
type CollectionOperator = "IN_ANY_COLLECTION";
```

### Metafield Operators

```typescript
type MetafieldOperator =
  | "METAFIELD_VALUE_EQUALS"
  | "METAFIELD_VALUE_CONTAINS"
  | "METAFIELD_VALUE_ONE_OF"
  | "METAFIELD_VALUE_GREATHER_THAN"
  | "METAFIELD_VALUE_LESS_THAN"
  | "METAFIELD_VALUE_GREATHER_OR_EQUAL_THAN"
  | "METAFIELD_VALUE_LESS_OR_EQUAL_THAN";
```

### Attribute Operators

```typescript
type AttributeOperator =
  | "ATTRIBUTE_VALUE_EQUALS"
  | "ATTRIBUTE_VALUE_DOES_NOT_EQUAL"
  | "ATTRIBUTE_VALUE_CONTAINS"
  | "ATTRIBUTE_VALUE_ONE_OF"
  | "ATTRIBUTE_VALUE_GREATHER_THAN"
  | "ATTRIBUTE_VALUE_LESS_THAN"
  | "ATTRIBUTE_VALUE_GREATHER_OR_EQUAL_THAN"
  | "ATTRIBUTE_VALUE_LESS_OR_EQUAL_THAN"
  | "ATTRIBUTE_VALUE_MATCHES_REGEX"
  | "ATTRIBUTE_VALUE_DOES_NOT_MATCH_REGEX"
  | "ATTRIBUTE_EXISTS";
```

### Date/Time Operators

```typescript
type DateTimeOperator =
  | "DATE_EQUAL"
  | "DATE_AFTER"
  | "DATE_BEFORE"
  | "DATE_EQUAL_OR_AFTER"
  | "DATE_EQUAL_OR_BEFORE"
  | "TIME_AFTER"
  | "TIME_BEFORE"
  | "DATE_TIME_AFTER"
  | "DATE_TIME_BEFORE";
```

---

## Properties Reference

### Product Properties

| Property | Type | Operators | AND Allowed |
|----------|------|-----------|-------------|
| `PRODUCT_TAGS` | array | Tag | Yes |
| `PRODUCT_TYPE` | string | String | Yes |
| `PRODUCT_TITLE` | string | String | Yes |
| `PRODUCT_VENDOR` | string | String | Yes |
| `PRODUCT_HANDLE` | string | String | Yes |
| `PRODUCT_SKU` | string | String | Yes |
| `PRODUCT_WEIGHT` | number | Numeric | Yes |
| `PRODUCT_REQUIRES_SHIPPING` | boolean | String (true/false) | Yes |
| `PRODUCT_IS_GIFT_CARD` | boolean | String (true/false) | Yes |
| `PRODUCT_COLLECTION` | array | Collection | Yes |
| `PRODUCT_METAFIELD` | varies | Metafield | Yes |

### Line Item Properties

| Property | Type | Operators | AND Allowed |
|----------|------|-----------|-------------|
| `LINE_QUANTITY` | number | Numeric | Yes |
| `LINE_TOTAL` | number | Numeric | Yes |
| `LINE_SUBTOTAL` | number | Numeric | Yes |
| `LINE_AMOUNT_PER_QUANTITY` | number | Numeric | Yes |
| `LINE_COMPARE_AT_AMOUNT_PER_QUANTITY` | number | Numeric | Yes |
| `LINE_ATTRIBUTE` | string | Attribute | Yes |
| `LINES_COUNT` | number | Numeric | No |
| `LINES_WITH_ANY_TAG_COUNT` | number | Numeric | No |
| `LINES_WITH_NO_TAGS_COUNT` | number | Numeric | No |

### Cart Properties

| Property | Type | Operators |
|----------|------|-----------|
| `CART_TOTAL` | number | Numeric |
| `CART_SUBTOTAL` | number | Numeric |
| `CART_TAX` | number | Numeric |
| `CART_DUTY` | number | Numeric |
| `CART_TOTAL_WEIGHT` | number | Numeric |
| `CART_ATTRIBUTE` | string | Attribute |

### Customer Properties

| Property | Type | Operators | AND Allowed |
|----------|------|-----------|-------------|
| `CUSTOMER_IS_AUTHENTICATED` | boolean | String | Yes |
| `CUSTOMER_FIRST_NAME` | string | String | Yes |
| `CUSTOMER_LAST_NAME` | string | String | Yes |
| `CUSTOMER_EMAIL` | string | String | Yes |
| `CUSTOMER_PHONE` | string | String | Yes |
| `CUSTOMER_COMPANY_NAME` | string | String | Yes |
| `CUSTOMER_ORDER_NUMBER` | number | Numeric | Yes |
| `CUSTOMER_AMOUNT_SPENT` | number | Numeric | Yes |
| `CUSTOMER_TAGS` | array | Tag | Yes |
| `CUSTOMER_METAFIELD` | varies | Metafield | Yes |

### Address Properties

| Property | Type | Operators |
|----------|------|-----------|
| `ADDRESS_COUNTRY_CODE` | string | String |
| `ADDRESS_PROVINCE_CODE` | string | String |
| `ADDRESS_CITY` | string | String |
| `ADDRESS_ZIP` | string | String |
| `ADDRESS_1` | string | String |
| `ADDRESS_2` | string | String |
| `ADDRESS_COMPANY` | string | String |
| `ADDRESS_FIRST_NAME` | string | String |
| `ADDRESS_LAST_NAME` | string | String |

### Time Properties

| Property | Type | Operators |
|----------|------|-----------|
| `DATE` | date | DateTime |
| `TIME` | time | DateTime |
| `DATE_TIME` | datetime | DateTime |

### Shipping Properties

| Property | Type | Operators |
|----------|------|-----------|
| `SHIPPING_SELECTED_SHIPPING_TITLE` | string | String |
| `SHIPPING_SELECTED_SHIPPING_TYPE` | string | String |
| `TOTAL_SHIPMENTS_COUNT` | number | Numeric |

### Reduction Properties (Plus Only)

| Property | Type | Operators |
|----------|------|-----------|
| `APPLIED_GIFT_CARDS_COUNT` | number | Numeric |
| `TOTAL_GIFT_CARD_DISCOUNTS_AMOUNT` | number | Numeric |
| `APPLIED_DISCOUNT_CODES_COUNT` | number | Numeric |
| `APPLIED_DISCOUNT_CODE` | string | String |

---

## Input Configuration

### Product Input

```typescript
interface ProductInput {
  hasProductTags?: string[];              // Tags to query
  hasAnyProductTag?: string[];            // Any-of tags to query
  inAnyCollection?: string[];             // Collection GIDs
  productMetafieldNamespace?: string;     // Metafield namespace
  productMetafieldKey?: string;           // Metafield key
  lineAttributeKey?: string;              // Line attribute key
}
```

### Customer Input

```typescript
interface CustomerInput {
  hasCustomerTags?: string[];             // Tags to query
  hasAnyCustomerTag?: string[];           // Any-of tags
  customerMetafieldNamespace?: string;    // Metafield namespace
  customerMetafieldKey?: string;          // Metafield key
}
```

### Cart Input

```typescript
interface CartInput {
  cartAttributeKey?: string;              // Cart attribute key
  lineAttributeKey?: string;              // Line attribute key
}
```

### Time Input

```typescript
interface TimeInput {
  timeAfter?: string;                     // HH:MM:SS format
  timeBefore?: string;                    // HH:MM:SS format
  dateTimeAfter?: string;                 // ISO datetime
  dateTimeBefore?: string;                // ISO datetime
}
```

---

## Value Formats

### Date Format
```
YYYY-MM-DD
Example: 2024-12-31
```

### Time Format
```
HH:MM
Example: 09:00
```

### DateTime Format
```
YYYY-MM-DDTHH:MM
Example: 2024-12-31T23:59
```

### Country Codes (ISO 3166-1 alpha-2)
```
US, CA, GB, AU, DE, FR, JP, CN, etc.
```

### Province/State Codes
```
US: CA, NY, TX, FL, etc.
CA: ON, BC, QC, AB, etc.
AU: NSW, VIC, QLD, etc.
```

---

## Error Message Placeholders

| Placeholder | Description | Example Output |
|-------------|-------------|----------------|
| `[product_title]` | Product title | "Blue T-Shirt" |
| `[product_variant]` | Variant title | "Large / Blue" |
| `[product_sku]` | SKU code | "TSH-BLU-L" |

---

## Supported Languages

```typescript
const SUPPORTED_LANGUAGES = [
  "en",  // English
  "es",  // Spanish
  "fr",  // French
  "de",  // German
  "it",  // Italian
  "pt",  // Portuguese
  "nl",  // Dutch
  "pl",  // Polish
  "ja",  // Japanese
  "ko",  // Korean
  "zh-CN", // Simplified Chinese
  "zh-TW"  // Traditional Chinese
];
```

---

## Metafield Storage

Rules are stored in Shopify metafields:

```
Namespace: $app:checkout-validation
Keys:
  - configuration: JSON string of Configuration object
  - input: JSON string of Input object
  - type: String validation type
```

---

## Building Rules Programmatically

### Example: Create Quantity Limit Rule

```javascript
const rule = {
  title: "Limit to 5 items for guests",
  type: "CUSTOMER",
  configuration: {
    conditions: [
      {
        property: "CUSTOMER_IS_AUTHENTICATED",
        operator: "EQUALS",
        value: { value: "false" },
        andConditions: []
      },
      {
        property: "LINE_QUANTITY",
        operator: "GREATHER_THAN",
        value: { value: "5" },
        andConditions: []
      }
    ],
    msg: {
      en: "Please log in to order more than 5 items"
    },
    mode: "ALL"
  },
  input: {
    hasCustomerTags: [],
    customerMetafieldNamespace: "BLANK",
    customerMetafieldKey: "BLANK"
  }
};
```

### Validation Tips

1. **Property-Operator Compatibility**: Ensure operator is valid for property type
2. **Input Consistency**: Input must match properties used in conditions
3. **BLANK Values**: Use "BLANK" for unused metafield/attribute keys
4. **Mode Selection**: ALL for strict rules, ANY for flexible rules
5. **AND Conditions**: Only use with line-level properties

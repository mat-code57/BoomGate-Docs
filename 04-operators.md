# Operators

Operators define HOW to compare values. BoomGate provides 50+ operators across different categories.

## Operator Categories

1. **String Operators** - Text comparisons
2. **Numeric Operators** - Number comparisons
3. **Tag Operators** - Tag array operations
4. **Collection Operators** - Collection membership
5. **Metafield Operators** - Custom field comparisons
6. **Attribute Operators** - Cart/line attribute comparisons
7. **Date/Time Operators** - Temporal comparisons

---

## String Operators

For text field comparisons.

| Operator | Description | Example |
|----------|-------------|---------|
| `EQUALS` | Exact match | "US" equals "US" |
| `DOES_NOT_EQUAL` | Not exact match | "US" does not equal "CA" |
| `CONTAINS` | Substring present | "hello world" contains "world" |
| `DOES_NOT_CONTAIN` | Substring absent | "hello" does not contain "bye" |
| `ONE_OF` | Value in list | "US" one of ["US", "CA", "GB"] |
| `NOT_ONE_OF` | Value not in list | "AU" not one of ["US", "CA"] |
| `MATCHES_REGEX` | Regex pattern match | "PO Box 123" matches regex |
| `DOES_NOT_MATCH_REGEX` | Regex pattern mismatch | Address doesn't match PO Box pattern |
| `CHARACTERS_COUNT_GREATER_THAN` | String length > N | ZIP code length > 5 |
| `CHARACTERS_COUNT_LESS_THAN` | String length < N | Name length < 50 |
| `IS_BLANK` | Empty or null | Company field is blank |
| `IS_NOT_BLANK` | Has value | Email is not blank |

### Regex Examples

**Block PO Boxes:**
```regex
([Pp][.\s]*[Oo][.\s]?\s*[Bb][Oo][Xx])|([Bb][Oo][Xx][^a-zA-Z]*\d+)
```

**Valid Email Pattern:**
```regex
^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$
```

**Numeric ZIP Code:**
```regex
^\d{5}(-\d{4})?$
```

---

## Numeric Operators

For number comparisons. Values are parsed as floats.

| Operator | Description | Example |
|----------|-------------|---------|
| `EQUALS` | Exact number | quantity = 5 |
| `GREATHER_THAN` | Greater than | total > 100 |
| `LESS_THAN` | Less than | weight < 50 |
| `GREATHER_OR_EQUAL_THAN` | Greater or equal | quantity >= 10 |
| `LESS_OR_EQUAL_THAN` | Less or equal | total <= 500 |
| `IS_MULTIPLE_OF` | Divisible by N | quantity is multiple of 6 |
| `IS_NOT_MULTIPLE_OF` | Not divisible | quantity is not multiple of 10 |

### Numeric Examples

**Minimum Order:**
```json
{
  "property": "CART_TOTAL",
  "operator": "LESS_THAN",
  "value": { "value": "50" }
}
```

**Quantity Limit:**
```json
{
  "property": "LINE_QUANTITY",
  "operator": "GREATHER_THAN",
  "value": { "value": "5" }
}
```

**Bundle Multiples:**
```json
{
  "property": "LINE_QUANTITY",
  "operator": "IS_NOT_MULTIPLE_OF",
  "value": { "value": "6" }
}
```

---

## Tag Operators

For product and customer tag arrays.

| Operator | Description | Match Type |
|----------|-------------|------------|
| `HAS_TAGS` | Has ALL specified tags | AND logic |
| `HAS_ANY_TAG` | Has at least ONE tag | OR logic |
| `DOES_NOT_HAVE_TAGS` | Has NONE of the tags | NOT logic |

### Tag Examples

**Has All Tags:**
```json
{
  "property": "PRODUCT_TAGS",
  "operator": "HAS_TAGS",
  "value": { "value": ["wholesale", "bulk"] }
}
// Product must have BOTH "wholesale" AND "bulk" tags
```

**Has Any Tag:**
```json
{
  "property": "CUSTOMER_TAGS",
  "operator": "HAS_ANY_TAG",
  "value": { "value": ["vip", "premium", "gold"] }
}
// Customer must have at least ONE of the tags
```

**Does Not Have Tags:**
```json
{
  "property": "PRODUCT_TAGS",
  "operator": "DOES_NOT_HAVE_TAGS",
  "value": { "value": ["restricted"] }
}
// Product must NOT have "restricted" tag
```

---

## Collection Operators

For product collection membership.

| Operator | Description |
|----------|-------------|
| `IN_ANY_COLLECTION` | Product in specified collection(s) |

### Collection Example

```json
{
  "property": "PRODUCT_COLLECTION",
  "operator": "IN_ANY_COLLECTION",
  "value": { "value": ["gid://shopify/Collection/123456"] }
}
```

---

## Metafield Operators

For custom metafield comparisons. Require namespace and key.

| Operator | Description |
|----------|-------------|
| `METAFIELD_VALUE_EQUALS` | Metafield exact match |
| `METAFIELD_VALUE_CONTAINS` | Metafield contains substring |
| `METAFIELD_VALUE_ONE_OF` | Metafield in list |
| `METAFIELD_VALUE_GREATHER_THAN` | Metafield numeric > |
| `METAFIELD_VALUE_LESS_THAN` | Metafield numeric < |
| `METAFIELD_VALUE_GREATHER_OR_EQUAL_THAN` | Metafield numeric >= |
| `METAFIELD_VALUE_LESS_OR_EQUAL_THAN` | Metafield numeric <= |

### Metafield Example

```json
{
  "property": "PRODUCT_METAFIELD",
  "operator": "METAFIELD_VALUE_EQUALS",
  "value": {
    "namespace": "inventory",
    "key": "restricted_shipping",
    "value": "true"
  }
}
```

---

## Attribute Operators

For cart and line item attributes.

| Operator | Description |
|----------|-------------|
| `ATTRIBUTE_VALUE_EQUALS` | Attribute exact match |
| `ATTRIBUTE_VALUE_DOES_NOT_EQUAL` | Attribute not equal |
| `ATTRIBUTE_VALUE_CONTAINS` | Attribute contains |
| `ATTRIBUTE_VALUE_ONE_OF` | Attribute in list |
| `ATTRIBUTE_VALUE_GREATHER_THAN` | Attribute numeric > |
| `ATTRIBUTE_VALUE_LESS_THAN` | Attribute numeric < |
| `ATTRIBUTE_VALUE_GREATHER_OR_EQUAL_THAN` | Attribute numeric >= |
| `ATTRIBUTE_VALUE_LESS_OR_EQUAL_THAN` | Attribute numeric <= |
| `ATTRIBUTE_VALUE_MATCHES_REGEX` | Attribute regex match |
| `ATTRIBUTE_VALUE_DOES_NOT_MATCH_REGEX` | Attribute regex mismatch |
| `ATTRIBUTE_EXISTS` | Attribute present/absent |

### Attribute Example

```json
{
  "property": "CART_ATTRIBUTE",
  "operator": "ATTRIBUTE_VALUE_EQUALS",
  "value": {
    "attributeKey": "gift_message",
    "value": ""
  }
}
```

---

## Date/Time Operators

For temporal comparisons. Use shop timezone.

| Operator | Description | Value Format |
|----------|-------------|--------------|
| `DATE_EQUAL` | Exact date | YYYY-MM-DD |
| `DATE_AFTER` | After date | YYYY-MM-DD |
| `DATE_BEFORE` | Before date | YYYY-MM-DD |
| `DATE_EQUAL_OR_AFTER` | On or after | YYYY-MM-DD |
| `DATE_EQUAL_OR_BEFORE` | On or before | YYYY-MM-DD |
| `TIME_AFTER` | After time | HH:MM |
| `TIME_BEFORE` | Before time | HH:MM |
| `DATE_TIME_AFTER` | After datetime | YYYY-MM-DDTHH:MM |
| `DATE_TIME_BEFORE` | Before datetime | YYYY-MM-DDTHH:MM |

### Date/Time Examples

**Block After Date:**
```json
{
  "property": "DATE_TIME",
  "operator": "DATE_TIME_AFTER",
  "value": { "value": "2024-12-31T23:59" }
}
```

**Business Hours Only:**
```json
{
  "property": "TIME",
  "operator": "TIME_BEFORE",
  "value": { "value": "09:00" }
}
```

---

## Operator Compatibility Matrix

Not all operators work with all properties. Here's a quick reference:

| Property Type | Compatible Operators |
|---------------|---------------------|
| Text fields (title, email, address) | String operators |
| Numeric fields (quantity, total, weight) | Numeric operators |
| Tag fields (product_tags, customer_tags) | Tag operators |
| Collection fields | Collection operators |
| Metafield properties | Metafield operators |
| Attribute properties | Attribute operators |
| Date/time properties | Date/time operators |

The app UI will automatically show only compatible operators for each property.

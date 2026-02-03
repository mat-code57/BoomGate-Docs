# Rule Examples

Real-world examples of BoomGate validation rules.

---

## Quantity Limits

### Limit Quantity to 5 for Non-Wholesale Customers

Block non-wholesale customers from ordering more than 5 units.

**Type:** CUSTOMER

```json
{
  "title": "Block non-wholesale customers from ordering more than 5",
  "type": "CUSTOMER",
  "configuration": {
    "conditions": [
      {
        "property": "CUSTOMER_TAGS",
        "operator": "DOES_NOT_HAVE_TAGS",
        "value": { "value": ["wholesale"] },
        "andConditions": []
      },
      {
        "property": "LINE_QUANTITY",
        "operator": "GREATHER_THAN",
        "value": { "value": "5" },
        "andConditions": []
      }
    ],
    "msg": {
      "en": "Only wholesale customers can order more than 5 items"
    },
    "mode": "ALL"
  },
  "input": {
    "hasCustomerTags": ["wholesale"],
    "customerMetafieldNamespace": "BLANK",
    "customerMetafieldKey": "BLANK"
  }
}
```

### Limit Specific SKU to 1 Per Order

**Type:** PRODUCT

```json
{
  "title": "Block checkout if more than 1 Limited Edition item",
  "type": "PRODUCT",
  "configuration": {
    "conditions": [
      {
        "property": "PRODUCT_SKU",
        "operator": "EQUALS",
        "value": { "value": "LIMITED-001" },
        "andConditions": [
          {
            "property": "LINE_QUANTITY",
            "operator": "GREATHER_THAN",
            "value": { "value": "1" }
          }
        ]
      }
    ],
    "msg": {
      "en": "Sorry, there is a 1 item limit for [product_title]"
    },
    "mode": "ALL"
  },
  "input": {
    "productMetafieldNamespace": "BLANK",
    "productMetafieldKey": "BLANK",
    "lineAttributeKey": "BLANK"
  }
}
```

### Force Wholesale Customers to Buy in Packs of 10

**Type:** TAGS

```json
{
  "title": "Wholesale must buy in multiples of 10",
  "type": "TAGS",
  "configuration": {
    "conditions": [
      {
        "property": "CUSTOMER_TAGS",
        "operator": "HAS_TAGS",
        "value": { "value": ["wholesale"] },
        "andConditions": []
      },
      {
        "property": "LINE_QUANTITY",
        "operator": "IS_NOT_MULTIPLE_OF",
        "value": { "value": "10" },
        "andConditions": []
      }
    ],
    "msg": {
      "en": "Wholesale customers must order in packs of 10"
    },
    "mode": "ALL"
  },
  "input": {
    "hasCustomerTags": ["wholesale"],
    "hasProductTags": [],
    "lineAttributeKey": "BLANK"
  }
}
```

---

## Shipping Restrictions

### Block Shipping to Hawaii, Alaska, Puerto Rico

**Type:** ADDRESS

```json
{
  "title": "No shipping to HI, AK, PR for restricted products",
  "type": "ADDRESS",
  "configuration": {
    "conditions": [
      {
        "property": "ADDRESS_PROVINCE_CODE",
        "operator": "ONE_OF",
        "value": { "value": ["HI", "AK", "PR", "GU"] },
        "andConditions": []
      },
      {
        "property": "PRODUCT_TAGS",
        "operator": "HAS_ANY_TAG",
        "value": { "value": ["restricted-shipping"] },
        "andConditions": []
      }
    ],
    "msg": {
      "en": "[product_title] cannot be shipped to Hawaii, Alaska, Guam, or Puerto Rico"
    },
    "mode": "ALL"
  },
  "input": {
    "hasAnyProductTag": ["restricted-shipping"],
    "cartAttributeKey": "BLANK"
  }
}
```

### Block PO Box Deliveries

**Type:** ADDRESS

```json
{
  "title": "Block PO Box deliveries",
  "type": "ADDRESS",
  "configuration": {
    "conditions": [
      {
        "property": "ADDRESS_1",
        "operator": "MATCHES_REGEX",
        "value": { "value": "([Pp][.\\s]*[Oo][.\\s]?\\s*[Bb][Oo][Xx])|([Bb][Oo][Xx][^a-zA-Z]*\\d+)" },
        "andConditions": []
      }
    ],
    "msg": {
      "en": "We don't ship to PO Boxes"
    },
    "mode": "ALL"
  },
  "input": {
    "hasProductTags": [],
    "cartAttributeKey": "BLANK"
  }
}
```

### Block International Shipping for Certain Products

**Type:** ADDRESS

```json
{
  "title": "US-only products",
  "type": "ADDRESS",
  "configuration": {
    "conditions": [
      {
        "property": "ADDRESS_COUNTRY_CODE",
        "operator": "DOES_NOT_EQUAL",
        "value": { "value": "US" },
        "andConditions": []
      },
      {
        "property": "PRODUCT_TAGS",
        "operator": "HAS_TAGS",
        "value": { "value": ["us-only"] },
        "andConditions": []
      }
    ],
    "msg": {
      "en": "[product_title] can only be shipped within the United States"
    },
    "mode": "ALL"
  },
  "input": {
    "hasProductTags": ["us-only"],
    "cartAttributeKey": "BLANK"
  }
}
```

---

## Customer Restrictions

### Require Customer Login

**Type:** CUSTOMER

```json
{
  "title": "Block unauthenticated checkout",
  "type": "CUSTOMER",
  "configuration": {
    "conditions": [
      {
        "property": "CUSTOMER_IS_AUTHENTICATED",
        "operator": "EQUALS",
        "value": { "value": "false" },
        "andConditions": []
      }
    ],
    "msg": {
      "en": "You must be logged in to complete checkout"
    },
    "mode": "ALL"
  },
  "input": {
    "hasCustomerTags": [],
    "customerMetafieldNamespace": "BLANK",
    "customerMetafieldKey": "BLANK"
  }
}
```

### Block Customers with Specific Tag

**Type:** CUSTOMER

```json
{
  "title": "Block restricted customers",
  "type": "CUSTOMER",
  "configuration": {
    "conditions": [
      {
        "property": "CUSTOMER_TAGS",
        "operator": "HAS_TAGS",
        "value": { "value": ["blocked", "fraud"] },
        "andConditions": []
      }
    ],
    "msg": {
      "en": "Your account is restricted. Please contact support."
    },
    "mode": "ANY"
  },
  "input": {
    "hasCustomerTags": ["blocked", "fraud"],
    "customerMetafieldNamespace": "BLANK",
    "customerMetafieldKey": "BLANK"
  }
}
```

---

## Time-Based Rules

### Block Checkout After Sale Ends

**Type:** TIME

```json
{
  "title": "Block sale items after sale ends",
  "type": "TIME",
  "configuration": {
    "conditions": [
      {
        "property": "DATE_TIME",
        "operator": "DATE_TIME_AFTER",
        "value": { "value": "2024-12-31T23:59" },
        "andConditions": []
      },
      {
        "property": "PRODUCT_TAGS",
        "operator": "HAS_TAGS",
        "value": { "value": ["holiday-sale"] },
        "andConditions": []
      }
    ],
    "msg": {
      "en": "The holiday sale has ended. Remove [product_title] from your cart."
    },
    "mode": "ALL"
  },
  "input": {
    "dateTimeAfter": "2024-12-31T23:59:00",
    "dateTimeBefore": "2000-01-01T00:00:00"
  }
}
```

### Business Hours Only

**Type:** TIME (Two rules needed)

Rule 1 - Block before opening:
```json
{
  "title": "Block checkout before 9 AM",
  "type": "TIME",
  "configuration": {
    "conditions": [
      {
        "property": "TIME",
        "operator": "TIME_BEFORE",
        "value": { "value": "09:00" },
        "andConditions": []
      }
    ],
    "msg": {
      "en": "Orders can only be placed between 9 AM and 5 PM"
    },
    "mode": "ALL"
  },
  "input": {
    "timeBefore": "09:00:00"
  }
}
```

Rule 2 - Block after closing:
```json
{
  "title": "Block checkout after 5 PM",
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
    "msg": {
      "en": "Orders can only be placed between 9 AM and 5 PM"
    },
    "mode": "ALL"
  },
  "input": {
    "timeAfter": "17:00:00"
  }
}
```

---

## Cart Rules

### Minimum Order Value

**Type:** CART

```json
{
  "title": "Minimum order $50",
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
    "msg": {
      "en": "Minimum order value is $50. Please add more items to your cart."
    },
    "mode": "ALL"
  },
  "input": {
    "lineAttributeKey": "BLANK",
    "cartAttributeKey": "BLANK"
  }
}
```

### Maximum Items Per Order

**Type:** CART

```json
{
  "title": "Maximum 10 items per order",
  "type": "CART",
  "configuration": {
    "conditions": [
      {
        "property": "LINES_COUNT",
        "operator": "GREATHER_THAN",
        "value": { "value": "10" },
        "andConditions": []
      }
    ],
    "msg": {
      "en": "Maximum 10 different items per order"
    },
    "mode": "ALL"
  },
  "input": {
    "lineAttributeKey": "BLANK",
    "cartAttributeKey": "BLANK"
  }
}
```

---

## Tag Combinations

### Block Customer Tag + Product Tag Combination

**Type:** TAGS

```json
{
  "title": "Retail customers cannot buy wholesale products",
  "type": "TAGS",
  "configuration": {
    "conditions": [
      {
        "property": "CUSTOMER_TAGS",
        "operator": "HAS_TAGS",
        "value": { "value": ["retail"] },
        "andConditions": []
      },
      {
        "property": "PRODUCT_TAGS",
        "operator": "HAS_TAGS",
        "value": { "value": ["wholesale-only"] },
        "andConditions": []
      }
    ],
    "msg": {
      "en": "[product_title] is only available to wholesale customers"
    },
    "mode": "ALL"
  },
  "input": {
    "hasCustomerTags": ["retail"],
    "hasProductTags": ["wholesale-only"],
    "lineAttributeKey": "BLANK"
  }
}
```

### Limit Free Gift Items

**Type:** TAGS

```json
{
  "title": "Maximum 1 free gift per order",
  "type": "TAGS",
  "configuration": {
    "conditions": [
      {
        "property": "LINES_WITH_ANY_TAG_COUNT",
        "operator": "GREATHER_THAN",
        "value": {
          "value": ["free-gift"],
          "count": "1"
        },
        "andConditions": []
      }
    ],
    "msg": {
      "en": "Only 1 free gift item allowed per order"
    },
    "mode": "ALL"
  },
  "input": {
    "hasAnyProductTag": ["free-gift"],
    "hasProductTags": [],
    "lineAttributeKey": "BLANK"
  }
}
```

---

## Shipping Method Rules

### Block Pickup for Large Items

**Type:** SHIPPING

```json
{
  "title": "No pickup for oversized items",
  "type": "SHIPPING",
  "configuration": {
    "conditions": [
      {
        "property": "SHIPPING_SELECTED_SHIPPING_TYPE",
        "operator": "EQUALS",
        "value": { "value": "PICK_UP" },
        "andConditions": []
      },
      {
        "property": "PRODUCT_TAGS",
        "operator": "HAS_TAGS",
        "value": { "value": ["oversized"] },
        "andConditions": []
      }
    ],
    "msg": {
      "en": "[product_title] is too large for store pickup"
    },
    "mode": "ALL"
  },
  "input": {
    "hasProductTags": ["oversized"],
    "hasCustomerTags": []
  }
}
```

### Block Split Shipments

**Type:** SHIPPING

```json
{
  "title": "No split shipments",
  "type": "SHIPPING",
  "configuration": {
    "conditions": [
      {
        "property": "TOTAL_SHIPMENTS_COUNT",
        "operator": "GREATHER_THAN",
        "value": { "count": "1" },
        "andConditions": []
      }
    ],
    "msg": {
      "en": "Your order would require multiple shipments. Please reduce items."
    },
    "mode": "ALL"
  },
  "input": {
    "hasProductTags": [],
    "hasCustomerTags": []
  }
}
```

---

## Discount Rules (Shopify Plus)

### Limit Gift Cards to 1 Per Order

**Type:** REDUCTION

```json
{
  "title": "Maximum 1 gift card per order",
  "type": "REDUCTION",
  "configuration": {
    "conditions": [
      {
        "property": "APPLIED_GIFT_CARDS_COUNT",
        "operator": "GREATHER_THAN",
        "value": { "count": "1" },
        "andConditions": []
      }
    ],
    "msg": {
      "en": "Only 1 gift card can be used per order"
    },
    "mode": "ALL"
  },
  "input": {}
}
```

### Block Discounts on Sale Items

**Type:** REDUCTION

```json
{
  "title": "No discount codes on sale items",
  "type": "REDUCTION",
  "configuration": {
    "conditions": [
      {
        "property": "APPLIED_DISCOUNT_CODES_COUNT",
        "operator": "GREATHER_THAN",
        "value": { "count": "0" },
        "andConditions": []
      },
      {
        "property": "PRODUCT_TAGS",
        "operator": "HAS_ANY_TAG",
        "value": { "value": ["sale", "clearance"] },
        "andConditions": []
      }
    ],
    "msg": {
      "en": "Discount codes cannot be used with sale items"
    },
    "mode": "ALL"
  },
  "input": {}
}
```

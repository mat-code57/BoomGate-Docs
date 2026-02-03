# Troubleshooting

Common issues and solutions for BoomGate validation rules.

---

## Rule Not Triggering

### Issue: Rule is enabled but not blocking checkout

**Possible Causes:**

1. **Conditions not matching**
   - Verify condition values match actual data
   - Check tag spelling (case-sensitive)
   - Verify country/province codes are correct

2. **Wrong validation type**
   - Ensure type matches the properties used
   - PRODUCT type for product-specific conditions
   - ADDRESS type for shipping address conditions

3. **Mode configuration**
   - ALL mode: All conditions must be true
   - ANY mode: At least one condition must be true

4. **Input not configured**
   - Tags must be listed in input for GraphQL query
   - Metafield namespace/key must match input

**Solution:**
- Review all conditions and values
- Test with simpler conditions first
- Check Shopify Functions logs

---

## Rule Blocking Unexpectedly

### Issue: Rule blocks checkout when it shouldn't

**Possible Causes:**

1. **Overly broad conditions**
   - Condition matches more than intended
   - Missing exception conditions

2. **Mode misconfiguration**
   - Using ANY mode when ALL is needed
   - Conditions trigger independently

3. **Tag matching issues**
   - HAS_ANY_TAG vs HAS_TAGS difference
   - Partial tag matches

**Solution:**
- Add more specific conditions
- Use HAS_TAGS for exact tag matching
- Add exception conditions with DOES_NOT_HAVE_TAGS

---

## Error Messages

### Issue: Wrong language displayed

**Cause:** Message not defined for checkout language

**Solution:**
- Add translations for all target languages
- English (en) is used as fallback

### Issue: Placeholder not replaced

**Cause:** Placeholder syntax incorrect

**Solution:**
- Use exact format: `[product_title]`
- Available: `[product_title]`, `[product_variant]`, `[product_sku]`

---

## Tag Validation Issues

### Issue: Tag condition not working

**Common Problems:**

1. **Case sensitivity**
   - Tags are case-sensitive
   - "Wholesale" ≠ "wholesale"

2. **Whitespace**
   - Leading/trailing spaces in tags
   - Check for hidden characters

3. **Input configuration**
   - Tags must be in input object for GraphQL
   - `hasProductTags` for HAS_TAGS
   - `hasAnyProductTag` for HAS_ANY_TAG

**Solution:**
```json
{
  "configuration": {
    "conditions": [{
      "property": "PRODUCT_TAGS",
      "operator": "HAS_TAGS",
      "value": { "value": ["wholesale"] }
    }]
  },
  "input": {
    "hasProductTags": ["wholesale"]  // Must match!
  }
}
```

---

## Address Validation Issues

### Issue: Country/province code not matching

**Common Problems:**

1. **Wrong code format**
   - Use ISO 3166-1 alpha-2 for countries
   - Use local province codes

2. **Code examples:**
   - United Kingdom: `GB` (not UK)
   - United States: `US`
   - California: `CA`
   - Ontario: `ON`

**Solution:**
- Verify codes at: https://en.wikipedia.org/wiki/ISO_3166-1_alpha-2
- Test with ONE_OF operator for multiple regions

---

## Regex Issues

### Issue: Regex pattern not matching

**Common Problems:**

1. **Escaping**
   - JSON requires double escaping: `\\d` not `\d`
   - Period needs escape: `\\.` not `.`

2. **Pattern too strict/loose**
   - Test regex independently first
   - Use regex101.com for testing

**Example - PO Box Pattern:**
```json
{
  "value": "([Pp][.\\s]*[Oo][.\\s]?\\s*[Bb][Oo][Xx])"
}
```

Note: `\\s` in JSON becomes `\s` in regex.

---

## Metafield Issues

### Issue: Metafield condition not working

**Common Problems:**

1. **Namespace/key mismatch**
   - Verify exact namespace and key
   - Check metafield exists on product/customer

2. **Value type**
   - Metafield value is always string
   - Numeric comparison requires proper operator

3. **Input configuration**
   - Namespace and key must be in input object

**Solution:**
```json
{
  "configuration": {
    "conditions": [{
      "property": "PRODUCT_METAFIELD",
      "operator": "METAFIELD_VALUE_EQUALS",
      "value": {
        "namespace": "custom",
        "key": "restricted",
        "value": "true"
      }
    }]
  },
  "input": {
    "productMetafieldNamespace": "custom",
    "productMetafieldKey": "restricted"
  }
}
```

---

## Time-Based Rules Issues

### Issue: Time rule triggers at wrong time

**Common Problems:**

1. **Timezone**
   - Rules use shop timezone
   - Not customer's timezone

2. **Format**
   - Time: `HH:MM` (24-hour format)
   - Date: `YYYY-MM-DD`
   - DateTime: `YYYY-MM-DDTHH:MM`

**Solution:**
- Verify shop timezone settings
- Use 24-hour format for times

---

## Reduction Rules Issues (Shopify Plus)

### Issue: Gift card/discount rules not working

**Common Problems:**

1. **Not Shopify Plus**
   - REDUCTION type requires Plus plan

2. **Metadata not captured**
   - BoomGate metadata extension must be installed
   - Extension captures gift card/discount data

**Solution:**
- Verify Shopify Plus subscription
- Check extension is enabled in checkout

---

## AND Conditions Issues

### Issue: AND conditions not evaluating correctly

**Common Problems:**

1. **Wrong nesting level**
   - AND conditions only check matching lines
   - Parent condition must find lines first

2. **Property not line-level**
   - Some properties can't be used in AND conditions
   - Cart-level properties don't support AND

**Correct Pattern:**
```json
{
  "conditions": [{
    "property": "PRODUCT_TAGS",
    "operator": "HAS_TAGS",
    "value": { "value": ["limited"] },
    "andConditions": [{
      "property": "LINE_QUANTITY",
      "operator": "GREATHER_THAN",
      "value": { "value": "1" }
    }]
  }]
}
```

---

## Performance Issues

### Issue: Slow checkout

**Possible Causes:**

1. **Too many rules**
   - Each enabled rule runs at checkout
   - Consolidate similar rules

2. **Complex conditions**
   - Many metafield queries
   - Large tag lists

**Solutions:**
- Disable unused rules
- Use specific tags instead of metafields
- Combine similar rules with ANY mode

---

## Debugging Steps

### Step 1: Verify Rule Configuration

1. Open rule in BoomGate app
2. Check all conditions
3. Verify input matches conditions
4. Test mode setting

### Step 2: Check Shopify Functions Logs

1. Go to Shopify Admin
2. Settings → Checkout → Customizations
3. View function logs

### Step 3: Test with Simple Rule

1. Create minimal rule with one condition
2. Test checkout
3. Add complexity gradually

### Step 4: Verify Data

1. Check actual product tags
2. Verify customer tags
3. Confirm address values
4. Check metafield existence

---

## Getting Help

If issues persist:

1. **Document the rule configuration**
   - Export rule JSON
   - Note validation type

2. **Capture test case**
   - Cart contents
   - Customer state
   - Expected vs actual behavior

3. **Contact Support**
   - Email: support@code57.pl
   - Include rule configuration and test case

---
title: 3. API Reference
description: Complete REST API documentation for the Revendo order finalization endpoint
published: true
date: 2025-10-17T13:36:28.628Z
tags: api, rest, endpoint, authentication, integration
editor: markdown
dateCreated: 2025-10-15T12:45:53.933Z
---

# Revendo Trade-In API Reference

## Endpoint

```
POST https://purchase.revendo.dev/wp-json/revendo/v1/orders/finalize
```

## Authentication

```
Header: X-Revendo-API-Key: your_api_key_here
```

Contact Revendo to get your API key.

## Request Format

```json
{
  "offer_id": "11153290",
  "partner_order_id": "YOUR-ORDER-12345",
  "customer": {
    "email": "customer@example.com",
    "first_name": "Max",
    "last_name": "Müller",
    "phone": "+41791234567",
    "iban": "CH9300762011623852957",
    "address": {
      "street": "Bahnhofstrasse 1",
      "zip": "8001",
      "city": "Zürich",
      "country": "CH"
    }
  },
  "shipping_method": "post"
}
```

### Required Fields

| Field | Description |
|-------|-------------|
| `offer_id` | From the iframe PostMessage event (`event.data.details.offer_id`) |
| `partner_order_id` | Your order ID (used to prevent duplicates) |
| `customer.email` | Customer email address |
| `customer.first_name` | Customer first name |
| `customer.last_name` | Customer last name |
| `customer.phone` | Customer phone number |
| `customer.iban` | Customer IBAN (required for bank transfer) |
| `customer.address.street` | Street address |
| `customer.address.zip` | ZIP/Postal code |
| `customer.address.city` | City name |
| `customer.address.country` | Country code (must be "CH" for Switzerland) |
| `shipping_method` | Shipping method (use `"post"`) |

### Important Notes

- **Account owner** is automatically set as `first_name + last_name`
- **Terms acceptance** is automatically confirmed for API orders
- **Data deletion** is automatically confirmed for API orders
- **No donation option** for API orders (automatic bank transfer)

## Response Format

### Success (200 OK)

```json
{
  "success": true,
  "order_id": "W-0105185",
  "status": "pending_device",
  "quote_amount": 145.00,
  "currency": "CHF",
  "created_at": "2025-10-17T13:19:47+00:00"
}
```

### Duplicate Order (409 Conflict)

If the same `partner_order_id` is submitted twice, the API returns the existing order:

```json
{
  "success": true,
  "message": "Order already exists",
  "order_id": "W-0105185",
  "status": "pending_device",
  "quote_amount": 145.00,
  "currency": "CHF",
  "created_at": "2025-10-17 15:19:45"
}
```

**Note:** HTTP 409 is not an error - it means the order was already created successfully.

### Error Response

```json
{
  "code": "error_code",
  "message": "Human readable error message",
  "data": {
    "status": 400
  }
}
```

## Error Codes

| HTTP | Code | Meaning | What to do |
|------|------|---------|------------|
| 400 | `invalid_request` | Missing or invalid field | Check required fields |
| 400 | `offer_expired` | Quote older than 60 minutes | Customer needs new quote |
| 401 | `unauthorized` | Wrong/missing API key | Check your API key |
| 404 | `offer_not_found` | offer_id doesn't exist | Customer needs new quote |
| 409 | _(success)_ | Order already created | **This is OK** - use existing order |
| 500 | `internal_error` | Server error | Retry after 1-2 seconds |

### Common Error Examples

**Missing IBAN:**
```json
{
  "code": "invalid_request",
  "message": "Missing required fields: IBAN",
  "data": {
    "status": 400
  }
}
```

**Expired Offer:**
```json
{
  "code": "offer_expired",
  "message": "Quote has expired (>60 minutes old)",
  "data": {
    "status": 400
  }
}
```

**Invalid API Key:**
```json
{
  "code": "unauthorized",
  "message": "Invalid API key",
  "data": {
    "status": 401
  }
}
```

## Code Examples

### cURL
```bash
curl -X POST https://purchase.revendo.dev/wp-json/revendo/v1/orders/finalize \
  -H "X-Revendo-API-Key: your_key" \
  -H "Content-Type: application/json" \
  -d '{
    "offer_id": "11153290",
    "partner_order_id": "ORDER-123",
    "customer": {
      "email": "customer@example.com",
      "first_name": "Max",
      "last_name": "Müller",
      "phone": "+41791234567",
      "iban": "CH9300762011623852957",
      "address": {
        "street": "Bahnhofstrasse 1",
        "zip": "8001",
        "city": "Zürich",
        "country": "CH"
      }
    },
    "shipping_method": "post"
  }'
```

### PHP (WooCommerce)
```php
$response = wp_remote_post('https://purchase.revendo.dev/wp-json/revendo/v1/orders/finalize', [
  'headers' => [
    'X-Revendo-API-Key' => get_option('revendo_api_key'),
    'Content-Type' => 'application/json'
  ],
  'body' => json_encode([
    'offer_id' => $offer_id,
    'partner_order_id' => $order->get_order_number(),
    'customer' => [
      'email' => $order->get_billing_email(),
      'first_name' => $order->get_billing_first_name(),
      'last_name' => $order->get_billing_last_name(),
      'phone' => $order->get_billing_phone(),
      'iban' => $order->get_meta('billing_iban'),
      'address' => [
        'street' => $order->get_billing_address_1(),
        'zip' => $order->get_billing_postcode(),
        'city' => $order->get_billing_city(),
        'country' => $order->get_billing_country()
      ]
    ],
    'shipping_method' => 'post'
  ]),
  'timeout' => 30
]);

if (!is_wp_error($response)) {
  $status = wp_remote_retrieve_response_code($response);
  $data = json_decode(wp_remote_retrieve_body($response), true);

  if ($status == 200 || $status == 409) {
    // Success or duplicate (both OK)
    $order->update_meta_data('revendo_order_id', $data['order_id']);
    $order->add_order_note('Revendo trade-in order: ' . $data['order_id']);
    $order->save();
  } else {
    error_log('Revendo API error: ' . $data['message']);
  }
}
```

### Node.js (Express)
```javascript
const axios = require('axios');

try {
  const response = await axios.post(
    'https://purchase.revendo.dev/wp-json/revendo/v1/orders/finalize',
    {
      offer_id: offerId,
      partner_order_id: order.id,
      customer: {
        email: order.customer.email,
        first_name: order.customer.firstName,
        last_name: order.customer.lastName,
        phone: order.customer.phone,
        iban: order.customer.iban,
        address: {
          street: order.billing.street,
          zip: order.billing.zip,
          city: order.billing.city,
          country: 'CH'
        }
      },
      shipping_method: 'post'
    },
    {
      headers: {
        'X-Revendo-API-Key': process.env.REVENDO_API_KEY,
        'Content-Type': 'application/json'
      },
      timeout: 30000
    }
  );

  // Success (200) or duplicate (409) - both OK
  if (response.status === 200 || response.status === 409) {
    await order.update({
      revendo_order_id: response.data.order_id,
      revendo_quote_amount: response.data.quote_amount
    });
  }
} catch (error) {
  console.error('Revendo API error:', error.response?.data || error.message);
}
```

### Python (Django/Flask)
```python
import requests

try:
    response = requests.post(
        'https://purchase.revendo.dev/wp-json/revendo/v1/orders/finalize',
        json={
            'offer_id': offer_id,
            'partner_order_id': order.id,
            'customer': {
                'email': customer.email,
                'first_name': customer.first_name,
                'last_name': customer.last_name,
                'phone': customer.phone,
                'iban': customer.iban,
                'address': {
                    'street': billing.street,
                    'zip': billing.zip,
                    'city': billing.city,
                    'country': 'CH'
                }
            },
            'shipping_method': 'post'
        },
        headers={
            'X-Revendo-API-Key': settings.REVENDO_API_KEY,
            'Content-Type': 'application/json'
        },
        timeout=30
    )

    if response.status_code in [200, 409]:
        data = response.json()
        order.revendo_order_id = data['order_id']
        order.save()
except requests.exceptions.RequestException as e:
    logger.error(f'Revendo API error: {e}')
```

## Important Implementation Notes

### About Duplicates (HTTP 409)

A 409 response means the order already exists (you probably retried). **This is NOT an error!** Just use the existing order data.

```javascript
if (response.status === 409) {
  // Order was already created successfully
  console.log('Using existing order:', response.data.order_id);
  // Continue with your normal success flow
}
```

### About Retries

**Only retry on 500 errors.** Use exponential backoff:

```javascript
async function callRevendoAPI(data, retries = 3) {
  for (let i = 0; i < retries; i++) {
    try {
      return await axios.post(endpoint, data, config);
    } catch (error) {
      const status = error.response?.status;

      // Only retry on 500 errors
      if (status === 500 && i < retries - 1) {
        await sleep(1000 * Math.pow(2, i)); // 1s, 2s, 4s
        continue;
      }

      throw error;
    }
  }
}
```

**Do NOT retry:**
- 400 errors (invalid request)
- 401 errors (bad API key)
- 404 errors (offer not found)
- 409 errors (duplicate - already success!)

### About Quote Expiration

Quotes expire **60 minutes** after creation. If you get `offer_expired` error:

```javascript
if (error.response?.data?.code === 'offer_expired') {
  alert('Your trade-in quote has expired. Please select your device again.');
  // Clear stored offer_id
  sessionStorage.removeItem('revendo_offer_id');
  // Redirect back to device selection
}
```

### About Country Validation

Only Switzerland (`"CH"`) is currently supported. The API will reject other countries:

```json
{
  "code": "invalid_request",
  "message": "Invalid customer data",
  "data": {
    "status": 400,
    "errors": ["We can only accept orders from Switzerland"]
  }
}
```

### Error Handling Best Practices

```javascript
try {
  const response = await callRevendoAPI(data);

  // Success or duplicate (both OK)
  if (response.status === 200 || response.status === 409) {
    return handleSuccess(response.data);
  }

} catch (error) {
  const status = error.response?.status;
  const code = error.response?.data?.code;

  switch (code) {
    case 'offer_expired':
    case 'offer_not_found':
      // Customer needs new quote
      return askCustomerToSelectDeviceAgain();

    case 'unauthorized':
      // API key problem
      console.error('Revendo API key invalid');
      return notifyAdmin();

    case 'invalid_request':
      // Missing/invalid fields
      console.error('Invalid data:', error.response.data.message);
      return checkRequiredFields();

    default:
      // Log error but don't fail checkout
      console.error('Revendo API error:', error);
      return completeCheckoutWithoutTradein();
  }
}
```

### Security Best Practices

- **Never expose API key** in frontend JavaScript
- **Store in environment variables** (`.env` file)
- **Use HTTPS only** (enforced by API)
- **Validate event.origin** when receiving iframe messages
- **Sanitize all customer inputs** before sending to API

## Testing

### Staging Environment

Use the staging endpoint for testing:

```
POST https://staging.revendo.dev/wp-json/revendo/v1/orders/finalize
```

Contact Revendo for a staging API key.

### Test Scenarios

1. **Successful order creation** - verify order_id is returned
2. **Duplicate prevention** - submit same order twice, verify 409 response
3. **Expired offer** - wait 61 minutes, verify 400 error
4. **Missing IBAN** - omit IBAN field, verify validation error
5. **Invalid API key** - use wrong key, verify 401 error

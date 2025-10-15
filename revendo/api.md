---
title: 3. API Reference
description: Complete REST API documentation for the Revendo order finalization endpoint
published: true
date: 2025-10-15T13:10:00.000Z
tags: api, rest, endpoint, authentication, integration
editor: markdown
dateCreated: 2025-10-15T12:46:00.000Z
---

# API Reference

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
  "offer_id": "11153269",
  "partner_order_id": "YOUR-ORDER-12345",
  "customer": {
    "email": "customer@example.com",
    "first_name": "Max",
    "last_name": "Müller",
    "phone": "+41791234567",
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
| `offer_id` | From the iframe PostMessage event (`event.data.details.price_id`) |
| `partner_order_id` | Your order ID (used to prevent duplicates) |
| `customer.email` | Customer email |
| `customer.first_name` | First name |
| `customer.last_name` | Last name |
| `customer.phone` | Phone number |
| `customer.address.*` | Full address |
| `shipping_method` | Use `"post"` for regular shipping |

## Response Format

### Success (200 OK)

```json
{
  "success": true,
  "order_id": 105030,
  "web_id": "ANK-2025-001234",
  "status": "pending_device",
  "quote_amount": 227.00,
  "currency": "CHF",
  "tracking_url": "https://revendo.ch/ankauf/tracking/?order=ANK-2025-001234",
  "created_at": "2025-10-07T10:30:45Z"
}
```

Store the `tracking_url` to show customers later.

## Error Codes

| Code | Error | Meaning | What to do |
|------|-------|---------|------------|
| 400 | `offer_expired` | Quote older than 60 minutes | Customer needs new quote |
| 401 | `unauthorized` | Wrong/missing API key | Check your API key |
| 404 | `offer_not_found` | offer_id doesn't exist | Check the offer_id from iframe |
| 409 | `duplicate_order` | Order already created | **This is OK** - use existing order |
| 500 | `internal_error` | Server error | Retry after 1-2 seconds |

### Error Response Example

```json
{
  "success": false,
  "error": "offer_expired",
  "message": "Quote has expired (>60 minutes old)",
  "code": 400
}
```

## Code Examples

### cURL
```bash
curl -X POST https://purchase.revendo.dev/wp-json/revendo/v1/orders/finalize \
  -H "X-Revendo-API-Key: your_key" \
  -H "Content-Type: application/json" \
  -d '{
    "offer_id": "11153269",
    "partner_order_id": "ORDER-123",
    "customer": {...},
    "shipping_method": "post"
  }'
```

### PHP
```php
$response = wp_remote_post('https://purchase.revendo.dev/wp-json/revendo/v1/orders/finalize', [
  'headers' => [
    'X-Revendo-API-Key' => $api_key,
    'Content-Type' => 'application/json'
  ],
  'body' => json_encode($data),
  'timeout' => 30
]);

$body = json_decode(wp_remote_retrieve_body($response), true);
$status = wp_remote_retrieve_response_code($response);

if ($status == 200 || $status == 409) {
  // Success or duplicate (both OK)
  $tracking_url = $body['tracking_url'];
}
```

### Node.js
```javascript
const response = await axios.post(
  'https://purchase.revendo.dev/wp-json/revendo/v1/orders/finalize',
  data,
  {
    headers: {
      'X-Revendo-API-Key': process.env.REVENDO_API_KEY,
      'Content-Type': 'application/json'
    },
    timeout: 30000
  }
);

console.log('Tracking URL:', response.data.tracking_url);
```

### Python
```python
response = requests.post(
    'https://purchase.revendo.dev/wp-json/revendo/v1/orders/finalize',
    json=data,
    headers={
        'X-Revendo-API-Key': api_key,
        'Content-Type': 'application/json'
    },
    timeout=30
)

if response.status_code in [200, 409]:
    tracking_url = response.json()['tracking_url']
```

## Important Notes

### About Duplicates (409)

A 409 error means the order already exists (you probably retried). **This is not a problem!** Just use the existing order data from the response.

```javascript
if (response.status === 409) {
  // Order was already created, use it
  const existingOrder = response.data.existing_order;
  console.log('Using existing order:', existingOrder.web_id);
}
```

### About Retries

Only retry on 500 errors. Use exponential backoff:

```javascript
async function callAPI(data, retries = 3) {
  for (let i = 0; i < retries; i++) {
    try {
      return await makeAPICall(data);
    } catch (error) {
      if (error.response?.status === 500 && i < retries - 1) {
        await sleep(1000 * Math.pow(2, i)); // 1s, 2s, 4s
        continue;
      }
      throw error;
    }
  }
}
```

### About Quote Expiration

Quotes expire 60 minutes after creation. If you get a 400 error, show the customer:

```
"Your trade-in quote has expired. Please select your device again to get a new quote."
```

### Security

- **Never expose API key** in frontend code
- **Store in environment variables** on backend
- **Use HTTPS only**

## Testing

Test with the provided `iframeTest.html` file to see how events work, then test your API integration in a staging environment.

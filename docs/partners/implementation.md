---
title: 2. Implementation Guide
description: Step-by-step guide to integrating Revendo trade-in system into your e-commerce platform
published: true
date: 2025-10-20T13:27:51.927Z
tags: api, integration, implementation, iframe, postmessage
editor: markdown
dateCreated: 2025-10-15T12:45:50.633Z
---

# Revendo Trade-In Integration Guide

## Frontend: Iframe + Events

### 1. Add iframe to your checkout page

```html
<div class="trade-in-section">
  <h3>Trade in your old device</h3>
  <iframe
    id="revendo-iframe"
    src="https://purchase.revendo.dev/sell-a-device-iframe/?iframe=1"
    style="width: 100%; border: none; min-height: 600px;">
  </iframe>
</div>
```

### 2. Add event listener (copy this exactly)

```javascript
// Enable auto-resize and auto-scroll features
let autoResize = true;
let autoScroll = true;

// Get iframe reference
const iframe = document.getElementById('revendo-iframe');

// Configure iframe scroll behavior
// Set to true for parent scrolling, false for internal iframe scrolling
iframe.addEventListener('load', function() {
  iframe.contentWindow.rva_parent_scroll_enabled = autoScroll;
});

window.addEventListener('message', function(event) {
  // Security check - very important!
  if (event.origin !== 'https://purchase.revendo.dev') return;

  // Handle auto-resize
  if (autoResize && event.data.type === 'revendo_resize') {
    const iframe = document.getElementById('revendo-iframe');
    const height = parseInt(event.data.height);
    if (height && !isNaN(height)) {
      iframe.style.height = Math.max(600, height) + 'px';
    }
  }

  // Handle auto-scroll to iframe top (after completing questions/adding to cart)
  if (autoScroll && event.data.type === 'revendo_scroll_to_top') {
    const iframe = document.getElementById('revendo-iframe');
    const iframeRect = iframe.getBoundingClientRect();
    const scrollTop = window.pageYOffset || document.documentElement.scrollTop;
    const iframeTop = iframeRect.top + scrollTop;

    window.scrollTo({
      top: iframeTop,
      behavior: 'smooth'
    });
  }

  // Handle auto-scroll to specific question (while answering questions)
  if (autoScroll && event.data.type === 'revendo_scroll_to_position') {
    const position = parseInt(event.data.position);
    if (position && !isNaN(position)) {
      const iframe = document.getElementById('revendo-iframe');
      const iframeRect = iframe.getBoundingClientRect();
      const scrollTop = window.pageYOffset || document.documentElement.scrollTop;
      const iframeTop = iframeRect.top + scrollTop;

      // Calculate absolute position: iframe top + position within iframe
      const absolutePosition = iframeTop + position;

      window.scrollTo({
        top: absolutePosition,
        behavior: 'smooth'
      });
    }
  }

  // Handle device selected
  if (event.data.type === 'revendo_device_selected') {
    const details = event.data.details;

    // Store for later API call
    sessionStorage.setItem('revendo_offer_id', details.offer_id);
    sessionStorage.setItem('revendo_credit', details.price.final_cash);
    sessionStorage.setItem('revendo_device', details.product.name);

    // Update your checkout UI
    showTradeInInCheckout(details.product.name, details.price.final_cash);
  }

  // Handle device removed
  if (event.data.type === 'revendo_remove_device_selected') {
    sessionStorage.removeItem('revendo_offer_id');
    sessionStorage.removeItem('revendo_credit');
    sessionStorage.removeItem('revendo_device');

    hideTradeInInCheckout();
  }
});
```

**Important:** Use `details.offer_id` (not `details.price_id`) - this is the permanent ID for API calls.

### 3. PostMessage Events Reference

#### `revendo_resize`
**When:** Iframe content height changes (questions appear/disappear, animations complete)

**Data:**
```javascript
{
  type: 'revendo_resize',
  height: 1450  // Height in pixels
}
```

**Your Action:** Resize iframe to fit content
```javascript
iframe.style.height = event.data.height + 'px';
```

---

#### `revendo_scroll_to_top`
**When:**
- All questions answered (showing price)
- Device added to cart
- Navigation to important section

**Data:**
```javascript
{
  type: 'revendo_scroll_to_top',
  timestamp: '2025-10-20T14:30:00.000Z'
}
```

**Your Action:** Scroll page to iframe top
```javascript
const iframe = document.getElementById('revendo-iframe');
const iframeTop = iframe.getBoundingClientRect().top + window.pageYOffset;
window.scrollTo({ top: iframeTop, behavior: 'smooth' });
```

---

#### `revendo_scroll_to_position`
**When:** User answers a question and moves to the next one

**Data:**
```javascript
{
  type: 'revendo_scroll_to_position',
  position: 317,  // Position within iframe (pixels from iframe top)
  timestamp: '2025-10-20T14:30:00.000Z'
}
```

**Your Action:** Scroll page to show that position within iframe
```javascript
const iframe = document.getElementById('revendo-iframe');
const iframeTop = iframe.getBoundingClientRect().top + window.pageYOffset;
const absolutePosition = iframeTop + event.data.position;
window.scrollTo({ top: absolutePosition, behavior: 'smooth' });
```

**Why:** Creates smooth question-by-question navigation experience. Each question scrolls into view as user progresses through the form.

---

#### `revendo_device_selected`
**When:** Customer adds device to cart (after answering all questions)

**Data:**
```javascript
{
  type: 'revendo_device_selected',
  details: {
    offer_id: '11153269',        // Use this for API call
    price_id: '123-tmp',         // Session ID (internal use only)
    product: {
      id: 123,
      name: 'iPhone 13 Pro 128GB',
      quantity: 1
    },
    price: {
      base: 450.00,
      final_cash: 450.00,
      final_coupon: 500.00
    },
    deductions: [
      { label: 'Screen damage', amount: -50 }
    ],
    bonuses: [
      { label: 'Original packaging', amount: 20 }
    ],
    coupons: [
      { code: 'SUMMER20', amount: 50 }
    ]
  },
  timestamp: '2025-10-20T14:30:00.000Z'
}
```

**Your Action:**
1. Store `offer_id` for API call
2. Apply credit to checkout
3. Show device in order summary

---

#### `revendo_remove_device_selected`
**When:** Customer removes device from cart

**Data:**
```javascript
{
  type: 'revendo_remove_device_selected',
  price_id: '11153269',
  timestamp: '2025-10-20T14:30:00.000Z'
}
```

**Your Action:**
1. Remove stored offer_id
2. Remove credit from checkout
3. Remove device from order summary

---

### 4. Update your checkout UI

```javascript
function showTradeInInCheckout(deviceName, credit) {
  // Show the trade-in in your checkout summary
  const summaryHTML = `
    <div class="trade-in-item">
      <span>Trade-in: ${deviceName}</span>
      <span class="credit">-CHF ${credit.toFixed(2)}</span>
    </div>
  `;
  document.getElementById('checkout-summary').insertAdjacentHTML('beforeend', summaryHTML);

  // Update order total
  const currentTotal = parseFloat(document.getElementById('order-total').textContent);
  const newTotal = Math.max(0, currentTotal - credit);
  document.getElementById('order-total').textContent = newTotal.toFixed(2);
}

function hideTradeInInCheckout() {
  // Remove trade-in from display and recalculate
  const tradeInElement = document.querySelector('.trade-in-item');
  if (tradeInElement) tradeInElement.remove();
  // Recalculate your order total
}
```

### 5. Features Configuration

#### Auto-Resize
Automatically adjusts iframe height to fit content (no scrollbar needed).

**Enable:**
```javascript
let autoResize = true;
```

**Disable:**
```javascript
let autoResize = false;
// Set fixed height instead
iframe.style.height = '800px';
```

#### Auto-Scroll
Automatically scrolls parent page to show relevant sections:
- While answering questions: scrolls to each next question
- After completion: scrolls to show price/cart

**Enable:**
```javascript
let autoScroll = true;

// Configure iframe on load
iframe.addEventListener('load', function() {
  iframe.contentWindow.postMessage({
    type: 'revendo_configure',
    autoScroll: true
  }, 'https://purchase.revendo.dev');
});
```

**Disable:**
```javascript
let autoScroll = false;

// Configure iframe on load (optional - false is the default)
iframe.addEventListener('load', function() {
  iframe.contentWindow.postMessage({
    type: 'revendo_configure',
    autoScroll: false
  }, 'https://purchase.revendo.dev');
});
```

**Important:**
- When set to `false` (or not configured), the iframe scrolls internally like a normal page
- When set to `true`, the parent page scrolls to show content as user progresses through questions
- Must be sent via postMessage due to cross-origin security restrictions
- Configuration should be sent on iframe load/reload

**Both features work together:** Iframe expands to full height (auto-resize) while page scrolls to show current question (auto-scroll).

## Backend: API Call

### When customer completes checkout

Call this from your backend after creating the order:

**Endpoint:** `POST https://purchase.revendo.dev/wp-json/revendo/v1/orders/finalize`

**Headers:**
- `X-Revendo-API-Key: your_api_key_here`
- `Content-Type: application/json`

**Required Fields:**
```json
{
  "offer_id": "string (from postMessage event)",
  "partner_order_id": "string (your unique order ID)",
  "shipping_method": "post",
  "customer": {
    "email": "string",
    "first_name": "string",
    "last_name": "string",
    "phone": "string",
    "iban": "string (required for bank transfer)",
    "address": {
      "street": "string",
      "city": "string",
      "zip": "string",
      "country": "CH"
    }
  }
}
```

**Success Response (HTTP 200):**
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

**Duplicate Order Response (HTTP 409):**
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

**Error Response (HTTP 400/404/500):**
```json
{
  "code": "error_code",
  "message": "Human readable error message",
  "data": {
    "status": 400
  }
}
```

### PHP Example (WooCommerce)

```php
$offer_id = $_SESSION['revendo_offer_id'] ?? null;

if ($offer_id) {
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
        'iban' => $order->get_meta('billing_iban'), // Your IBAN field
        'address' => [
          'street' => $order->get_billing_address_1(),
          'zip' => $order->get_billing_postcode(),
          'city' => $order->get_billing_city(),
          'country' => $order->get_billing_country() // Must be 'CH'
        ]
      ],
      'shipping_method' => 'post'
    ])
  ]);

  if (!is_wp_error($response)) {
    $status_code = wp_remote_retrieve_response_code($response);
    $data = json_decode(wp_remote_retrieve_body($response), true);

    if ($status_code == 200 || $status_code == 409) {
      // Success or duplicate (both OK)
      $order->update_meta_data('revendo_order_id', $data['order_id']);
      $order->add_order_note('Revendo trade-in order created: ' . $data['order_id']);
      $order->save();
    } else {
      // Log error but don't fail checkout
      error_log('Revendo API error: ' . $data['message']);
    }
  }
}
```

### Node.js Example

```javascript
const offerId = req.session.revendo_offer_id;

if (offerId) {
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
            country: 'CH' // Must be Switzerland
          }
        },
        shipping_method: 'post'
      },
      {
        headers: {
          'X-Revendo-API-Key': process.env.REVENDO_API_KEY,
          'Content-Type': 'application/json'
        }
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
    // Log error but don't fail checkout
    console.error('Revendo API error:', error.response?.data || error.message);
  }
}
```

## Important Notes

### Offer Expiration
- **Quotes expire after 60 minutes** - customer needs a new quote if they wait too long
- API will return `offer_expired` error (HTTP 400) if offer is too old
- API will return `offer_not_found` error (HTTP 404) if offer doesn't exist

### IBAN Requirement
- **IBAN is required** for all API orders (used for bank transfer)
- Customer's full name (first + last) is automatically used as account owner
- No donation option for API orders (automatic bank transfer)

### Storage
- **Use sessionStorage** - survives page refresh during checkout
- Store `offer_id` (not `price_id`) from the postMessage event
- Clear storage after successful order

### Error Handling
- **Don't fail checkout** if Revendo API fails
- Complete your main order and log the Revendo error
- Customer can still contact support to add trade-in later

### Security
- **Always verify** `event.origin` is exactly `https://purchase.revendo.dev`
- **Never expose** your API key in frontend JavaScript
- **Only call API** from your secure backend

### Duplicate Prevention
- API uses `partner_order_id` to prevent duplicate orders
- If same `partner_order_id` is sent twice, API returns existing order with HTTP 409
- This is safe to handle as success (not an error)

## Scrolling Behavior

### Three Configuration Modes

The iframe supports three distinct scrolling configurations:

#### 1. Both Features Enabled (Auto-Resize + Auto-Scroll)
**Configuration:**
```javascript
let autoResize = true;
let autoScroll = true;

iframe.contentWindow.postMessage({
  type: 'revendo_configure',
  autoScroll: true
}, 'https://purchase.revendo.dev');
```

**Behavior:**
- Iframe expands to show all content (no internal scrollbar)
- Parent page scrolls to bring relevant sections into view
- **Question navigation:** Parent page scrolls to each next question
- **Completion:** Parent page scrolls to iframe top to show price/cart
- Uses `revendo_scroll_to_position` and `revendo_scroll_to_top` events

**Example Flow:**
```
User selects iPhone 13 Pro
→ Iframe resizes to 2057px (shows all 7 questions)
→ User answers Question 1
→ Parent page scrolls to Question 2 position within iframe
→ User answers Question 2
→ Parent page scrolls to Question 3 position
→ ... continues for all questions ...
→ After Question 7 answered
→ Parent page scrolls to iframe top (shows price section)
```

#### 2. Auto-Resize Only
**Configuration:**
```javascript
let autoResize = true;
let autoScroll = false;

// No postMessage needed (false is default)
// Or explicitly send:
iframe.contentWindow.postMessage({
  type: 'revendo_configure',
  autoScroll: false
}, 'https://purchase.revendo.dev');
```

**Behavior:**
- Iframe expands to show all content
- Iframe scrolls internally like a normal page (smooth internal scrolling)
- No parent page scrolling
- No postMessage scroll events sent

#### 3. Both Features Disabled
**Configuration:**
```javascript
let autoResize = false;
let autoScroll = false;

// Set fixed height
iframe.style.height = '800px';
```

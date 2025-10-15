# Implementation Guide

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
window.addEventListener('message', function(event) {
  // Security check - very important!
  if (event.origin !== 'https://purchase.revendo.dev') return;
  
  // Handle resize
  if (event.data.type === 'revendo_resize') {
    const iframe = document.getElementById('revendo-iframe');
    iframe.style.height = event.data.height + 'px';
  }
  
  // Handle device selected
  if (event.data.type === 'revendo_device_selected') {
    const details = event.data.details;
    
    // Store for later API call
    sessionStorage.setItem('revendo_offer_id', details.price_id);
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

### 3. Update your checkout UI

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

## Backend: API Call

### When customer completes checkout

Call this from your backend after creating the order:

**PHP Example:**
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
        'address' => [
          'street' => $order->get_billing_address_1(),
          'zip' => $order->get_billing_postcode(),
          'city' => $order->get_billing_city(),
          'country' => $order->get_billing_country()
        ]
      ],
      'shipping_method' => 'post'
    ])
  ]);
  
  if (!is_wp_error($response) && wp_remote_retrieve_response_code($response) == 200) {
    $data = json_decode(wp_remote_retrieve_body($response), true);
    // Store tracking URL for customer
    $order->update_meta_data('revendo_tracking_url', $data['tracking_url']);
    $order->save();
  }
}
```

**Node.js Example:**
```javascript
const offerId = req.session.revendo_offer_id;

if (offerId) {
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
        address: {
          street: order.billing.street,
          zip: order.billing.zip,
          city: order.billing.city,
          country: order.billing.country
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
  
  // Store tracking URL
  await order.update({ revendo_tracking_url: response.data.tracking_url });
}
```

## Important Notes

- **Quotes expire after 60 minutes** - customer needs new quote if they wait too long
- **Use sessionStorage** - survives page refresh
- **Don't fail checkout** - if Revendo API fails, complete the main order anyway and log the error
- **Security** - always verify `event.origin` is exactly `https://purchase.revendo.dev`

## Testing

1. Open the included `iframeTest.html` in your browser
2. Select a device
3. Watch the events appear in the log
4. Verify offer_id is captured correctly

## That's it!

You now have a working integration. See API Documentation for full API details.

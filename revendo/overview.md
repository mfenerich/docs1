# Revendo Trade-In Integration

## What is it?

Add a trade-in calculator to your checkout. Customers select their old device, get an instant quote, and the credit is applied to their purchase.

## How it works

1. Customer adds new product to cart
2. Trade-in iframe appears in checkout
3. Customer selects their old device and answers questions
4. Instant quote provided (e.g., CHF 237)
5. Credit shows in checkout summary
6. Customer completes purchase
7. Your system calls Revendo API to create the trade-in order
8. Customer receives shipping label via email

## What you need

- **API Key**: Contact Revendo to get yours
- **HTTPS Website**: Required for iframe security
- **Backend API Access**: To call Revendo when order completes

## Three simple steps

### Step 1: Add the iframe

```html
<iframe 
  id="revendo-iframe"
  src="https://purchase.revendo.dev/sell-a-device-iframe/?iframe=1"
  style="width: 100%; border: none;">
</iframe>
```

### Step 2: Listen for events

```javascript
window.addEventListener('message', function(event) {
  if (event.origin !== 'https://purchase.revendo.dev') return;
  
  // Device selected - store the offer ID
  if (event.data.type === 'revendo_device_selected') {
    const offerId = event.data.details.price_id;
    const credit = event.data.details.price.final_cash;
    
    sessionStorage.setItem('revendo_offer_id', offerId);
    sessionStorage.setItem('revendo_credit', credit);
    
    // Update your checkout UI to show the credit
    updateCheckout(credit);
  }
  
  // Auto-resize the iframe
  if (event.data.type === 'revendo_resize') {
    document.getElementById('revendo-iframe').style.height = event.data.height + 'px';
  }
});
```

### Step 3: Call the API when checkout completes

```javascript
const response = await fetch('https://purchase.revendo.dev/wp-json/revendo/v1/orders/finalize', {
  method: 'POST',
  headers: {
    'X-Revendo-API-Key': 'YOUR_API_KEY',
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({
    offer_id: sessionStorage.getItem('revendo_offer_id'),
    partner_order_id: yourOrderId,
    customer: {
      email: customerEmail,
      first_name: firstName,
      last_name: lastName,
      phone: phone,
      address: {
        street: street,
        zip: zipCode,
        city: city,
        country: 'CH'
      }
    },
    shipping_method: 'post'
  })
});
```

## That's it!

The integration is complete. Test using the included `iframeTest.html` file.

## Next steps

- **Implementation details**: See Implementation Guide
- **API reference**: See API Documentation
- **Issues?**: See Troubleshooting

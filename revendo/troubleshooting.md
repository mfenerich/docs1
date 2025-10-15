---
title: 4. Troubleshooting
description: Common issues, debugging tips, and solutions for Revendo integration problems
published: true
date: 2025-10-15T13:15:00.000Z
tags: troubleshooting, debugging, errors, support, faq
editor: markdown
dateCreated: 2025-10-15T12:47:00.000Z
---

# Troubleshooting

## Iframe Issues

### Iframe not showing

**Check:**
- URL includes `?iframe=1` parameter
- Website uses HTTPS (not HTTP)
- No ad blockers interfering

**Fix:**
```html
<!-- Correct URL with parameter -->
<iframe src="https://purchase.revendo.dev/sell-a-device-iframe/?iframe=1"></iframe>
```

### Iframe not resizing

**Check browser console** - you should see messages like:
```
Message: https://purchase.revendo.dev {type: "revendo_resize", height: 1430}
```

If you see messages but iframe doesn't resize:

**Check origin verification:**
```javascript
// Must match EXACTLY
if (event.origin === 'https://purchase.revendo.dev') {
  // Not 'http://', not 'www.revendo.dev', not anything else
}
```

**Check iframe ID:**
```javascript
const iframe = document.getElementById('revendo-iframe'); // Must match your HTML
```

## Event Issues

### No events received

**Add debugging:**
```javascript
window.addEventListener('message', function(event) {
  console.log('Origin:', event.origin);
  console.log('Type:', event.data?.type);
  console.log('Data:', event.data);
});
```

Open browser console (F12) and interact with iframe. You should see logs.

If nothing appears:
- Event listener not attached
- JavaScript error preventing execution
- Wrong iframe URL

### Events received but not processed

Check your origin verification:
```javascript
if (event.origin !== 'https://purchase.revendo.dev') {
  console.warn('Blocked:', event.origin); // Add this to see what's blocked
  return;
}
```

## API Issues

### 400 - Offer Expired

**Reason:** Quote is older than 60 minutes

**Fix:** Customer needs a new quote. Show message:
```
"Your trade-in quote has expired. Please select your device again."
```

### 401 - Unauthorized

**Reason:** Wrong or missing API key

**Check:**
1. Header name is exactly `X-Revendo-API-Key`
2. API key is correct (no typos)
3. API key included in request

```javascript
// Correct header
headers: {
  'X-Revendo-API-Key': 'your_key_here'
}

// Common mistakes:
// 'Authorization': 'Bearer ...'  ❌ Wrong header
// 'X-API-Key': '...'             ❌ Wrong name
```

### 404 - Offer Not Found

**Reason:** Wrong `offer_id` sent to API

**Check:**
```javascript
// The offer_id comes from this event field:
const offerId = event.data.details.price_id; // ← This one

// Make sure you're storing it correctly:
sessionStorage.setItem('revendo_offer_id', offerId);

// And retrieving it correctly:
const storedId = sessionStorage.getItem('revendo_offer_id');
console.log('Sending offer_id:', storedId); // Debug this
```

### 409 - Duplicate Order

**This is OK!** It means the order already exists (from a previous request).

```javascript
if (response.status === 409) {
  // Not an error - just use the existing order
  const existingOrder = response.data.existing_order;
  console.log('Order already created:', existingOrder.web_id);
  // Continue normally
}
```

### 500 - Server Error

**Reason:** Temporary server issue

**Fix:** Retry with exponential backoff:
```javascript
async function callAPIWithRetry(data) {
  for (let attempt = 1; attempt <= 3; attempt++) {
    try {
      return await callAPI(data);
    } catch (error) {
      if (error.response?.status === 500 && attempt < 3) {
        await sleep(1000 * attempt); // Wait 1s, then 2s, then 3s
        continue;
      }
      throw error;
    }
  }
}
```

## Data Issues

### Trade-in lost on page refresh

**Reason:** Not using sessionStorage

**Fix:**
```javascript
// ❌ Wrong - lost on refresh
let offerId = '12345';

// ✅ Right - survives refresh
sessionStorage.setItem('revendo_offer_id', '12345');
```

### Trade-in showing on wrong order

**Reason:** Data not cleared after order completion

**Fix:**
```javascript
// After successful order creation
sessionStorage.removeItem('revendo_offer_id');
sessionStorage.removeItem('revendo_credit');
sessionStorage.removeItem('revendo_device');
```

## Quick Debug Checklist

When something's not working:

1. **Open browser console** (F12)
2. **Look for JavaScript errors** (red text)
3. **Check for PostMessage events**:
   ```javascript
   window.addEventListener('message', e => console.log(e.data));
   ```
4. **Verify iframe URL** includes `?iframe=1`
5. **Check origin verification** is exactly `https://purchase.revendo.dev`
6. **Test with included `iframeTest.html`** - if it works, compare to your code

## Still stuck?

1. Test with `iframeTest.html` - does it work?
2. Compare your code to the examples
3. Check browser console for errors
4. Contact Revendo support with:
   - Browser and version
   - Error messages from console
   - What you've tried

## Testing Tips

### Test the iframe alone first

Create a simple test page:
```html
<!DOCTYPE html>
<html>
<body>
  <iframe 
    id="revendo-iframe"
    src="https://purchase.revendo.dev/sell-a-device-iframe/?iframe=1"
    style="width: 100%; border: none; height: 800px;">
  </iframe>
  
  <script>
    window.addEventListener('message', e => {
      console.log('Event:', e.data.type);
      if (e.data.type === 'revendo_device_selected') {
        alert('Device selected! Offer ID: ' + e.data.details.price_id);
      }
    });
  </script>
</body>
</html>
```

Save as `test.html` and open in browser. If this works, the issue is in your integration.

### Use the provided test file

Open `iframeTest.html` in your browser. It shows:
- All PostMessage events in real-time
- Auto-resize working
- Event data structure

Use this as reference for your implementation.

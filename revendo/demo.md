---
title: 5. Interactive Demo
description: Live interactive testing tool for debugging Revendo iframe integration
published: true
date: 2025-10-16T08:22:39.251Z
tags: iframe, demo, testing, debug, interactive
editor: markdown
dateCreated: 2025-10-15T15:58:09.024Z
---

# Interactive Demo & Testing Tool

## Access the Live Demo

The interactive demo tool allows you to test the Revendo iframe integration in real-time and see all postMessage events as they happen.

### **[🚀 Open Interactive Demo Tool](https://iframeintegrationdemo.netlify.app/)**

> **Note**: The demo page opens in a new tab/window with full JavaScript functionality.

## What Does the Demo Do?

The demo page is a **developer testing tool** that provides:

### 1. **Real-Time Event Monitoring**
- Displays all `postMessage` events from the Revendo iframe
- Shows event types, data structure, and timestamps
- Color-coded event types for easy identification

### 2. **Interactive Features**
- **Reload Iframe**: Test iframe loading behavior
- **Open in Lightbox**: View iframe in modal overlay
- **Auto-Resize Toggle**: Enable/disable dynamic height adjustment
- **Clear Messages**: Reset event log

### 3. **Cross-Origin Testing**
- Verifies cross-origin iframe communication
- Shows session token management
- Displays origin verification status

### 4. **Debug Information**
- Parent page origin
- Iframe origin (https://purchase.revendo.dev)
- Cross-origin status
- Session management mode

## How to Use the Demo

### Step 1: Open the Demo
Click the link above to open the interactive demo in a new window.

### Step 2: Interact with the Iframe
1. Select a device category (e.g., "iPhone")
2. Choose your specific model
3. Answer condition questions
4. Get an instant quote

### Step 3: Watch the Events
As you interact, you'll see events appear in the **PostMessage Events** section:

- **`revendo_resize`** - When iframe content height changes
- **`revendo_device_selected`** - When customer selects a device
- **`revendo_remove_device_selected`** - When customer removes device
- **`revendo_cart_updated`** - When payment method changes

### Step 4: Inspect Event Data
Click on any event to expand and see the full data structure, including:
- `price_id` (offer_id for API)
- Product name and details
- Price information (cash vs voucher)
- Timestamp

### Step 5: Test Features
Use the test action buttons:
- **Reload Iframe**: Verify iframe loads correctly
- **Auto-Resize**: Enable to see dynamic height adjustment
- **Clear Messages**: Reset log to start fresh

## What to Look For

### ✅ Successful Integration Signs

1. **Iframe Loads**: The Revendo device selection interface appears
2. **Events Fire**: Messages appear in the event log when you interact
3. **Origin Verified**: Shows "✅ Active" for cross-origin status
4. **Data Structure**: Events contain expected fields (price_id, product, price)

### ❌ Potential Issues

1. **Iframe Doesn't Load**
   - Check browser console for errors
   - Verify URL includes `?iframe=1` parameter
   - Check for ad blockers

2. **No Events Appearing**
   - Origin verification may be blocking messages
   - JavaScript errors preventing event listener
   - Iframe not fully loaded

3. **Events Missing Data**
   - Customer hasn't completed selection flow
   - Temporary quote (price_id with `-tmp` suffix)

## Using Demo for Integration Testing

### Copy Event Handlers
The demo page source code contains working event handlers you can copy:

1. Right-click on the demo page
2. Select "View Page Source"
3. Find the `window.addEventListener('message', ...)` code
4. Copy and adapt for your integration

### Compare Event Data
Use the demo to:
- Verify the exact structure of event data
- Confirm `price_id` field location
- See all possible event types
- Test different user flows (select, remove, change payment)

### Test Edge Cases
- Selecting and removing devices multiple times
- Switching between cash and voucher payment
- Reloading the iframe during selection
- Long device names or special characters

## Demo vs Production Integration

### Demo Page Differences

| Feature | Demo Page | Your Production Site |
|---------|-----------|---------------------|
| **Purpose** | Testing & debugging | Actual customer checkout |
| **Event Display** | Visible log with details | Hidden, processed by your code |
| **Styling** | Basic test UI | Your branded design |
| **Data Storage** | Console only | sessionStorage + backend |
| **Order Creation** | Not connected | Calls Revendo API |

### Production Integration Steps

After testing with the demo:

1. **Embed iframe** in your checkout page
2. **Copy event listener** logic (adapt from demo source)
3. **Store offer data** in sessionStorage
4. **Update checkout UI** to show trade-in credit
5. **Call finalization API** after order completion

See [Implementation Guide](implementation) for complete production integration steps.

## Troubleshooting with the Demo

### If Demo Works But Your Site Doesn't

**Check these differences:**
- Origin verification code (exact match required)
- Event listener placement (must be before iframe loads)
- sessionStorage usage (check browser privacy settings)
- Console errors (open browser DevTools)

### If Demo Doesn't Work

**Common causes:**
- Ad blocker blocking iframe
- Browser privacy settings blocking cross-origin cookies
- Corporate firewall blocking revendo.dev domain
- JavaScript disabled in browser

### Browser Console Commands

Open browser console (F12) and try:

```javascript
// Check if event listener is attached
window.addEventListener('message', (e) => {
  console.log('Event received:', e.data);
});

// Verify origin
console.log('Current origin:', window.location.origin);

// Check sessionStorage
console.log('Stored data:', sessionStorage);
```

## Demo Source Code

The demo page is a **standalone HTML file** located at `demo/index.html` in the repository.

### Key Features in the Code:
- Complete postMessage event handling
- Origin verification example
- Event logging and display
- Auto-resize implementation
- Lightbox modal for iframe

You can use this as a reference implementation for your integration.

## Download for Local Testing

To test locally:

1. Download `demo/index.html` from the repository
2. Open directly in your web browser (no server needed)
3. Works with `file://` protocol for testing

Or host on your local development server:
```bash
# Python
python -m http.server 8000

# Node.js
npx http-server

# PHP
php -S localhost:8000
```

Then visit: `http://localhost:8000/demo/index.html`

## Next Steps

After testing with the demo:

1. **Understand the events** - You've seen them in action
2. **Copy the patterns** - Use similar code structure
3. **Implement in your site** - Follow [Implementation Guide](https://revendo-wiki-305731639539.europe-west6.run.app/en/revendo/implementation)
4. **Test in staging** - Use demo as reference
5. **Go live** - Deploy with confidence

---

**Need Help?** If the demo isn't working or you're seeing unexpected behavior, check the [Troubleshooting](https://revendo-wiki-305731639539.europe-west6.run.app/en/revendo/troubleshooting) guide.

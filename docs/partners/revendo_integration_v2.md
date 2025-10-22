---
title: revendo_integration_v2
description: 
published: true
date: 2025-10-16T07:35:03.351Z
tags: 
editor: markdown
dateCreated: 2025-10-15T15:04:03.253Z
---

# DQ Solutions × Revendo Trade-In Integration
## Technical Feasibility & Integration Proposal
**Version: 2.0 (Internal Draft)**
**Date: October 7, 2025**
**Status: Discussion Draft**

---

### Overview
This document outlines the technical approach for integrating Revendo's trade-in system into the DQ Solutions e-commerce platform. The integration allows customers to receive instant trade-in quotes during checkout and automatically creates trade-in orders when purchases are completed.

### Integration Summary
- **Frontend**: Iframe-based trade-in calculator embedded in DQ Solutions checkout.
- **Client Communication**: `postMessage` API for real-time data exchange between the parent window and the iframe.
- **Backend API**: REST endpoint for automated order creation.

---

### System Architecture
*Note: High-level diagrams illustrating the system architecture should be inserted here. These diagrams are crucial for a clear understanding of the integration flows.*

- **Diagram 1: High-Level Overview**: Shows the relationship between the DQ Solutions webshop, the Revendo iframe, and the Revendo backend.
- **Diagram 2: Customer Quote Generation Flow**: Details the steps from the customer launching the iframe to the quote details being passed back to the webshop.
- **Diagram 3: Order Finalization Flow**: Illustrates the backend communication, where the DQ Solutions server sends a POST request to the Revendo API to finalize the order.

---

### Frontend Integration: Iframe & `postMessage` API

#### Iframe Implementation
The trade-in functionality is embedded into the DQ Solutions webshop using an HTML `<iframe>`.

**Iframe URL:**
The `src` attribute of the iframe should point to the following URL:
`https://purchase.revendo.dev/sell-a-device-iframe/?iframe=1`

**Best Practices:**
- **Loading**: Load the iframe asynchronously to avoid blocking the parent page.
- **Styling**: Ensure the container of the iframe is styled to be responsive and fits seamlessly within the user journey.
- **Permissions**: If sensitive actions (like payment processing) are ever integrated into the iframe, use the `allow` attribute to grant necessary permissions (e.g., `allow="payment"`).

#### `postMessage` Events
Communication between the DQ Solutions parent window and the Revendo iframe is handled via the `window.postMessage` API. This is a secure way to exchange information between different origins.

**Listening for Events:**
The parent window must add an event listener to handle messages coming from the iframe.

```javascript
window.addEventListener('message', function(event) {
    // IMPORTANT: Always verify the origin of the message for security
    if (event.origin !== 'https://purchase.revendo.dev') {
        return;
    }

    // Handle the event data
    console.log('Message from Revendo iframe:', event.data);

    // Route event data to different handlers based on type
    switch (event.data.type) {
        case 'revendo_device_selected':
            // Action: Store offer details in session
            break;
        case 'revendo_resize':
            // Action: Adjust iframe height for a seamless view
            break;
        // Add other event handlers here
    }
});
```

**Key `postMessage` Events from the Iframe:**

1.  **`revendo_device_selected`**:
    - **Trigger**: Customer successfully adds a device for trade-in within the iframe.
    - **DQ Solutions Action**:
        - Store `offer_id` in the user's session.
        - Display the trade-in credit amount in the checkout summary.
        - Pass the `offer_id` to the backend when the customer completes the purchase.
    - **Data Structure**:
      ```json
      {
        "type": "revendo_device_selected",
        "details": {
          "offer_id": "11153269",
          "product": { "name": "iPhone 13" },
          "price": { "final_cash": 227 }
        }
      }
      ```

2.  **`revendo_resize`**:
    - **Trigger**: The content height of the iframe changes.
    - **DQ Solutions Action**:
        - Adjust the height of the `<iframe>` element dynamically to avoid internal scrollbars and provide a seamless user experience.
    - **Data Structure**:
      ```json
      {
        "type": "revendo_resize",
        "height": 850
      }
      ```

3.  **`revendo_iframe_loaded`**:
    - **Trigger**: The iframe has finished its initial loading.
    - **DQ Solutions Action**:
        - Can be used to hide a loading spinner on the parent page.
    - **Data Structure**:
      ```json
      {
        "type": "revendo_iframe_loaded",
        "status": "ready"
      }
      ```

---

### Backend Integration: REST API

#### Endpoint: Finalize Trade-In Order
When a customer completes their purchase on the DQ Solutions webshop, the backend system must send a POST request to Revendo's API to finalize the trade-in order.

- **Endpoint**: `POST /wp-json/revendo/v1/orders/finalize`
- **Authentication**: API Key (sent in headers).

**Request Headers**:
```
X-Revendo-API-Key: rva_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
Content-Type: application/json
```

**Request Body**:
```json
{
  "offer_id": "11153269",
  "partner_order_id": "DQ-2025-001234",
  "customer": {
    "email": "max.mustermann@example.com",
    "first_name": "Max",
    "last_name": "Mustermann",
    "phone": "+41791234567",
    "address": {
      "street": "Bahnhofstrasse 1",
      "zip": "8001",
      "city": "Zürich",
      "country": "CH"
    }
  }
}
```

**Success Response (200 OK)**:
```json
{
  "success": true,
  "order_id": 105030,
  "web_id": "ANK-2025-001234",
  "status": "pending_device"
}
```

**Error Responses**:
- **400 `offer_expired`**: Quote has expired.
- **401 `unauthorized`**: Invalid or missing API key.
- **404 `offer_not_found`**: `offer_id` does not exist.
- **409 `duplicate_order`**: An order with the same `partner_order_id` already exists.

---

### Security Best Practices

1.  **Origin Verification**:
    - **Action**: In the `postMessage` event listener, **always** verify that the message origin is `https://purchase.revendo.dev`. This prevents malicious third-party pages from sending fraudulent data.

2.  **API Key Management**:
    - **Action**: Store the `X-Revendo-API-Key` securely in an environment variable or secrets management system on the DQ Solutions backend. **Never** expose the API key in frontend code.

3.  **Data Sanitization**:
    - **Action**: Before passing data from the iframe (like `offer_id`) to the backend, ensure it is sanitized and validated to prevent injection attacks.

4.  **Idempotency**:
    - **Action**: Utilize the `partner_order_id` to prevent duplicate order creation from accidental retries. The Revendo API is designed to handle this by returning the existing order if the same ID is sent again.

---

### Open Questions for Discussion
1.  **Payment Information (IBAN)**:
    - **Question**: When/how does the customer provide bank details for the trade-in?
    - **Options**:
        - **A**: Collected separately by Revendo after the order is finalized.
        - **B**: DQ Solutions collects and sends the IBAN via a secure field in the API.
        - **C**: Embedded securely within the iframe flow.
    - **Recommendation**: Option A is the simplest and most secure, as it limits the scope of sensitive data handled by DQ Solutions.

2.  **Frontend Error Handling**:
    - **Question**: How should the DQ Solutions checkout behave if the Revendo iframe fails to load or an error occurs?
    - **Recommendation**: Implement a timeout for the `revendo_iframe_loaded` event. If the event is not received, gracefully hide the trade-in section or display a message like "Trade-in is currently unavailable."

---
title: Revendo Integration Documentation
description: Complete technical documentation for integrating Revendo trade-in system into partner e-commerce platforms
published: true
date: 2025-10-15T13:00:00.000Z
tags: revendo, integration, documentation, getting-started
editor: markdown
dateCreated: 2025-10-15T12:45:00.000Z
---

# Revendo Integration Documentation

Welcome to the technical documentation for integrating Revendo's device trade-in system into your e-commerce platform.

## What is This Integration?

This integration allows your customers to:
- Get **instant quotes** for their old devices during checkout
- Receive **trade-in credit** applied directly to their purchase
- Complete everything in one seamless flow without leaving your website

## How Does It Work?

The integration uses an **embedded iframe** with secure `postMessage` communication and a **backend API** for order finalization.

mermaid
```
flowchart TB
subgraph subGraph0["DQ Solutions Webshop"]
       A["Customer Browser"]
       B["Checkout Page"]
       C["Backend API"]
 end
subgraph subGraph1["Revendo System"]
       D["Trade-In Iframe"]
       F["REST API"]
       G["Order System"]
       I["Database"]
 end
   A -- Views Products --> B
   B -- Embeds --> D
   D -- PostMessage --> A
   A -- Stores Quote --> B
   B -- Checkout Complete --> C
   C -- POST /finalize --> F
   F -- Create Order --> G
   G -- Store --> I

```

## Documentation Structure

This documentation is organized into four main sections:

### 1. [Overview](overview)
High-level architecture, concepts, and how the integration works. **Start here** if you're new to the integration.

**Key Topics:**
- Integration architecture (3 layers)
- Customer journey and data flow
- Important concepts (offer_id, payment methods, order lifecycle)
- Security requirements and best practices

### 2. [Implementation Guide](implementation)
Step-by-step instructions with copy-paste code examples for frontend and backend integration.

**Key Topics:**
- Embedding the iframe
- Handling postMessage events
- Updating checkout UI
- Backend API call examples (PHP, Node.js)
- Testing your integration

### 3. [API Reference](api)
Complete REST API documentation including authentication, request/response formats, and error codes.

**Key Topics:**
- Endpoint specification
- Authentication with API key
- Request and response formats
- Error codes and handling
- Code examples in multiple languages

### 4. [Troubleshooting](troubleshooting)
Common issues, debugging tips, and solutions for integration problems.

**Key Topics:**
- Iframe not showing or resizing
- postMessage events not working
- API errors (400, 401, 404, 409, 500)
- Data persistence issues
- Debug checklist

### 5. [Interactive Demo](demo)
Live testing tool to see the integration in action and debug postMessage events in real-time.

**Key Features:**
- Real-time event monitoring
- Interactive iframe testing
- Debug information display
- Copy-paste event handlers

## Quick Start Guide

### For Developers New to This Integration

1. **Read the [Overview](overview)** to understand the architecture (15 min)
2. **Get your API key** from Revendo technical contact
3. **Open the demo tool** at `demo/index.html` in your browser to see events in action
4. **Follow the [Implementation Guide](implementation)** step-by-step
5. **Test** with the demo page, then integrate into your staging environment
6. **Refer to [API Reference](api)** for backend implementation details
7. **Use [Troubleshooting](troubleshooting)** if you encounter issues

### For Experienced Developers

If you're familiar with iframe postMessage integrations:

1. **Embed iframe**: `https://purchase.revendo.dev/sell-a-device-iframe/?iframe=1`
2. **Listen for `revendo_device_selected`** event → store `price_id` in sessionStorage
3. **Call finalization API** after checkout: `POST /wp-json/revendo/v1/orders/finalize`
4. **Headers**: `X-Revendo-API-Key: your_key`
5. **Body**: `{ offer_id, partner_order_id, customer: {...} }`

See [Implementation Guide](implementation) for complete code examples.

## Key Features

### Frontend Integration
- **Zero dependencies** - Pure HTML + JavaScript
- **Responsive** - Auto-resize iframe based on content
- **Secure** - Origin verification for all messages
- **Session persistence** - Data survives page refresh

### Backend Integration
- **Simple REST API** - Single endpoint for order finalization
- **Idempotent** - Duplicate order protection with partner_order_id
- **Error handling** - Clear error codes and messages
- **Multi-language** - Examples in PHP, Node.js, Python, cURL

### Developer Experience
- **Interactive demo tool** - Test all events in real-time
- **Clear documentation** - Step-by-step with copy-paste examples
- **Comprehensive troubleshooting** - Debug checklist and common issues
- **Multiple code examples** - PHP (WordPress), Node.js, Python

## Requirements

### Technical Requirements
- **HTTPS** - SSL certificate required for security
- **Modern browser** - postMessage API support (all modern browsers)
- **Backend server** - To call Revendo API after checkout
- **Session management** - sessionStorage for client-side state

### What You Need to Get Started
- **API Key** from Revendo (`X-Revendo-API-Key`)
- **Iframe URL**: `https://purchase.revendo.dev/sell-a-device-iframe/?iframe=1`
- **Basic web development knowledge** (HTML, JavaScript, HTTP APIs)

## Integration Timeline

**Typical integration time:** 2-4 hours for experienced developers

- **Frontend integration**: 1-2 hours
  - Embed iframe
  - Add event listeners
  - Update checkout UI

- **Backend integration**: 30-60 minutes
  - Add API call after order completion
  - Store tracking URL

- **Testing & debugging**: 30-60 minutes
  - Test with demo page
  - Test in staging environment
  - Handle edge cases

## Support & Resources

### Documentation Resources
- **[Overview](overview)** - Architecture and concepts
- **[Implementation Guide](implementation)** - Step-by-step integration
- **[API Reference](api)** - Complete API documentation
- **[Troubleshooting](troubleshooting)** - Common issues and solutions

### Testing Tools
- **Demo Page**: `demo/index.html` - Interactive event testing tool
- **Browser Console**: Debug postMessage events and API calls

### Getting Help
If you encounter issues:
1. Check the [Troubleshooting](troubleshooting) guide
2. Test with the provided demo HTML file
3. Review browser console for errors
4. Contact Revendo technical support with error logs

## Important Notes

### Security
- **Always verify origin** of postMessage events (`https://purchase.revendo.dev`)
- **Never expose API key** in frontend code
- **Use HTTPS** for all communication
- **Sanitize iframe data** before processing

### Business Rules
- **Quotes expire after 60 minutes** - customers need new quote if delayed
- **Two payment methods** - cash vs voucher (different prices)
- **Don't fail checkout** if Revendo API errors - complete main order anyway
- **409 errors are OK** - duplicate order already exists, continue normally

### Best Practices
- Use `sessionStorage` for data persistence
- Clear stored data after successful order
- Implement iframe auto-resize for better UX
- Add retry logic for API calls (500 errors only)

## Next Steps

Ready to integrate? **[Start with the Overview →](overview)**

Need to jump straight to code? **[Go to Implementation Guide →](implementation)**

Looking for API details? **[Check the API Reference →](api)**

Having issues? **[Visit Troubleshooting →](troubleshooting)**

---
title: 1. Overview
description: High-level overview of the Revendo trade-in integration architecture and workflow
published: true
date: 2025-10-15T13:00:00.000Z
tags: overview, architecture, integration, revendo
editor: markdown
dateCreated: 2025-10-15T12:45:00.000Z
---

# Revendo Integration Overview

## What is Revendo?

Revendo is a device trade-in system that allows customers to get instant quotes for their old devices and receive credit during checkout. This integration enables partner e-commerce platforms to offer seamless trade-in functionality directly in their purchase flow.

## Integration Architecture

The Revendo integration uses a **three-layer architecture**:

### 1. Frontend Layer (Iframe + postMessage)
- Partner website embeds a Revendo iframe into their checkout or product pages
- Communication happens via secure `postMessage` API
- No page redirects - seamless user experience

### 2. State Management (Client-side)
- Trade-in offer details stored in browser `sessionStorage`
- Survives page refreshes during checkout
- Data persists until order completion

### 3. Backend Layer (REST API)
- Partner backend calls Revendo API when customer completes purchase
- Creates finalized trade-in order in Revendo system
- Returns tracking URL for customer

## How It Works

### Customer Journey

1. **Device Selection**
   - Customer browses devices in embedded iframe
   - Selects their device and answers condition questions
   - Receives instant trade-in quote

2. **Quote Applied**
   - Trade-in credit appears in checkout summary
   - Order total automatically reduced
   - Customer completes purchase as normal

3. **Order Finalization**
   - Partner backend notifies Revendo API
   - Trade-in order created with customer details
   - Customer receives tracking URL and shipping label

### Data Flow

```
[Customer] → [Iframe] → postMessage → [Partner Frontend]
                                            ↓ sessionStorage
                                      [Checkout Page]
                                            ↓
                                     [Order Complete]
                                            ↓
                              [Partner Backend] → API Call
                                            ↓
                                   [Revendo Backend]
                                            ↓
                                  [Trade-in Order Created]
```

## Key Integration Points

### Frontend Integration
- **Iframe URL**: `https://purchase.revendo.dev/sell-a-device-iframe/?iframe=1`
- **Events**: Device selection, removal, cart updates, resize
- **Security**: Origin verification required (`https://purchase.revendo.dev`)

### Backend Integration
- **Endpoint**: `POST /wp-json/revendo/v1/orders/finalize`
- **Authentication**: API key in `X-Revendo-API-Key` header
- **Trigger**: Called after customer completes checkout

## Important Concepts

### Offer ID (price_id)
- Unique identifier for each trade-in quote
- Returned in `revendo_device_selected` event as `price_id`
- Must be sent to API as `offer_id` when finalizing order
- **Expires after 60 minutes**

### Payment Methods
- **Cash**: Customer receives bank transfer
- **Voucher**: Customer receives store credit (usually higher value)
- Customer can switch between methods in iframe
- `revendo_cart_updated` event fires when they switch

### Order Lifecycle
1. **Quote Created** - Customer gets instant price in iframe
2. **Quote Selected** - Customer adds device to their cart
3. **Order Finalized** - Partner calls API after checkout
4. **Device Shipped** - Customer sends device to Revendo
5. **Payment Processed** - Customer receives payment after inspection

## Security & Best Practices

### Security Requirements
- Always verify `event.origin === 'https://purchase.revendo.dev'`
- Store API key securely in backend (never expose in frontend)
- Use HTTPS for all communication
- Sanitize data from iframe before processing

### Error Handling
- **Quote expired**: Show message, ask customer to select device again
- **API errors**: Log error but don't fail customer's main order
- **Network issues**: Implement retry logic with exponential backoff
- **Duplicate orders**: 409 errors are OK - order already exists

### Integration Best Practices
- Use `sessionStorage` for offer data (survives page refresh)
- Clear stored data after successful order completion
- Implement iframe auto-resize for seamless UI
- Test with provided demo page before production

## What You Need to Get Started

1. **API Key** - Contact Revendo to receive your `X-Revendo-API-Key`
2. **Iframe Access** - Production URL: `https://purchase.revendo.dev/sell-a-device-iframe/?iframe=1`
3. **Testing Tool** - Use the provided `demo/index.html` to test events
4. **Documentation** - This wiki contains complete implementation guide

## Next Steps

1. Read the [Implementation Guide](implementation) for step-by-step integration
2. Review the [API Reference](api) for endpoint specifications
3. Check [Troubleshooting](troubleshooting) for common issues
4. Test with the demo HTML file included in the repository

## Support

For technical questions or issues:
- Review the troubleshooting guide
- Test with the demo tool to isolate issues
- Contact Revendo technical support with error logs and browser details

# Opay Payment Integration Documentation

## Overview

Opay is a payment processing service that allows you to accept payments from customers. This documentation covers the integration process for creating payment sessions using the Opay API.

## Base URL

```
https://opay.orbtronics.co
```

## Authentication

> **Note:** API credentials can be generated directly from your Opay dashboard. Navigate to **API Keys** to create and manage your API keys.

---

## Create Payment Session

### Endpoint

```
POST /api/v1/payments
```

### Description

Creates a new payment session and returns a checkout URL for the customer to complete their payment. Product items are managed on the client side.

### Request Headers

```
Content-Type: application/json
Authorization: Bearer YOUR_API_KEY
```

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `amount` | integer | Yes | Payment amount in the smallest currency unit (e.g., cents for USD) |
| `currency` | string | Yes | Three-letter ISO currency code (e.g., "usd", "eur", "gbp") |
| `customer` | object | Yes | Customer information object |
| `customer.email` | string | Yes | Customer's email address |
| `customer.name` | string | Yes | Customer's full name |
| `success_url` | string | Yes | URL to redirect customer after successful payment |
| `cancel_url` | string | Yes | URL to redirect customer if payment is cancelled |

### Request Example

**USD Payment:**

```json
{
  "amount": 30000,
  "currency": "usd",
  "customer": {
    "email": "garret@gmail.com",
    "name": "Garret"
  },
  "success_url": "https://example.com/success",
  "cancel_url": "https://example.com/cancel"
}
```

**JMD (Jamaican Dollar) Payment:**

```json
{
  "amount": 450000,
  "currency": "jmd",
  "customer": {
    "email": "customer@example.com",
    "name": "John Smith"
  },
  "success_url": "https://example.com/success",
  "cancel_url": "https://example.com/cancel"
}
```

**XCD (East Caribbean Dollar) Payment:**

```json
{
  "amount": 80000,
  "currency": "xcd",
  "customer": {
    "email": "customer@example.com",
    "name": "Jane Doe"
  },
  "success_url": "https://example.com/success",
  "cancel_url": "https://example.com/cancel"
}
```

### Response Format

#### Success Response (200 OK)

```json
{
  "success": true,
  "session_id": "sess_1234567890abcdef",
  "checkout_url": "https://checkout.opay.orbtronics.co/pay/sess_1234567890abcdef",
  "expires_at": "2025-12-08T10:30:00Z"
}
```

#### Error Response (400/401/500)

```json
{
  "success": false,
  "error": {
    "code": "invalid_request",
    "message": "Invalid currency code provided"
  }
}
```

---

## Implementation Guide

### Step 1: Prepare Payment Data

Collect the necessary payment information from your client application:

```javascript
const paymentData = {
  amount: 30000, // $300.00 in cents
  currency: "usd",
  customer: {
    email: customerEmail,
    name: customerName
  },
  success_url: "https://yourwebsite.com/payment/success",
  cancel_url: "https://yourwebsite.com/payment/cancel"
};
```

### Step 2: Create Payment Session

Make a POST request to the Opay API:

**JavaScript/Node.js Example:**

```javascript
async function createPaymentSession(paymentData) {
  try {
    const response = await fetch('https://opay.orbtronics.co/api/v1/payments', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'Authorization': 'Bearer YOUR_API_KEY'
      },
      body: JSON.stringify(paymentData)
    });

    const result = await response.json();
    
    if (result.success) {
      // Redirect customer to checkout
      window.location.href = result.checkout_url;
    } else {
      console.error('Payment session creation failed:', result.error);
    }
  } catch (error) {
    console.error('Error:', error);
  }
}
```

**Python Example:**

```python
import requests
import json

def create_payment_session(payment_data):
    url = "https://opay.orbtronics.co/api/v1/payments"
    headers = {
        "Content-Type": "application/json",
        "Authorization": "Bearer YOUR_API_KEY"
    }
    
    response = requests.post(url, headers=headers, json=payment_data)
    result = response.json()
    
    if result.get('success'):
        return result.get('checkout_url')
    else:
        raise Exception(f"Payment creation failed: {result.get('error')}")

# Usage
payment_data = {
    "amount": 30000,
    "currency": "usd",
    "customer": {
        "email": "garret@gmail.com",
        "name": "Garret"
    },
    "success_url": "https://example.com/success",
    "cancel_url": "https://example.com/cancel"
}

checkout_url = create_payment_session(payment_data)
```

**cURL Example:**

```bash
curl -X POST https://opay.orbtronics.co/api/v1/payments \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -d '{
    "amount": 30000,
    "currency": "usd",
    "customer": {
      "email": "garret@gmail.com",
      "name": "Garret"
    },
    "success_url": "https://example.com/success",
    "cancel_url": "https://example.com/cancel"
  }'
```

### Step 3: Redirect to Checkout

Once you receive the `checkout_url` from the API response, redirect your customer to this URL to complete the payment.

### Step 4: Handle Callbacks

Implement handlers for both success and cancel URLs:

**Success Handler:**
```javascript
// https://yourwebsite.com/payment/success?session_id=sess_xxx
app.get('/payment/success', (req, res) => {
  const sessionId = req.query.session_id;
  
  // Verify payment status with Opay (recommended)
  // Update order status in your database
  // Display success message to customer
  
  res.render('payment-success');
});
```

**Cancel Handler:**
```javascript
// https://yourwebsite.com/payment/cancel?session_id=sess_xxx
app.get('/payment/cancel', (req, res) => {
  const sessionId = req.query.session_id;
  
  // Log cancellation
  // Allow customer to retry
  
  res.render('payment-cancelled');
});
```

---

## Currency Codes

Opay supports all currencies that Stripe supports, including over 135+ currencies worldwide. Use standard ISO 4217 currency codes in lowercase.

### Common Currencies

| Currency | Code | Smallest Unit |
|----------|------|---------------|
| US Dollar | `usd` | cents (100 = $1.00) |
| Euro | `eur` | cents (100 = €1.00) |
| British Pound | `gbp` | pence (100 = £1.00) |
| Canadian Dollar | `cad` | cents (100 = $1.00) |
| Jamaican Dollar | `jmd` | cents (100 = $1.00) |
| East Caribbean Dollar | `xcd` | cents (100 = $1.00) |
| Australian Dollar | `aud` | cents (100 = $1.00) |
| Japanese Yen | `jpy` | yen (1 = ¥1) |

### Caribbean Currencies

| Currency | Code | Smallest Unit |
|----------|------|---------------|
| Jamaican Dollar | `jmd` | cents (100 = $1.00) |
| East Caribbean Dollar | `xcd` | cents (100 = $1.00) |
| Bahamian Dollar | `bsd` | cents (100 = $1.00) |
| Barbadian Dollar | `bbd` | cents (100 = $1.00) |
| Trinidad and Tobago Dollar | `ttd` | cents (100 = $1.00) |

### Zero-Decimal Currencies

Some currencies don't use decimal places. For these currencies, use the actual amount without multiplying by 100:

| Currency | Code | Amount Format |
|----------|------|---------------|
| Japanese Yen | `jpy` | ¥1000 = 1000 |
| Korean Won | `krw` | ₩1000 = 1000 |
| Chilean Peso | `clp` | $1000 = 1000 |
| Vietnamese Dong | `vnd` | ₫1000 = 1000 |

### Full Currency Support

Opay supports all currencies available in Stripe's platform, including but not limited to:

**Americas:** `usd`, `cad`, `mxn`, `brl`, `ars`, `clp`, `cop`, `pen`, `jmd`, `xcd`, `bsd`, `bbd`, `ttd`

**Europe:** `eur`, `gbp`, `chf`, `sek`, `nok`, `dkk`, `pln`, `czk`, `huf`, `ron`, `bgn`, `hrk`

**Asia-Pacific:** `jpy`, `aud`, `nzd`, `sgd`, `hkd`, `inr`, `thb`, `myr`, `php`, `idr`, `krw`

**Middle East & Africa:** `aed`, `sar`, `ils`, `egp`, `zar`, `ngn`, `kes`, `ghs`

For a complete list of supported currencies, refer to the [Stripe Currency Documentation](https://stripe.com/docs/currencies)

---

## Error Codes

| Code | Description | Solution |
|------|-------------|----------|
| `invalid_request` | Request payload is malformed | Check JSON structure and required fields |
| `invalid_amount` | Amount is negative or zero | Ensure amount is a positive integer |
| `invalid_currency` | Currency code not supported | Use valid ISO currency code |
| `invalid_email` | Email format is invalid | Validate email format before sending |
| `authentication_failed` | API key is invalid or missing | Check your API credentials |
| `rate_limit_exceeded` | Too many requests | Implement exponential backoff |

---

## Best Practices

### 1. Amount Handling
Always send amounts in the smallest currency unit to avoid floating-point errors:

```javascript
// ✅ Correct
const amount = 30000; // $300.00

// ❌ Incorrect
const amount = 300.00; // Will be interpreted as $3.00
```

### 2. URL Configuration
- Use HTTPS for all callback URLs
- Ensure success and cancel URLs are publicly accessible
- Include session tracking parameters in your URLs

### 3. Error Handling
Always implement proper error handling:

```javascript
try {
  const result = await createPaymentSession(paymentData);
  // Handle success
} catch (error) {
  // Log error
  // Show user-friendly message
  // Provide retry option
}
```

### 4. Security
- Never expose API keys in client-side code
- Always make payment API calls from your backend server
- Validate payment status on your server before fulfilling orders

### 5. Client-Side Product Management
Since product items are managed on the client side:
- Maintain product details in your frontend state
- Store order details in your database before creating payment session
- Reference your order ID in success/cancel URLs for reconciliation

---

## Integration Checklist

- [ ] Obtain API credentials from Opay
- [ ] Set up backend endpoint to create payment sessions
- [ ] Configure success and cancel callback URLs
- [ ] Implement payment session creation
- [ ] Handle success callback
- [ ] Handle cancel callback
- [ ] Test with various amounts and currencies
- [ ] Implement error handling and logging
- [ ] Set up payment verification (webhook recommended)
- [ ] Test in production environment

---

## Support

For additional support or questions about the Opay integration:

- **Email:** support@orbtronics.co
- **Documentation:** https://docs.opay.orbtronics.co
- **API Status:** https://status.opay.orbtronics.co

---

## Changelog

### Version 1.0
- Initial documentation
- Core payment session creation endpoint

---

*Last Updated: December 2025*
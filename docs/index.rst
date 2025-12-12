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

## Webhooks

Opay can send webhook notifications to your server when payment events occur. This allows you to update your system in real-time when payments succeed or fail.

### Setting Up Webhooks

1. **Configure Webhook URL**: Set your webhook endpoint URL in the Opay dashboard under Settings → Webhooks
2. **Test Your Endpoint**: Use the "Test Webhook" button to verify your endpoint is working
3. **Handle Events**: Process incoming webhook events in your application

### Webhook Events

Opay sends the following webhook events:

| Event | Description |
|-------|-------------|
| `payment.succeeded` | Payment completed successfully |
| `payment.failed` | Payment failed or was declined |
| `webhook.test` | Test event (when using Test button) |

### Webhook Payload Format

All webhook events follow this structure:

```json
{
  "event": "payment.succeeded",
  "data": {
    "payment_id": "pi_1234567890",
    "transaction_id": "uuid-transaction-id",
    "amount": 30000,
    "currency": "usd",
    "status": "succeeded"
  },
  "timestamp": "2025-12-12T10:30:00Z"
}
```

### Event-Specific Data

**payment.succeeded:**
```json
{
  "event": "payment.succeeded",
  "data": {
    "payment_id": "pi_1234567890",
    "transaction_id": "uuid-transaction-id",
    "amount": 30000,
    "currency": "usd",
    "status": "succeeded"
  },
  "timestamp": "2025-12-12T10:30:00Z"
}
```

**payment.failed:**
```json
{
  "event": "payment.failed",
  "data": {
    "payment_id": "pi_1234567890",
    "transaction_id": "uuid-transaction-id",
    "amount": 30000,
    "currency": "usd",
    "status": "failed",
    "error": "Your card was declined."
  },
  "timestamp": "2025-12-12T10:30:00Z"
}
```

### Implementing Webhook Handlers

**Node.js/Express Example:**

```javascript
app.post('/webhooks/opay', express.raw({type: 'application/json'}), (req, res) => {
  const event = req.body;
  
  switch (event.event) {
    case 'payment.succeeded':
      handlePaymentSucceeded(event.data);
      break;
    case 'payment.failed':
      handlePaymentFailed(event.data);
      break;
    case 'webhook.test':
      console.log('Webhook test received');
      break;
    default:
      console.log(`Unhandled event type: ${event.event}`);
  }
  
  res.status(200).send('OK');
});

function handlePaymentSucceeded(data) {
  // Update order status in database
  // Send confirmation email
  // Fulfill order
  console.log(`Payment succeeded: ${data.payment_id}`);
}

function handlePaymentFailed(data) {
  // Log failed payment
  // Notify customer
  // Update order status
  console.log(`Payment failed: ${data.payment_id} - ${data.error}`);
}
```

**Python/Flask Example:**

```python
from flask import Flask, request, jsonify

@app.route('/webhooks/opay', methods=['POST'])
def handle_webhook():
    event = request.get_json()
    
    if event['event'] == 'payment.succeeded':
        handle_payment_succeeded(event['data'])
    elif event['event'] == 'payment.failed':
        handle_payment_failed(event['data'])
    elif event['event'] == 'webhook.test':
        print('Webhook test received')
    
    return jsonify({'status': 'success'}), 200

def handle_payment_succeeded(data):
    # Update order status
    # Send confirmation
    print(f"Payment succeeded: {data['payment_id']}")

def handle_payment_failed(data):
    # Handle failed payment
    print(f"Payment failed: {data['payment_id']} - {data.get('error', 'Unknown error')}")
```

### Webhook Best Practices

1. **Respond Quickly**: Return a 200 status code within 10 seconds
2. **Handle Duplicates**: Webhook events may be sent multiple times
3. **Validate Events**: Check the event structure before processing
4. **Use HTTPS**: Always use HTTPS endpoints for webhook URLs
5. **Log Events**: Keep logs of all webhook events for debugging

### Testing Webhooks

Use the webhook test feature in your Opay dashboard:

1. Navigate to Settings → Webhooks
2. Enter your webhook URL
3. Click "Test Webhook" to send a test event
4. Verify your endpoint receives and processes the test event

### Webhook Security

- Use HTTPS endpoints only
- Validate the webhook payload structure
- Implement idempotency to handle duplicate events
- Consider implementing webhook signature verification for additional security

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
- [ ] Set up webhook endpoint (recommended)
- [ ] Configure webhook URL in Opay dashboard
- [ ] Test webhook functionality
- [ ] Test with various amounts and currencies
- [ ] Implement error handling and logging
- [ ] Test in production environment

---

## Support

For additional support or questions about the Opay integration:

- **Email:** support@orbtronics.co
- **Documentation:** https://docs.opay.orbtronics.co
- **API Status:** https://status.opay.orbtronics.co


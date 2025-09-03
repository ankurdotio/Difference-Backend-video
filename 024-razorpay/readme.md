# Razorpay Payment Integration with Express.js

This guide will help you set up Razorpay payment integration with Express.js.It includes creating orders, handling payments, verifying payments, and saving payment details in MongoDB.

## Prerequisites

1. Node.js installed
2. MongoDB installed and running
3. Razorpay account(for API keys)

## Setup

### 1. Install Razorpay

Install Razorpay using npm:

``` sh
npm install razorpay
```

### 2. Create an Instance to Access Resources from Razorpay API

In your `index.js` file, create an instance of Razorpay:

```js
require('dotenv').config();
const Razorpay = require('razorpay');

const razorpay = new Razorpay({
  key_id: process.env.RAZORPAY_KEY_ID,
  key_secret: process.env.RAZORPAY_KEY_SECRET,
});
```

### 3. Setup MongoDB Connection

Connect to MongoDB:

```js
const mongoose = require('mongoose');
mongoose.connect(process.env.MONGODB_URI, { useNewUrlParser: true, useUnifiedTopology: true })
  .then(() => console.log('MongoDB connected'))
  .catch((err) => console.log(err));
```

### 4. Define the Payment Schema

First, define the Payment schema using Mongoose.Create a new file called `models/Payment.js`:

```js
const mongoose = require('mongoose');

const paymentSchema = new mongoose.Schema({
  orderId: {
    type: String,
    required: true,
  },
  paymentId: {
    type: String,
  },
  signature: {
    type: String,
  },
  amount: {
    type: Number,
    required: true,
  },
  currency: {
    type: String,
    required: true,
  },
  status: {
    type: String,
    default: 'pending',
  },
}, { timestamps: true });

const Payment = mongoose.model('Payment', paymentSchema);

module.exports = Payment;
```


### 5. Import the Payment Model in `index.js`

Import the Payment model in your`index.js` file:

```js
const Payment = require('../models/Payment');
```



### 6. Create a Checkout Page

Create an HTML file `index.html` for your frontend:

```html
<!DOCTYPE html>
<html>

<head>
  <title>Razorpay Integration</title>
  <link rel='stylesheet' href='/stylesheets/style.css' />
</head>

<body>
  <h1>Razorpay Integration</h1>
  <p>Welcome to Razorpay Integration</p>

  <button id="rzp-button1">Pay with Razorpay</button>

  
</body>

</html>
```

### 7. Make a POST Route for URL = '/create/orderId' on`index.js`

In your `index.js` file, create a POST route to create an order:

```js
router.post('/create/orderId', async (req, res) => {
  const options = {
    amount: 5000 * 100, // amount in smallest currency unit
    currency: "INR",
  };
  try {
    const order = await razorpay.orders.create(options);
    res.send(order);

    const newPayment = await Payment.create({
      orderId: order.id,
      amount: order.amount,
      currency: order.currency,
      status: 'pending',
    });

  } catch (error) {
    res.status(500).send('Error creating order');
  }
});
```

### 8. Add Axios CDN on Checkout Page

Ensure the Axios and Jquery CDN is added in the`index.html` file(already included in step 4):

```html
<script src="https://checkout.razorpay.com/v1/checkout.js"></script>
<script src="https://cdn.jsdelivr.net/npm/axios/dist/axios.min.js"></script>
```

### 9. Generate Order Code in Checkout Page Inside Script Tags

Add the AJAX request to generate the order inside the `<script>` tags in `index.html`:

```html
<script>
document.getElementById('rzp-button1').onclick = function(e) {
  axios.post('/create/orderId')
    .then(function (response) {
      var options = {
        "key": "YOUR_RAZORPAY_KEY_ID", // Enter the Key ID generated from the Dashboard
        "amount": response.data.amount, // Amount in currency subunits. Default currency is INR.
        "currency": response.data.currency,
        "name": "YOUR_COMPANY_NAME",
        "description": "Test Transaction",
        "image": "https://example.com/your_logo",
        "order_id": response.data.id, 
        "handler": function(response) {
          axios.post('/api/payment/verify', {
            razorpayOrderId: response.razorpay_order_id,
            razorpayPaymentId: response.razorpay_payment_id,
            signature: response.razorpay_signature
          })
          .then(function (response) {
            alert('Payment verified successfully');
          })
          .catch(function (error) {
            console.error(error);
          });
        },
        "prefill": {
          "name": "Gaurav Kumar",
          "email": "gaurav.kumar@example.com",
          "contact": "9000090000"
        },
        "notes": {
          "address": "Razorpay Corporate Office"
        },
        "theme": {
          "color": "#3399cc"
        }
      };
      var rzp1 = new Razorpay(options);
      rzp1.on('payment.failed', function(response) {
        alert('Payment Failed');
        alert('Error Code: ' + response.error.code);
        alert('Description: ' + response.error.description);
        alert('Source: ' + response.error.source);
        alert('Step: ' + response.error.step);
        alert('Reason: ' + response.error.reason);
        alert('Order ID: ' + response.error.metadata.order_id);
        alert('Payment ID: ' + response.error.metadata.payment_id);
      });
      rzp1.open();
      e.preventDefault();
    })
    .catch(function (error) {
      console.error(error);
    });
};
</script>
```

### 10. Create POST Route for '/api/payment/verify' and Verify Payment Signature

In your `index.js` file, create a POST route to verify the payment signature:

```js

router.post('/api/payment/verify', async (req, res) => {
  const { razorpayOrderId, razorpayPaymentId, signature } = req.body;
  const secret = process.env.RAZORPAY_KEY_SECRET

  try {
    const { validatePaymentVerification } = require('../node_modules/razorpay/dist/utils/razorpay-utils.js')

    const result = validatePaymentVerification({ "order_id": razorpayOrderId, "payment_id": razorpayPaymentId }, signature, secret);
    if (result) {
      const payment = await Payment.findOne({ orderId: razorpayOrderId });
      payment.paymentId = razorpayPaymentId;
      payment.signature = signature;
      payment.status = 'completed';
      await payment.save();
      res.json({ status: 'success' });
    } else {
      res.status(400).send('Invalid signature');
    }
  } catch (error) {
    console.log(error);
    res.status(500).send('Error verifying payment');
  }
});

```


# Frontend (React)

### Install Axios

```sh
npm install axios
```

### Add Razorpay Checkout Script

In `public/index.html`, inside `<head>`:

```html
<script src="https://checkout.razorpay.com/v1/checkout.js"></script>
```

### Create Payment Button Component

```jsx
import React from "react";
import axios from "axios";

function PaymentButton() {
  const handlePayment = async () => {
    try {
      // Step 1: Create order on backend
      const { data: order } = await axios.post("http://localhost:5000/api/payment/orders", {
        amount: 500 // Amount in INR
      });

      // Step 2: Razorpay options
      const options = {
        key: "YOUR_RAZORPAY_KEY_ID", // from .env (frontend can use only key_id)
        amount: order.amount,
        currency: order.currency,
        name: "My Company",
        description: "Test Transaction",
        order_id: order.id,
        handler: async function (response) {
          const { razorpay_order_id, razorpay_payment_id, razorpay_signature } = response;
          try {
            await axios.post("http://localhost:5000/api/payment/verify", {
              razorpayOrderId: razorpay_order_id,
              razorpayPaymentId: razorpay_payment_id,
              signature: razorpay_signature,
            });
            alert("Payment successful!");
          } catch (err) {
            alert("Payment verification failed!");
          }
        },
        prefill: {
          name: "Test User",
          email: "test@example.com",
          contact: "9999999999"
        },
        theme: {
          color: "#3399cc"
        }
      };

      const rzp = new window.Razorpay(options);
      rzp.open();
    } catch (err) {
      console.error(err);
    }
  };

  return (
    <button onClick={handlePayment} style={{ padding: "10px 20px", background: "#3399cc", color: "#fff", border: "none", borderRadius: "5px" }}>
      Pay Now
    </button>
  );
}

export default PaymentButton;
```


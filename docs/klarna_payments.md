# Klarna Payments
?> Klarna offers 'Pay Now', 'Pay Later', and 'Slice it' payment options via a widget that the merchant adds inline on the checkout page

> Klarna's payment widget lets you:<br> - easily add Klarna's payment options in your checkout <br> - integrate this using a JSON REST API and a JavaScript SDK <br> - apply your visual guidelines to get a branded experience

## Integration Process Overview
<img align="center" src="https://developers.klarna.com/static/Klarna_Payments-3e34293a104a177fc60851c9d2d97589-89906.png">

--- 

### Steps 👣

## 1. Create Session Call 🏁
> When consumer proceeds to merchant checkout page, create a session with Klarna
The session is created server side via Klarna's REST API before the widget is used.

!> When the session is created, you receive available `payment method categories`, a `session id`, and a `client token`
The session_id can be used to update the session using the REST API, the client_token should be passed to the browser. A Session is valid for 48 hours.

### 1.1 Create the session through the API call 📲
Send the shopping cart details, country, currency, and locale via server-side to [Klarna's REST API](https://developers.klarna.com/api/#payments-api)

```JSON
POST /payments/v1/sessions
Authorization: Basic pwhcueUff0MmwLShJiBE9JHA==
Content-Type: application/json

{
	"purchase_country": "GB",
	"purchase_currency": "GBP",
	"locale": "en-GB",
	"order_amount": 10,
	"order_tax_amount": 0,
	"order_lines": [{
		"type": "physical",
		"reference": "19-402",
		"name": "Battery Power Pack",
		"quantity": 1,
		"unit_price": 10,
		"tax_rate": 0,
		"total_amount": 10,
		"total_discount_amount": 0,
		"total_tax_amount": 0
	}]
}
```
### 1.2 Create the Session Response 🎣
?> API returns the following data <br> - `session_id` used for server-side updates on the session via server-side REST API <br> - `client_token` used to initialize the widget <br> - `payment_method_category` represents payment methods that are available for this purchase. It's required when loading the widget.

Example response:
```JSON
HTTP/1.1 200 OK
Content-Type: application/json

{
  "session_id": "068df369-13a7-4d47-a564-62f8408bb760",
  "client_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6IjAwMDAwMDAwMDAtMDAwMDAtMDAwMC0wMDAwMDAwMC0wMDAwIiwidXJsIjoiaHR0cHM6Ly9jcmVkaXQtZXUua2xhcm5hLmNvbSJ9.A_rHWMSXQN2NRNGYTREBTkGwYwtm-sulkSDMvlJL87M",
  "payment_method_categories": [{
      "identifier": "pay_later"
      "name" : "Pay later.",
      "asset_urls" : {
        "descriptive" : "https://cdn.klarna.com/1.0/shared/image/generic/badge/en_us/pay_later/descriptive/pink.svg",
        "standard" : "https://cdn.klarna.com/1.0/shared/image/generic/badge/en_us/pay_later/standard/pink.svg"
      }
  }]
}
```

### 1.3 Session Updates
If the checkout page is reloaded (e.g. customer goes back to product page and then back to checkout), you can update an existing session. This is done by making a POST call to [update the session](https://developers.klarna.com/api/#payments-api-update-an-existing-credit-session) using the same payload structure as when creating the session.

```JSON
POST /payments/v1/sessions/068df369-13a7-4d47-a564-62f8408bb760
Authorization: Basic pwhcueUff0MmwLShJiBE9JHA==
Content-Type: application/json

{
	"order_amount": 998,
	"order_tax_amount": 0,
	"order_lines": [{
		"type": "physical",
		"reference": "19-315",
		"name": "Battery Power Pack",
		"quantity": 2,
		"unit_price": 499,
		"tax_rate": 0,
		"total_amount": 998,
		"total_discount_amount": 0,
		"total_tax_amount": 0
	}]
}

```
?> This call will update the Klarna session and has an empty response
```JSON
HTTP/1.1 204 No content
Content-Type: application/json
```

---

## 2. Present Widget 🎉
> When you want to display the Klarna Payment methods to the consumer, initiate and load the widget
During this step, the Klarna widget is initialized, loaded and displays the available Klarna payment options to the user. You should render this on the checkout page along other payment methods offered.
This should be done using the [JavaScript SDK](https://credit.klarnacdn.net/lib/v1/index.html)

### 2.1 Add the JavaScript SDK to your page 👩‍💻
Add the following script in the body section of your checkout page and it will load the SDK into your page
```javascript
<script>
  window.klarnaAsyncCallback = function () {

    // This is where you start calling Klarna's JS SDK functions
    // 
    // Klarna.Payments.init({....})

  };
</script>
<script src="https://x.klarnacdn.net/kp/lib/v1/api.js" async></script>
```
---

### 2.2 Initialize the SDK 🌟
Initiate the SDK by calling `init`, passing the `client_token` that was returned in the ['create a session'](http://localhost:3000/#/klarna_payments?id=_1-create-session-call-%f0%9f%8f%81) step

```javascript
Klarna.Payments.init({
  client_token: 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJmb28iOiJiYXIifQ.dtxWM6MIcgoeMgH87tGvsNDY6cH'
})
```
---

### 2.3 Add a container on your page ➕
To specify where you want the widget to display, add a container to your checkout page.
```html
<div id="klarna_container"></div>
```

--- 

### 2.4 Load the Klarna Widget 🤸‍
Load the Klarna widget using the `load` call. Pass the id of your container, and specify the `payment_method_category`. The available payment method categories were provided in response of the [create session call](http://localhost:3000/#/klarna_payments?id=_12-create-the-session-response-%f0%9f%8e%a3).

```javascript
Klarna.Payments.load({
    container: '#klarna-payments-container',
    payment_method_category: 'pay_later'
  }, function (res) {
    console.debug(res);
})
```
!> Best Practice: the `load` method should be called when rendering the checkout page so that you ensure the widget has fully loaded by the time the consumer wants to interact with or select Klarna as payment method.

---

### 2.5 Receive Response from load method execution 🎣
When the Javascript SDK has processed the load call, the provided callback will be invoked. This callback will be an object containing the following properties:
- `show_form: true/false` indicating whether you should make the Klarna option available to the user in your checkout or if not for this order
- `error` containing details of potential error messages.

#### 2.5.1 show_form: true | Klarna Payments is offered
No errors returned and Klarna renders the available payment methods to the user in the widget.
Example:
<img align="center" src="https://developers.klarna.com/static/UK-KP-d48ae63bd86dd69d0ebbeb1a8dc80e94-e2716.png">

#### 2.5.2 show_form: true && error of invalid fields | Adjust and try again
In this case, the user needs to take action before moving forward. Klarna informs the user about the details of the error in the widget. Optionally, you can interpret the invalid fields in the error object and take appropriate actions.
```JSON

   show_form: true, 
   error: {
            invalid_fields: ["billing_address.email"]
          }
}
```

#### 2.5.3 show_form: false | Klarna Payments not offered
Klarna Payment option will not be provided to the user based on Klarna's evaluation. A message will be displayed to the user in the Widget.
```JSON
{
   show_form: false
}
```
?> In this case, best practice is to hide, grey out, or remove the Klarna option, disable clicking, and provide the user with other payment options.

---

### 2.6 Cart Updates 🛒
If the user changes something in their cart or their billing details, you can also use load to pass any updates to the Klarna session that might have occurred since the session was created. E.g. the order info has changed since the user added an item to the cart.
```javascript
Klarna.Payments.load({
  container: "#klarna_container",
  payment_method_category: 'pay_later' 
}, {
  "purchase_country": "GB",
  "purchase_currency": "GBP",
  "locale": "en-GB",
  "order_amount": 20,
  "order_tax_amount": 0,
  "order_lines": [{
    "type": "physical",
    "reference": "19-402",
    "name": "Battery Power Pack",
    "quantity": 2,
    "unit_price": 10,
    "tax_rate": 0,
    "total_amount": 20,
    "total_discount_amount": 0,
    "total_tax_amount": 0	
 }]
}
, function(res) {
  console.debug(res);
})
```

?> When the customer presses 'Buy' or 'Continue' button on your site, you will authorize the order.

---
<h2 align="center">SMOOOTH BREAK</h2>

<div align="center">

  ![logo](https://media.giphy.com/media/ESF5Iltz4M1MY/giphy.gif )

</div>


---




## 3. Authorize 👮‍
> When customer presses 'buy' button on your page, make an authorization request 

When the user presses buy / continue submit button on your checkout page, you call our Javascript SDK to authorize the order at Klarna and receive an authorization token in return, giving you the flexibility to authorize and place the order in several steps.

!> If you have a: <br> - single-page checkout, you call authorize when the user presses the buy button <br> - multi-page checkout, call authorize when the user clicks on continue/review order button and finalize the order when user presses buy button.

### 3.1 Authorize the order 🚓
As a merchant, you will not handle sensitive details the user has entered. This will be handled by Klarna. Successful authorization guarantees the order can be created within 60 minutes.

Authorization is made with a client side call to `authorize()`
Successful authorization: `approved: true` results in an `authorization_token` that should be used when creating the order. The response object also contains `show_form` key which indicates whether Klarna payment option should remain available (e.g. in case there are errors that the user can resolve) or if you should remove it completely.
You may also pass billing and shipping address details in `authorize()` - this will all be used by Klarna when authorizing the order.

!> **_User interaction during authorize call_** <br> When authorizing the order, Klarna conducts a full risk assessment. Therefore, from the point where you call `authorize()` until you receive the callback, you must: <br> 1. Avoid sending another authorize call (e.g. disable the buy button from being clicked again) <br> 2. Show to the user that the order is being processed (e.g. by showing a loading spinner) <br> 3. Prevent user from changing order or billing details (e.g. lock the input fields on your page).

--- 

### 3.2 Act on the callback from the authorize call 🚔
When the widget has processed the authorization, the callback will be executed.
The callback is an object containing the following properties:
- `approved: true/false` - the authorization result is approved / denied 
- `show_form: true/false` - whether the Klarna Widget should be displayed / hidden
- `authorization_token` - allows you to create the order server-side - only returned if authorization was approved.
- `error` - contains details of potential error messages


#### 3.2.1 Finalize the authorization 🔚
`finalize()` is a special call that is needed for 'Pay Now' payment method category in multi-step checkouts. In multi-step checkouts, `authorize()` can be triggered when the user clicks on 'continue' to go to the next step of the checkout.

With 'Pay Now' however, transferring funds should only happen when the user presses the 'buy' button to finalize the purchase. To handle this scenario, you can still call authorize() when user has selected a payment method but with `auto_finalize: false`

```javascript
Klarna.Payments.authorize(
  { payment_method_category: ‘pay_now’, auto_finalize: false},
  {}, 
  function(res) {
  // proceed to next checkout page. The finalize_required property in the response indicates 
  // if finalize is needed or not.
  // 
  // res = {
  //   show_form: true,
  //   approved: false,
  //   finalize_required: true
  // }
})
```
When the user reaches the final page in the checkout and can finalize the purchase, `finalize()` is called. This triggers the transfer of funds and returns the `authorization_token` in the finalize callback.
If finalization was not needed in the `authorize()` call (e.g. for pay later) `finalize()` can still be called and will return the `authorization_token` so that the implementation remains the same for all payment method categories.

`finalize()` example:
```javascript
Klarna.Payments.finalize(
  {payment_method_category: ‘pay_now’},
  {}, 
  function(res) {
  // res = {
  //   show_form: true,
  //   approved: true,
  //   authorization_token: ...
  // }
})
```

#### 3.2.2 Order approved 🙌
If `approved: true` - Klarna has approved the authorization of credit for this order
```javascript
{
   authorization_token: "b4bd3423-24e3", 
   approved: true, 
   show_form: true
}
```
The `authorization_token` allows you to complete the order by the server-side place order call. The token is valid for 60 minutes, during which authorization is guaranteed. In case, place order is done after expiry, Klarna will try to re-authorize the order, but successful outcome cannot be guaranteed.

?> Best practice: Store authorization_token in hidden form field and submit it to backend with 'buy'/'place order' form submit button


#### 3.2.2 Order not approved 😞
If `approved: false` - Klarna cannot approve the purchase. There are now 2 options:
- **_More info needed_**: The widget will display an error message to the user, asking them to correct / add additional information before re-authorizing the order.
- **_Customer interaction aborted_**: If `show_form: true` and no `error` included in callback, user has aborted a required interaction in the widget. In this case, you should keep showing Klarna's payment options as the customer might want to try to complete the purchase again with Klarna.
- **_Order declined_**
If `show_form: false`, the order is declined. The widget should be hidden and the user should select another payment method.

---

### 3.3 Tokenization 🉑
With an `authorization_token`, it’s possible to create tokens for recurring charges or directly place an order using the order endpoint.
The token endpoint is called to tokenise the payment method and create the chargeable customer token. The server call includes the `authorization_token` in the URL and a successful registration for a token would return a `customer_token` id, which can be used in order to charge the customer at a later stage without the customer being present.

!> Create Customer Token with authorization token:
```JSON
POST /payments/v1/authorizations/{authorizationToken}/customer-token
Authorization: Basic pwhcueUff0MmwLShJiBE9JHA==
Content-Type: application/json

{
  "purchase_country": "SE",
  "locale": "sv-SE",
  "billing_address" : {
    "given_name": "Doe",
    "family_name": "John",
    "email": "direct_debit@klarna.com",
    "phone": "01895808221",
    "street_address": "Stårgatan 1",
    "postal_code": "12345",
    "city": "Ankeborg",
    "country": "SE"
  },
  "description": "MySaaS subscription",
  "intended_use": "subscription",
  "merchant_urls": {
    "confirmation": "string"
  }
}
```

?> The response will contain a redirect URL to which the user should be redirected. The user will bounce through Klarna and be redirected to the confirmation URL provided in the API calls.

```JSON
HTTP/1.1 200 OK

Content-Type: application/json
Klarna-Correlation-Id: e19dc121-1276-419d-882a-c343d58fb9aa

{
  "token_id": "0b1d9815-165e-42e2-8867-35bc03789e00",
  "redirect_url": "string"
}
```
### 3.4 Release Authorization 🆓
After creating a customer token, the authorized amount can be released if the `authorization_token` won't be used to place the order immediately. Releasing the authorized amount will free up available purchase amount for the user. This is done by performing a DELETE operation:
```JSON
DELETE /payments/v1/authorizations/{authorizationToken}
Authorization: Basic pwhcueUff0MmwLShJiBE9JHA==
Content-Type: application/json
{ }
```

Example response:
```JSON
HTTP/1.1 204 No Content

Content-Type: application/json
Klarna-Correlation-Id: e19dc121-1276-419d-882a-c343d58fb9aa

{ }
```




---
<h2 align="center">SMOOOTH BREAK</h2>

<div align="center">

  ![logo](https://media1.giphy.com/media/X6uVxBumRxgzA1ANfe/200w.webp?cid=3640f6095bda051062464c3667877051 )

</div>

---

## 4. Place Order 💸
> Once the order is authorized successfully, place the order using the authorization token from the Step 3. Authorize. For a recurring order, the customer_token from the previous step can be used.

After the order is created, you can manage it either manually via the Merchant Portal or through Klarna's [Order Management API](https://developers.klarna.com/en/gb/kco-v3/order-management)

### 4.1 The Merchant places a single order 👆
When you received the `authorization_token`, you can [place an order](https://developers.klarna.com/api/#payments-api-create-a-new-order).

!> If the order is placed more than 60 minutes after authorization_token is provided or different details are provided, order may be rejected.

_Example of order placement:_
```JSON
POST /payments/v1/authorizations/<authorization_token>/order
Authorization: Basic pwhcueUff0MmwLShJiBE9JHA==
Content-Type: application/json

{
    "purchase_country": "GB",
    "purchase_currency": "GBP",
    "billing_address": {
        "given_name": "John",
        "family_name": "Doe",
        "email": "john@doe.com",
        "title": "Mr",
        "street_address": "13 New Burlington St",
        "street_address2": "Apt 214",
        "postal_code": "W13 3BG",
        "city": "London",
        "region": "",
        "phone": "01895808221",
        "country": "GB"
    },
    "shipping_address": {
        "given_name": "John",
        "family_name": "Doe",
        "email": "john@doe.com",
        "title": "Mr",
        "street_address": "13 New Burlington St",
        "street_address2": "Apt 214",
        "postal_code": "W13 3BG",
        "city": "London",
        "region": "",
        "phone": "01895808221",
        "country": "GB"
    },
    "order_amount": 10,
    "order_tax_amount": 0, // optional
    "order_lines": [
        {
            "type": "physical", // optional
            "reference": "19-402", // optional
            "name": "Battery Power Pack",
            "quantity": 1,
            "unit_price": 10,
            "tax_rate": 0, // optional
            "total_amount": 10,
            "total_discount_amount": 0, // optional
            "total_tax_amount": 0, // optional
            "product_url": "https://www.estore.com/products/f2a8d7e34", // optional
            "image_url": "https://www.exampleobjects.com/logo.png" // optional
        }
    ],
    "merchant_urls": {
        "confirmation": "https://example.com/confirmation",
        "notification": "https://example.com/pending" // optional
    },
    "merchant_reference1": "45aa52f387871e3a210645d4", // optional
}
```
---
### 4.2 Handle Order 👷‍
!> You either get a successful response or an error
The response contains the following information:
- `order_id` - used for order management interactions, such as capturing (when Klarna pays the merchant) or refunding the order
- `redirect_url` - a URL to which you immediately redirect the user
- `fraud_status` - may indicate that this is a suspected fraudulent order

#### 4.2.1 Successful response: the order is confirmed ✅
You received `order_id`, `redirect_url` and `fraud_status`: ACCEPTED
_Example response:_
```JSON
{
  "order_id": "3eaeb557-5e30-47f8-b840-b8d987f5945d",
  "redirect_url": "https://payments.klarna.com/redirect/...",
  "fraud_status": "ACCEPTED"
}
```
?> Store `order_id` for future reference during order fulfillment or service center management.

!> In order for Klarna to securely handle the data and optimize the purchase flow, they need to interact with the customer's browser as a first party (not in an iframe or similar). <br> - _This is achieved by bouncing the browser to a klarna.com page before presenting the merchant's confirmation page._<br> - The merchant provides their confirmation page together with the rest of the order details and Klarna responds with a redirect url that will take the customer to the confirmation page. <br> - The merchant should send the customer's browser to the `redirect_url` provided in the response.


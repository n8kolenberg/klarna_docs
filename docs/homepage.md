# Klarna Implementation Documentation
> Klarna has two main products that we care about for the time being - I will add to this document as time goes on

!> [Klarna Checkout](/klarna_checkout)

!> [Klarna Payments](/klarna_payments)

<p align="center">
  <img src="https://res.cloudinary.com/n8dawg/image/upload/v1541408493/KCO_vs_KP.png">
</p>
<br>

For testing purposes, I used the following test credentials:
- Username: _PK03011_\__6c13f07cff67_
- Password: _CnXzI9E2K5E8NCtz_

> Note:
There's Klarna Checkout V2 and V3. V3 has benefits above using V2 in the sense that it will keep on receiving updates from Klarna and is done to be smooother to implement and use by merchants.

?> **The advantages of V3 above V2** <br> 
🕶️ Merchant Portal <br> 
🕶️ Order creation (acknowledge) ▶️ At order creation (confirmation) instead of push <br>
️️️️️🕶️️️️️️️️️ ️️️️️On Site Messaging <br> 
️️🕶 ️Klarna Shipping Selector <br> 
️️🕶 ️Pre-fill whole address ▶️ Not only email and postal code <br>
️️🕶 ️Settlement API ▶️ Fetching Settlements via API <br> 
️️🕶 ️Country Selector ▶️ No need to reload session when changing country <br> 
️️🕶 ️More use of JavaScript API <br> 
️️🕶 ️Order status ▶️ Follow the complete cycle of your order <br>
️️🕶 ️Global ▶️ Let merchants sell with cards to more countries <br>
️️🕶 ️Inline card <br>
🕶 Image URI ▶️ Making the post purchase UX better


## Klarna Checkout V3
<p align="center">
  <img  src="https://res.cloudinary.com/n8dawg/image/upload/v1541169707/Klarna%20Checkout%20Diagram.png">
</p>

--- 

> ### Order management of Klarna Payments and Klarna Checkout is the same
## Klarna Payments
<p align="center">
  <img  src="https://developers.klarna.com/static/Klarna_Payments-3e34293a104a177fc60851c9d2d97589-89906.png">
</p>



### So SMOOOTH
<p align="center">
  <img  src="https://res.cloudinary.com/n8dawg/image/upload/v1531110521/fish.gif">
</p>

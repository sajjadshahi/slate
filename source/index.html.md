---
title: Zibal API Refrence

toc_footers:
  - <a href='https://zibal.ir'>Zibal Main Page</a>
  - <a href='mailto:info@zibal.ir'>Contact Us</a>

includes:
  - callback
  - errors

search: true
---

# Introduction

Welcome to Zibal API Refrence.
This documentation is gathered to provide ease of usage of Zibal APIs.
In case of any questions and misunderstanding contact us ASAP!
We eagerly wait to respond you, don't hesitate!

<br>

There are some points we want you to consider before starting implementation:

* Zibal APIs are fully RESTful and all requests and responses are in the JSON format
* Do not forget to use `https` before URLs since all communications with our servers commence over SSL.
* All URLs here are under `https://sandbox-api.zibal.ir` domain which you may have read about it in panel's sandbox. Sandbox is developed for you in order to test all APIs and features so POS devices cannot recognize `zibalId`s which are returned from sandbox.
* In case you are ready to use Zibal service, make all requests to `https://api.zibal.ir/`.

# Authentication

> Every request you pass to zibal should contain `merchantId` and `secretKey`:

```json
{
    "merchantId": "Your Merchant ID",
    "secretKey": "Your Secret Key",
    // OTHER FIELDS
}
```
> Make sure to replace `Your Merchant ID` and `Your Secret Key` with the ones you got from info@zibal.ir

Zibal uses `merchantId` and `secretKey` to authorize its merchants. You must have got yours when registering your account from our official mail address: info@zibal.ir.

If you have lost them, or in case you needed to revoke them  [contact us](mailto:info@zibal.ir).

# Order

## Add Order
> Example JSON Request Object:

```json
{
  "merchantId": "Your Merchant ID",
  "secretKey": "Your Secret Key",
  "orderId": "25",
  "callbackUrl": "http://yourapiurl.com/callback.php",
  "amount": 150000,
  "percentMode": 0,
  "description": "Hello World!",
  "multiplexingInfos": [
  	  {
        "id": "self",
        "amount": 100000
      },
  	  {
        "id": "QEsxEq",
        "amount": 50000
      }
  	]
}
```
> In case you use multiplexing, the `id : self` is meant your company and is connected to your SHEBA code.

This endpoint is used to add a new order in Zibal.

### HTTP Request

`POST https://sandbox-api.zibal.ir/merchant/addOrder`

### Payload Parameters

Parameter | Required | Type | Description
--------- | ------- | ------- | -----------
merchantId | true | String | Necessary for authentication as mentioned before.
secretKey | true | String | Necessary for authentication as mentioned before.
orderId | true | String | Your unique order ID.
amount | true | Integer | Order total amount (In Rials)
callBackUrl | true | String | The URL which Zibal `POST`s payment's information to it.
description | false | String | Further description which can be displayed in POS devices. (e.g: Customer name, Product name etc.)
percentMode | false | 0 or 1 | Make it 1 if your multiplexing policies are based on percents. (Default: 0)
multiplexingInfos | false | Array | List of benficiaries in case you need to share the order amount. (as mentioned below)
<aside class="notice">
If you use multiplexing, make sure the total amount becomes equal to all benficiaries' share. 
</aside>
<aside class="notice">
If you do not send <code>multiPlexingInfos</code> all of the amount is transfered to your <code>self</code> account.
Else you should contain one object which its' <code>id</code> is <code>self</code> which means its your share of order. the <code>self</code> amount must be more than 10000 IRR. 
</aside>

### Response
> Example JSON Response Object:

```json
{
  "zibalId": 727,
  "secretKey": 10000,
  "result": 1,
  "message" : "درخواست شما با موفقیت انجام شد"
}
```
Parameter | Description
--------- | ------- 
zibalId | In case of success, Zibal returns `zibalId` which is the communication way on POS devices. It follows UPCA standard for barcode generation in case you are working with a barcode reader feature.
zibalFee | Zibal revenue based on order amount and features (in IRR)
result | The result code. you can see further description about result codes below.
message | English developer readable (!) message about your request response


## Read Order
> Example JSON Request Object:

```json
{
  "merchantId": "Your Merchant ID",
  "secretKey": "Your Secret Key",
  "orderId": "25"
}
```
This endpoint retrieves details of a specific order.

<aside class="notice">Zibal will <code>POST</code> each order details when payment is successful to the <code>callbackUrl</code> you send to us in <a href="#add-order">add order API</a>.
Instead, you can retrieve all details from Read Order API each time you want. 
</aside>

### HTTP Request

`GET https://sandbox-api.zibal.ir/merchant/readOrder`

### Payload Parameters

Parameter | Required | Type | Description
--------- | ------- | ------- | -----------
merchantId | true | String | Necessary for authentication as mentioned before.
secretKey | true | String | Necessary for authentication as mentioned before.
orderId | or zibalId | String | The `orderId` of the order you want to retrieve the details. The unique id which you have recently sent to Zibal in [add order](#add-order). 
zibalId | or orderId | Integer | Zibal's unique ID for each order which has been returned to you in [add order](#add-order)'s response

### Response
> Example JSON Response Object:

```json
{
  "orderId": "250",
  "result": 1,
  "refNumber": null,
  "createdAt": "2017-01-25T23:43:01.053000",
  "paidAt": "2017-02-27T09:24:31.053000",
  "amount": 150000,
  "zibalFee": 10000,
  "status": 1,
  "zibalId": 17,
  "description": "Hello World!",
  "canceledAt": null,
  "message": "success",
  "multiplexingInfos": [
    {
      "bankAccount": "IR120620000000322394250009",
      "amount": 100000
    },
    {
      "bankAccount": "IR120620000000202374130392",
      "amount": 50000
    }
  ],
}
```

Parameter | Description
--------- | ------- 
result | The result code. you can see further description about result codes below.
message | English developer readable (!) message about your request response.
zibalId | Zibal's unique ID for each order
orderId | The `orderId` of the order you want to retrieve the details. The unique id which you have recently sent to Zibal in [add order](#add-order). 
zibalFee | Zibal revenue based on order amount and features (in IRR)
status | Order status, as described below
refNumber | In case the order is paid bank's refrence number
createdAt | ISO-Date which you have submitted the order.
paidAt | In case the order is paid ISO-Date for payment time.
canceledAt | In case the order is cancelled ISO-Date for cancellation time.
multiplexingInfos | In case you have used multiplexing feature, the list of beneficiaries and their SHEBA number as `bankAccount` and their share as `amount` is returned

<aside class="notice">
  As you may have noticed, read order response containing <code>multiplexingInfos</code> is an object containing <code>bankAccount</code> which is SHEBA number instead of <code>code</code>.
</aside>

### Order Status Table

Status | Status Message
--------- | ------- 
0 | Pending
1 | Paid
2 | Cancelled

## Cancel Order
> Example JSON Request Object:

```json
{
  "merchantId": "Your Merchant ID",
  "secretKey": "Your Secret Key",
  "orderId": "25"
}
```
This endpoint changes an order's status to cancelled.

<aside class="warning">
Please make sure before using this API, since when an order is cancelled, it cannot be paid with POS devices. 
</aside>

### HTTP Request

`GET https://sandbox-api.zibal.ir/merchant/cancelOrder`

### Payload Parameters

Parameter | Required | Type | Description
--------- | ------- | ------- | -----------
merchantId | true | String | Necessary for authentication as mentioned before.
secretKey | true | String | Necessary for authentication as mentioned before.
orderId | or zibalId | String | The `orderId` of the order you want to retrieve the details. The unique id which you have recently sent to Zibal in [add order](#add-order). 
zibalId | or orderId | Integer | Zibal's unique ID for each order which has been returned to you in [add order](#add-order)'s response

### Response

> Example JSON 

```json
{
  "result": 0,
  "message": "success"
}
```

Parameter | Descirption
-------- | --------
result | The result code. you can see further description about result codes below.
message | English developer readable (!) message about your request response.


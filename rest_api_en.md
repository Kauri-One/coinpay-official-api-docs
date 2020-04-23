- [Request processing](#request-processing)
- [Public data](#public-data)
  - [Getting active currencies list](#getting-active-currencies-list)
  - [Getting active pairs list](#getting-active-pairs-list)
  - [Getting prices](#getting-prices)
  - [Calculating exchange cost](#calculating-exchange-cost)
- [User data](#user-data)
  - [Creating account](#creating-account)
  - [Account types](#account-types)
  - [Authorization types](#authorization-types)
  - [JWT authorization](#jwt-authorization)
  - [Authorization by request signature](#authorization-by-request-signature)
  - [Getting balance](#getting-balance)
  - [Retrieving account settings](#retrieving-account-settings)
- [Orders](#orders)
  - [Getting order details](#getting-order-details)
  - [Getting order history](#getting-order-history)
  - [Callback notification](#callback-notification)

## Request Processing
* Base api url - https://coinpay.org.ua/
* Content-Type – json

Each response contains a structure in which there is a ‘status’ field indicating if such response was processed successfully. A successfully processed response has 2XX status code, and an unsuccessfully processed one - either 4XXX or 5XX.

###### Example of a successfully processed request:

```javascript
 {"status": "success"}
```

###### Example of an unsuccessfully processed request:

```javascript
 {"status": "error", "error": <имя ошибки>}
```

There are no error codes; there is only a verbally described reason.
By default, all errors are returned in English.
It is possible to localize errors in Russian by adding the following title to the request itself – ‘Accept-Language’ with the value ‘ru’.

# Public Data
## Getting active currencies list

```javascript
 GET "/api/v1/currency"
```

###### Sample response:

```javascript
{
  "status": "success",
  "currencies": ["BTC","EUR",...]
}

```

The system returns a list of currencies active at the moment as well as the name of a currency itself, as identified in the system.

## Getting active pairs list
`` `javascript
GET "/ api / v1 / pair"
```

###### Sample response:
```javascript
{
  "status": "success",
  "pairs": [
    [
      {
        "currency_to_spend": "UAH",
        "currency_to_get": "BTC",
        "name": "BTC_UAH",
        "base_currency": "BTC"
      }
    ],
```

API endpoint returns a list of active pairs by which exchanges can be made.

** Explanation of a pair name: **

Usually two currencies are involved in an exchange; one of them is what someone spends, another one – what they receive.
In the current example, ‘BTC_UAH’ pair stands for buying BTC for UAH.
In order to purchase UAH for BTC, one needs to use another pair – ‘UAH_BTC’. In fact, we only have one pair; this is the direction of exchange.

## Getting prices

```javascript
 GET "api/v1/exchange_rate"
```

###### Sample response:

```javascript
{  
  "rates": [  
    {  
      "pair": "UAH_EUR",  
      "base_currency_price": 0.03800836,  
      "price": 26.31  
    }  
  ]  
}                              
																						
```

For the current API endpoint, you can get all the prices for all pairs for an exchange (ticker).


## Calculating exchange cost

```javascript
 GET "api/v1/exchange/calculate"
```

###### Parameters:

```javascript
currency_to_get - required. The currency a user receives
currency_to_spend - required. The currency a user spends
currency_to_get_amount, currency_to_spend_amount - In order to calculate, you need to specify one of the parameters, either ‘the amount of the currency one wants to receive’ or ‘the amount of the currency that a user wants to spend’
exchange_price is an optional parameter. You can specify an exchange price; if the price is not specified, market price is used.
```

###### Sample response:

```javascript
{
  "currency_to_get_amount": 1,
  "currency_to_get": "BTC",
  "currency_to_spend_amount": 202170.7475,
  "currency_to_spend": "UAH",
  "status": "success",
  "exchange_price": 202170.7475
}
```

At each calculation, a response always returns the calculated exchange (exchange price, how much will be spent, the currency, and how much will be received).

# User data
## Creating account

```javascript
  POST "api/v1/user/create"
```

###### Parameters:

```javascript
{
    "email": "string",
    "username": "string",
    "password": "string",
}                                                  

```

All the above fields are required. After API request is successfully completed, one needs to confirm their email (a mail will be sent to their email address); on can go through authorization only after confirming their email.
In the body of the mail, there is a link one needs to click to confirm their email. Link lifetime is 1 day.

## Account types
After a user has confirmed their email, their account becomes ‘NOT_VERIFIED’.
After verification, the account changes to ‘VERIFIED’.
The third account type is designated for business users (merchants) – ‘BUSINESS’.

## Authorization types
Currently, there are two types of API authorization - with JWT token and with request signature through api_key and api_secret.
Users choose the preferable authorization type themselves. We recommend using the request signature.

### JWT authorization


```javascript
POST "api/v1/user/obtain_token"
```
###### Parameters:

```javascript
{
  "email": "string",
  "password": "string"
}                                              

```

###### Example of a successful response (token received):
```javascript
{
  "username": "<username>",
  "token": "<token>"
}
```

After a token has been received, one can perform requests that require authorization.
The token must be specified in the header --header 'Authorization: Bearer <token> header. Token lifetime is 15 minutes.
The token for ‘Swagger’ (through Authorize button) is specified in the same format so that it reboots and shows all available methods allowed to a user.

After the token expires, it should be ‘reobtained’ through ‘obtain token’ method.

### Authorization by request signature
After passing the verification, api_key and api_secret will be provided.
In order to use authorization by request signature, one needs to provide two headers:
```javascript
--header 'Sign: <generated_signature>' --header 'Key: <api_key>'                                          
```

###### Signature creation algorithm:
** Step 1: ** You need to sort the request parameters in alphabetical order and transform nesting in parameters, if there is any.
** Example without nesting: **
```javascript
{
  "currency": "string",
  "amount": "string"
}							
```

In this case, you should get a dictionary, but with sorted keys:
```javascript
{
  "amount": "string",
  "currency": "string"
}							
```

** Example if there is nesting: **
```javascript
{
  "currency": "string",
  "amount": "string",
  "additional_info": {"client_ip": "8.8.8.8"}
}
```

** Transforms into the following dictionary: **
{
  "currency": "string",
  "amount": "string",
  "additional_info_client_ip": "8.8.8.8"
}

The keys of the received dictionary should be sorted in alphabetical order.

** Step 2: ** A line for signature is created in this format:
```javascript
{request.method} - {request.path} - {sorted_params}
```

** Step 3: ** The api_secret string is signed using the algorithm hmac_sha_256.
Similar manipulations are to be carried out with each request.

## Getting balance

```javascript
  GET "api/v1/user/balance"
```

###### Sample response:
```javascript
{
"wallets": {
    "USDT": {
      "address": "0x85E12eA5270C8848D9258FB76de475144dBC1E3e",
      "qr_file_data": img in base64
    },
    ...
  },
  "balance": {
    "EUR": {
      "currency": "EUR",
      "EUR": {
        "reserved": 0,
        "total": 0
      },
      "ETH": {
        "reserved": 0,
        "total": 0
      },
      "UAH": {
        "reserved": 0,
        "total": 0
      },
      "USDT": {
        "reserved": 0,
        "total": 0
      },
      "USD": {
        "reserved": 0,
        "total": 0
      },
      "RUB": {
        "reserved": 0,
        "total": 0
      },
      "BTC": {
        "reserved": 0,
        "total": 0
      }
    },
    ...
  }
}                                                  

```

The balance contains crypto addresses that can be replenished; the addresses are assigned to the users. The balance itself also comes in two fields, ‘total’ and ‘reserved’. ‘Total’ stands for the total amount of funds, ‘Reserved’ – for the funds already taken and located in the orders. Using the balance, you can find out how much money is currently available in a particular currency; moreover, you can get a conversion of funds from one currency to another.

**For example:**

1) The amount of ‘EUR’ in your account - balance [‘EUR’] [‘EUR’]

2) The amount of ‘EUR’ in ‘ETH’ - balance [‘EUR’] [‘ETH’]

## Getting account settings
```javascript
  GET "api/v1/user/account_info"
```

```javascript
  {
  "name": "Свят",
  "email": "svyatoslavkravchenko@gmail.com",
  "account_type": "VERIFIED",
  "is_email_verified": true,
  "referral_id": "89f67887-cae6-4a6f-a393-a89079878a57",
  "created": "2019-02-05T08:13:48.952334",
  "is_2fa_enabled": false,
  //limits, fee, processing_rules
  }
```

A response includes a username, email address, type of account, information on whether such mail has been verified, whether two-factor authorization is enabled, as well as limits, personal funds, etc. for each order type. Each order will be considered separately

# Orders
## Getting order details
When creating an order through API, the system gives the order_id of such order, thus giving the opportunity to get the details.
```javascript
GET "api/v1/orders/detail"
```
###### Параметры:

```javascript
{
  "order_id": $order_id
}						
```

The response provides order details such as whether the order has been created by a specific account or the account has participated in this order. There are orders in which several accounts participate.
The chapter provides an example of what exactly each order returns, for each order type.
## Getting order history
```javascript
GET "api/v1/orders/history"
```
###### Sample response:

```javascript
{
  "orders": [<orders_details_list>]
  "status": "success",
  "pages_count": 78
}
```

Order history has paging with 10 orders displayed on a page.

###### Filters:
  - order_type - the type of an order. Possible values are ‘DEPOSIT’, ‘WITHDRAWAL’, ‘EXCHANGE’, ‘INTERNAL_MOVEMENT’, ‘INVOICE’.

- order_status - the status of an order. Possible values are ‘NEW’, ‘ERROR’, ‘CLOSED’, ‘EXPIRED’, ‘CANCELLED’, ‘CANCELLING’, ‘WAITING_FOR_OPERATOR_CONFIRMATION’, ‘CONFIRMED_BY_OPERATOR’, ‘CANCELLED_BY_OPERATOR’, ‘WAITING_FOR_CONFIRMATION’, ‘WAITING_FOR_PRICE’, ‘PAYMENT_IN_PROGRESS’.
- order_sub_type - the subtype of an order. Some orders have their own subtypes that can be used to filter them.
- page - the page for withdrawal.
- from_timestamp, till_timestamp - filtering by time of order creation.
- currency - filtering by currency. It displays all orders for a particular currency.
- address finds all orders with an address. They can be either replenishment or withdrawal orders.

### Callback notification
When creating an order, it is possible to specify the url for notification when the status of such order changes.
With callback notification, a POST request is sent to the specified address. The 200th response will be a successful request processing, and if an error occurs, the system will try to send a callback 3 times with an interval of 5 minutes.

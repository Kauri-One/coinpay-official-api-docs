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

---
title: "Card Payments and Authorization Holds"
---

# Card Payments and Authorization Holds


## Overview


Most end users today expect to be able to function in a fully cashless way, using debit and credit cards to make and recevive payments as well as make direct withdrawals from debit accounts and cash advances against credit.

Mambu fully supports industry standard Authorization Hold flows for card payments and allows you to request, adjust, reverse and settle holds against Current Account type deposit products.


### Requirements


- You must be using API 2.0
- The account against which you wish to make an Authorization Hold request must be of the type "Current Account"
- "Cards" functionality must be enable for your instance
- There must be a Card Token Reference associated with the account

{{% notice info  %}}
A Card Token Reference is used to identify a Credit or Debit card without needing to always provide sensitive information such as the card number and, as such, is a shared reference between your instance and your card issuer's system
{{% /notice %}}


## Authorization Hold Flow


When paying with a credit or debit card there can often be a delay between the intitiation of a payment and transaction being verified or cancelled. In such cases, the common banking industry practice is to issue an **Authorization Hold** which will block this amount from the client's account, ensuring that the full amount is available when the transaction is verified and settled. Another common example is when a hold is made for a deposit when renting a car or hotel room which will be lifted if no incidental charges are incurred during the rental period. This is also sometimes referred to as a 'pre-authorisation'.

Industry best practices suggest that Authorization Holds should automatically expire if no action is taken after no more than 7 days for debit card transactions and no more than 30 days for credit card transactions.

### Authorization Requests


An Authorization Request is used to specify the amount to be held and can contain optional information on the merchant. If not otherwise specified the amount will always be in the base currency of the account.

Successful requests will return a reference ID and will be created with the statu "PENDING".

Requests can fail for a number of reasons such as the client is inactive or blacklisted, or the account does not have sufficient available balance (see below for more information and example calculations).

### Retrieving Authorization Holds


API requests can be used to retrieve either an array of [all Authorization Holds](https://api.mambu.com/#depositaccounts-getallauthorizationholds) for a given deposit account or [details on a specific Hold](https://api.mambu.com/#getauthorizationholdbyid) on a given card by its ID.

### Impact on Account Balance


An Authorization Request is made to specify an amount awaiting authorization. This amount will be reflected in the cardholder's available balance and will be unavailable to the cardholder until the merchant either settles or voids the transaction, or the hold expires (see [below](/#expiration-of-authorization-holds)).

An account can have multiple pending Authorization Requests making a total Holds Balance which is used for calculating the Available Balance for an end user.

### Hold Balance vs Locked Balance


When account holders use their balance as a guarantee for a loan, a portion of it can be locked, essentially making it unavailable for use. This will also affect the success or failure of Authorization Hold Requests.

Consider the following example:
For a deposit account with current data:

- Booked Balance of 1000
- Overdraft limit of 1500(see note)
- Locked Balance of 500
- Holds Balance of 100


  - -> available balance = 400 ([see here](https://support.mambu.com/docs/working-with-deposit-accounts) for more details)



|Hold Amount Request|	Authorization Request Response|	Holds Balance|	Available Balance|	Notes|
|-------------------|-------------------------------|--------------|-------------------|-------|
|100|	success|	200|	300|	The overdraft limit is not taken into consideration when there is a positive locked balance (the account acts like it has overdraft disabled).|
|400|	success|	500|	0|	The hold request is successful as it reaches but does not exceeed the Available Balance of the account|
|401|	fail|	100|	400|	Despite the overdraft limit, the hold request fails since the booked balance may become lower than the amount under lock.|

{{% notice note %}}
As a loan can not be used as a guarantee for another loan, when an account has a Locked Balance, it acts as if the overdraft facility has been disabled. This means that the overdraft limit is no longer taken into account for the calculation of the Available Balance
{{% /notice %}}

### Expiration of Authorization Holds


By default Authorization Holds will expire after a period of 7 days although this can be changed to reflect your bank's policies using the Administration Panel. You can also set different expiry periods for specific Merchant Categories, in which case, this period will be used for Holds requested for this Merchant Category Code.

Every hour, a scheduled job will run to automatically cancel any Authorization Hold Requests on which no action has been taken before the expiration you have set.

When a hold has expired its status will change to **EXPIRED** and the hold amount will no longer influence the end user's available balance.


### Adjusting Authorization Holds


Authorization Holds with a **PENDING** status can be [increased](https://api.mambu.com/#increaseauthorizationhold) or [decreased](https://api.mambu.com/#decreaseauthorizationhold) by making an API request and supplying the Card Token Reference and Authorization Hold Request ID along with the amount with which to increase or decrease the held amount.

A decrease of the same or more than the full amount of the hold will effectively reverse the hold (see below).


### Reversing Authorization Holds


You can reverse an Authorization Hold either fully or partially using the [decrease](https://api.mambu.com/#decreaseauthorizationhold) endpoint either if the transaction is aborted or the cardholder will be charged a lesser amount.

A fully reversed Authorization Hold will receive the status **REVERSED** and will no longer affect the cardholder's available balance.

Partially reversed Authorization Hold's will retain the **PENDING** status and its expiry period will be reset, effectively restarting it from the point at which the partial reversal was made.

**Example:**

- Hold amount = 100 -> Decrease = 75 -> New Hold amount = 25.



### Settling Authorization Holds


Notification to settle an Authorization Hold and create an actual transaction will come as Financial Advice from a card acquirer. Usually this Advice will be to debit the full amount of a previously requested Authorization Hold from a cardholder's account. It is possible, however, that the Advice will be for more or less than the amount under Hold, or that it comes without any prior Authorization Hold having been made at all.

Debits will be recorded with the transaction domain code 'Payments', family code 'Merchant Card Transactions' and sub-family code 'Credit Card Payment'.

Once the transaction has been completed the Authorization Hold will be updated to include the transaction ID and have its status set to **SETTLED**.




### Refunding Previously Settled Authorization Holds


Once an Authorization Hold has been settled and monies already debited from a cardholder's account it can no longer be reversed and must be refunded by way of a new transaction for the full amount charged or, in the case of a partial refund, less than the originally charged amount.

Any interest applied before the refund has been made will not be affected and should be considered seperately.


## Other Card Payment Operations


### Offline Transactions


In cetain situations transactions may be made without the possibility to communicate with a card issuer's system. For example, a system with no internet connectivity, such as an airplane while in flight or when transactions are processed in a batch at a set time period like the end of a business day. Such transactions are commonly known as **Offline Transactions**.

These transactions are made using the same API request as a standard Authorization Hold but with the **advice** flag set to true. This will cause the request to be made without validation and as a result can both  allow a cardholder to make a transaction for an amount over any [defined limits](https://support.mambu.com/docs/setting-up-new-deposit-products#deposits-and-withdrawals), as well as allow for usage without the normal balance checks, causing an account to go into [Technical Overdraft](https://support.mambu.com/docs/technical-overdraft/).



### Financial Requests


For transactions such as ATM withdrawls or PIN Debits, which are not routed via a credit card processing network, Financial Requests are used to authorize an amount and immediately debit the cardholder's account with a single request.

Financial Requests can only be made in the base currency of an account given that it has sufficient Available Balance and is active and not currently blocked.

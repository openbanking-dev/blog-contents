# Overview of the Open Banking UK APIs.

*This article is part of a set of tutorial about Open Banking UK. If you are interested to learn about it from the beginning, you may want to check-out our first intro article.*

Before going into a deep dive in the next articles into how Open Banking UK works, let's talk about the APIs and what they have to offer to you.

There are four different types of APIs:

- Account and Transaction
- Payments
- Event notification
- Confirmation of funds

## Account and Transaction

That's the name used by the standard, and no, it's not just two APIs, ie one to see the user account and another to see the transactions.
Those APIs will cover the access to all the user financial data. 
I usually compare them with my bank web portal. Basically, everything you can retrieve from your bank portal, you will _pretty much_ be able to access them via APIs.

Here is the list of the APIs:

- Get accounts
- Get account
- Get balances
- Get transactions
- Get beneficiaries
- Get direct-debits
- Get product
- Get standing-orders
- Get party
- Get offers
- Get scheduled-payments
- Get statements
- Get statement transactions

As you can notice, they are all `Get` APIs, meaning you got a read access privilege. It's kind of expected for some of them, don't think you ever imagine you could create transactions!
Although for APIs like standing orders, you may have wish a way to create new one. It's actually covered by the payment APIs.


Before we got in the details of each APIs, let explain the concept of bulk.
Some APIs will offer you two modes: Data for a specific account or for all accounts: the last one is called bulk. The easiest is to take an example, the `Get transactions`.

- You can request the transactions for a specific account
- You can request the transaction for all the accounts at the same time -> Bulk

They could have designed the APIs to take an account id in the GET parameters. Instead, they choose to define the account id in the path. It's no big deal, just thought you needed to be aware of it, as it creates this concept of `bulk`.
What is good for us is the fact that the output formats of the two modes, are the same. 


### Get accounts

It's the first one you will use most likely. It gives you the list of bank accounts owned by the user that he choose to share with you. Yes, the user has controls over what accounts he can share with you. He may have choose to share access all of his account, or a subset. That's why starting by 'get accounts' is a good way for you to know the list of accounts the user is sharing with you.

Here is an example of what you would received from this API.
The important bit for the using the other APIs is the `AccountId`. It will be the id you would need to use to filter the resources to that specific account.

```
GET /open-banking/v3.1.1/aisp/accounts/
```

```
{
    "Data": {
        "Account": [
            {
                "AccountId": "12a03d84-1a34-46c6-acd1-53065e80d282",
                "Currency": "GBP",
                "AccountType": "Personal",
                "AccountSubType": "CurrentAccount",
                "Nickname": "UK Bills",
                "Account": [
                    {
                        "SchemeName": "SortCodeAccountNumber",
                        "Identification": "73815621781678",
                        "Name": "test_b0ee1d08-3e58-49cb-b5f2-24fae6fe399b",
                        "SecondaryIdentification": "62196167"
                    }
                ]
            },
            {
                "AccountId": "6e812bff-fa29-4f2d-840e-bf696935c348",
                "Currency": "EUR",
                "AccountType": "Personal",
                "AccountSubType": "CurrentAccount",
                "Nickname": "FR Bills",
                "Account": [
                    {
                        "SchemeName": "SortCodeAccountNumber",
                        "Identification": "9836217030285",
                        "Name": "test_b0ee1d08-3e58-49cb-b5f2-24fae6fe399b",
                        "SecondaryIdentification": "63466758"
                    }
                ]
            },
            {
                "AccountId": "c1ca1db5-8cc2-44ff-9685-b3bec5e4c762",
                "Currency": "GBP",
                "AccountType": "Personal",
                "AccountSubType": "CurrentAccount",
                "Nickname": "Household",
                "Account": [
                    {
                        "SchemeName": "SortCodeAccountNumber",
                        "Identification": "835084690427",
                        "Name": "test_b0ee1d08-3e58-49cb-b5f2-24fae6fe399b"
                    }
                ]
            }
        ]
    },
    "Links": {
        "Self": "https://matls.rs.aspsp.ob.forgerock.financial/open-banking/v3.1.1/aisp/accounts"
    },
    "Meta": {
        "TotalPages": 1
    }
}
```

### Get account

You may want to get just a specific account that you already got the id for. In a way, the `Get accounts` is the bulk version of `Get account`.

The output is the same than the `get accounts` but with one account in the `Data.Account` array.

```
GET /open-banking/v3.1.1/aisp/accounts/12a03d84-1a34-46c6-acd1-53065e80d282
```

```
{
  "Data": {
    "Account": [
      {
        "AccountId": "055bdf14-9b5d-42b5-bdb3-c3e118ecee02",
        "Currency": "GBP",
        "Nickname": "Bills",
        "Account": [
          {
            "SchemeName": "SortCodeAccountNumber",
            "Identification": "33775040872714",
            "Name": "demo",
            "SecondaryIdentification": "36906218"
          }
        ]
      }
    ]
  },
  "Links": {
    "Self": "https://matls.rs.aspsp.ob.forgerock.financial/open-banking/v3.1.1/aisp/accounts/055bdf14-9b5d-42b5-bdb3-c3e118ecee02"
  },
  "Meta": {
    "TotalPages": 1
  }
}
```


### Get balances

You can retrieve the balance of an account. It's good to note that an account can have multiple balances. I first thought it was a 1-1 relationship but it's not. An account can have one balance of each type. 

By account
```
GET /open-banking/v3.1.1/aisp/accounts/12a03d84-1a34-46c6-acd1-53065e80d282/balances
```

Bulk version
```
GET /open-banking/v3.1.1/aisp/balances
```


```
{
    "Data": {
        "Balance": [
            {
                "AccountId": "12a03d84-1a34-46c6-acd1-53065e80d282",
                "CreditDebitIndicator": "Debit",
                "Type": "InterimAvailable",
                "DateTime": "2019-10-23T11:00:16.290Z",
                "Amount": {
                    "Amount": "4078.75",
                    "Currency": "GBP"
                }
            }
        ]
    },
    "Links": {
        "Self": "https://matls.rs.aspsp.ob.forgerock.financial/open-banking/v3.1.1/aisp/accounts/12a03d84-1a34-46c6-acd1-53065e80d282/balances"
    },
    "Meta": {
        "TotalPages": 1
    }
}
```

### Get transactions

Our super star, the transactions APIs. It allows you to retrieve the transactions of an account or all accounts.
I just let you discover the payload, it self-explanatory.

By account
```
GET /open-banking/v3.1.1/aisp/accounts/12a03d84-1a34-46c6-acd1-53065e80d282/transactions
```

Bulk version
```
GET /open-banking/v3.1.1/aisp/transactions
```

```
{
    "Data": {
        "Transaction": [
            {
                "AccountId": "12a03d84-1a34-46c6-acd1-53065e80d282",
                "TransactionId": "d0487833-e980-4a45-a44e-a6aa68521f01",
                "TransactionReference": "Ref 7871",
                "CreditDebitIndicator": "Debit",
                "Status": "Booked",
                "BookingDateTime": "2018-10-01T21:06:37.352Z",
                "ValueDateTime": "2018-10-01T21:09:59.352Z",
                "Amount": {
                    "Amount": "264.37",
                    "Currency": "GBP"
                },
                "BankTransactionCode": {
                    "Code": "ReceivedCreditTransfer",
                    "SubCode": "DomesticCreditTransfer"
                },
                "ProprietaryBankTransactionCode": {
                    "Code": "Transfer",
                    "Issuer": "AlphaBank"
                },
                "TransactionInformation": "Cash to Snider, Linda L.",
                "Balance": {
                    "Amount": {
                        "Amount": "4078.75",
                        "Currency": "GBP"
                    },
                    "CreditDebitIndicator": "Debit",
                    "Type": "InterimBooked"
                }
            },
            ...
        ]
    },
    "Links": {
        "Self": "https://matls.rs.aspsp.ob.forgerock.financial/open-banking/v3.1.1/aisp/accounts/12a03d84-1a34-46c6-acd1-53065e80d282/transactions",
        "First": "https://matls.rs.aspsp.ob.forgerock.financial/open-banking/v3.1.1/aisp/accounts/12a03d84-1a34-46c6-acd1-53065e80d282/transactions?page=0",
        "Next": "https://matls.rs.aspsp.ob.forgerock.financial/open-banking/v3.1.1/aisp/accounts/12a03d84-1a34-46c6-acd1-53065e80d282/transactions?page=1",
        "Last": "https://matls.rs.aspsp.ob.forgerock.financial/open-banking/v3.1.1/aisp/accounts/12a03d84-1a34-46c6-acd1-53065e80d282/transactions?page=23"
    },
    "Meta": {
        "TotalPages": 24
    }
}
```

### Get beneficiaries

Something I am quite used to as a bank customer, is to define the beneficiaries of my future direct debits.
This API allows you to retrieve all the beneficiary that the user has defined in his bank.

By account:
```
GET /open-banking/v3.1.1/aisp/accounts/12a03d84-1a34-46c6-acd1-53065e80d282/beneficiaries
```

Bulk version
```
GET /open-banking/v3.1.1/aisp/beneficiaries
```

```
{
    "Data": {
        "Beneficiary": [
            {
                "AccountId": "12a03d84-1a34-46c6-acd1-53065e80d282",
                "BeneficiaryId": "4812b831-1a73-42c1-b671-b8eebc66736d",
                "Reference": "Nam Incorporated",
                "CreditorAccount": {
                    "SchemeName": "UK.OBIE.SortCodeAccountNumber",
                    "Identification": "41587980350025",
                    "Name": "Dickerson, Keith O."
                }
            },
            ...
        ]
    },
    "Links": {
        "Self": "https://matls.rs.aspsp.ob.forgerock.financial/open-banking/v3.1.1/aisp/accounts/12a03d84-1a34-46c6-acd1-53065e80d282/beneficiaries"
    },
    "Meta": {
        "TotalPages": 1
    }
}
```

### Get direct-debits

You can get all the direct debits from an account or all accounts.


By account
```
GET /open-banking/v3.1.1/aisp/accounts/12a03d84-1a34-46c6-acd1-53065e80d282/direct-debits
```

Bulk version
```
GET /open-banking/v3.1.1/aisp/accounts/12a03d84-1a34-46c6-acd1-53065e80d282/direct-debits
```

```
{
    "Data": {
        "DirectDebit": [
            {
                "AccountId": "12a03d84-1a34-46c6-acd1-53065e80d282",
                "DirectDebitId": "870db938-7d69-429e-a705-7037bf061cce",
                "MandateIdentification": "Enim Corporation",
                "DirectDebitStatusCode": "Active",
                "Name": "Enim Corporation",
                "PreviousPaymentDateTime": "2019-09-23T11:00:16.295Z",
                "PreviousPaymentAmount": {
                    "Amount": "240.44",
                    "Currency": "GBP"
                }
            },
            ...
        ]
    },
    "Links": {
        "Self": "https://matls.rs.aspsp.ob.forgerock.financial/open-banking/v3.1.1/aisp/accounts/12a03d84-1a34-46c6-acd1-53065e80d282/direct-debits"
    },
    "Meta": {
        "TotalPages": 1
    }
}
```

### Get product

Product is probably something you are less used to. It's an API to retrieve what kind of financial product is behind the account.
It's a way to know if it's a current account, a saving, a mortgage, etc kind of account.

By account
```
GET /open-banking/v3.1.1/aisp/accounts/12a03d84-1a34-46c6-acd1-53065e80d282/product
```

Bulk version
```
GET /open-banking/v3.1.1/aisp/product
```

```
{
    "Data": {
        "Product": [
            {
                "ProductName": "321 Product",
                "ProductId": "b872d7b6-d5e2-439e-b104-8c43b751d5cc",
                "AccountId": "12a03d84-1a34-46c6-acd1-53065e80d282",
                "ProductType": "PersonalCurrentAccount"
            }
        ]
    },
    "Links": {
        "Self": "https://matls.rs.aspsp.ob.forgerock.financial/open-banking/v3.1.1/aisp/accounts/12a03d84-1a34-46c6-acd1-53065e80d282/product"
    },
    "Meta": {
        "TotalPages": 1
    }
}
```

### Get standing-orders

You can get all the standing order from an account or all accounts.

By account
```
GET /open-banking/v3.1.1/aisp/accounts/12a03d84-1a34-46c6-acd1-53065e80d282/standing-orders
```

Bulk versions
```
GET /open-banking/v3.1.1/aisp/standing-orders
```

```
{
    "Data": {
        "StandingOrder": [
            {
                "AccountId": "12a03d84-1a34-46c6-acd1-53065e80d282",
                "StandingOrderId": "87d4162c-9412-4209-be0f-7b8c90917b1b",
                "Frequency": "EvryWorkgDay",
                "Reference": "Habitant Morbi Tristique Industries",
                "FirstPaymentDateTime": "2018-10-23T11:00:16.351Z",
                "NextPaymentDateTime": "2019-12-23T11:00:16.351Z",
                "FinalPaymentDateTime": "2029-10-23T11:00:16.351Z",
                "StandingOrderStatusCode": "Active",
                "FirstPaymentAmount": {
                    "Amount": "206.51",
                    "Currency": "GBP"
                },
                "NextPaymentAmount": {
                    "Amount": "206.51",
                    "Currency": "GBP"
                },
                "CreditorAccount": {
                    "SchemeName": "UK.OBIE.SortCodeAccountNumber",
                    "Identification": "83688949953407",
                    "Name": "Ratliff, Jamal J."
                }
            },
            ...
        ]
    },
    "Links": {
        "Self": "https://matls.rs.aspsp.ob.forgerock.financial/open-banking/v3.1.1/aisp/accounts/12a03d84-1a34-46c6-acd1-53065e80d282/standing-orders"
    },
    "Meta": {
        "TotalPages": 1
    }
}
```

### Get party

This API is about getting the party, and by party, we mean the identity of the bank account.

This one is a bit special. Remember what I said earlier about bulk? Forget about it for that API.
There isn't the concept of bulk for this one, even if the path convention suggest there is.

Let's look first at the one by account.
It will return the identity of the owner(s) for the select account.
The user may have shared a joined account and therefore, the identity associated with the account is not only the current user.


```
GET /open-banking/v3.1.1/aisp/accounts/12a03d84-1a34-46c6-acd1-53065e80d282/party
```

```
{
    "Data": {
        "Party": {
            "PartyId": "bbeaa758-27c8-4ba7-89da-6a8d83ed4208",
            "Name": "test_b0ee1d08-3e58-49cb-b5f2-24fae6fe399b"
        }
    },
    "Links": {
        "Self": "https://matls.rs.aspsp.ob.forgerock.financial/open-banking/v3.1.1/aisp/accounts/12a03d84-1a34-46c6-acd1-53065e80d282/party"
    },
    "Meta": {
        "TotalPages": 1
    }
}
```


Now let's talk about the /party. As I said, you may think it's a bulk like the others but it's not.
It's actually returning the identity of the current user. 

Personal note: Haven been working for an identity provider, this endpoint has always look dodgy to me. The reason I never been a huge fan of it, is because identity should be provided via an ID token in my opinion. There is already standards to do that, I am not entirely convinced we needed a dedicated API for it.
The reason behind it is surely more political than technical. Defining standards is about compromise and I do feel that this `/party` is an illustration of it.


```
GET /open-banking/v3.1.1/aisp/party
```

_note that the PartyID is different than the previous payload_
```
{
    "Data": {
        "Party": {
            "PartyId": "4eaf2f08-7da8-4c19-b4ea-3ebe0e6caac4",
            "Name": "test_b0ee1d08-3e58-49cb-b5f2-24fae6fe399b"
        }
    },
    "Links": {
        "Self": "https://matls.rs.aspsp.ob.forgerock.financial/open-banking/v3.1.1/aisp/party"
    },
    "Meta": {
        "TotalPages": 1
    }
}
```

### Get offers

Banks can do offers to the user. It's a different kind of APIs compared to the one we saw earlier.
You may not need it, although it does open the door to new kind of financial services: Comparing banks offers.
This time, I kind of feel that only seeing the offer of a user may be a bit limiting. As a developer, you could imagine a comparator of offers, but not sure I see the value if the user needs to be already a customer of the bank I can only list.
In any case, you will have more imagination than me to see a usage for that API. Sure thing is, it's nice to be able to get the offers available to the current user.

By account
```
GET /open-banking/v3.1.1/aisp/accounts/12a03d84-1a34-46c6-acd1-53065e80d282/offers
```

Bulk version
```
GET /open-banking/v3.1.1/aisp/offers
```

```
{
    "Data": {
        "Offer": [
            {
                "OfferId": "b6ad0e89-efd1-40c8-a81f-1ac50c048737",
                "OfferType": "LimitIncrease",
                "Description": "Credit limit increase for the account up to £9700.00",
                "Amount": {
                    "Amount": "9700.00",
                    "Currency": "GBP"
                }
            },
            {
                "OfferId": "6789b034-bc37-4876-a591-1f316c6b588a",
                "OfferType": "BalanceTransfer",
                "Description": "Balance transfer offer up to £3000.00",
                "Amount": {
                    "Amount": "3000.00",
                    "Currency": "GBP"
                }
            }
        ]
    },
    "Links": {
        "Self": "https://matls.rs.aspsp.ob.forgerock.financial/open-banking/v3.1.1/aisp/accounts/12a03d84-1a34-46c6-acd1-53065e80d282/offers"
    },
    "Meta": {
        "TotalPages": 1
    }
}
```

### Get scheduled-payments

You can retrieve the user scheduled payments.

By account
```
GET /open-banking/v3.1.1/aisp/accounts/12a03d84-1a34-46c6-acd1-53065e80d282/scheduled-payments
```

Bulk version
```
GET /open-banking/v3.1.1/aisp/scheduled-payments
```

```
{
    "Data": {
        "ScheduledPayment": [
            {
                "ScheduledPaymentId": "2927e244-4e33-406c-85cf-6c46e2c6263e",
                "ScheduledPaymentDateTime": "2020-05-08T11:00:16.527Z",
                "ScheduledType": "Execution",
                "Reference": "Risus Corp.",
                "InstructedAmount": {
                    "Amount": "475.69",
                    "Currency": "GBP"
                },
                "CreditorAccount": {
                    "SchemeName": "UK.OBIE.SortCodeAccountNumber",
                    "Identification": "34928225323999",
                    "Name": "Wooten, Xenos T."
                }
            },
            ...
        ]
    },
    "Links": {
        "Self": "https://matls.rs.aspsp.ob.forgerock.financial/open-banking/v3.1.1/aisp/accounts/12a03d84-1a34-46c6-acd1-53065e80d282/scheduled-payments"
    },
    "Meta": {
        "TotalPages": 1
    }
}
```

### Get statements

You can get the statements of a user.

By account
```
GET /open-banking/v3.1.1/aisp/accounts/12a03d84-1a34-46c6-acd1-53065e80d282/statements
```

Bulk version
```
GET /open-banking/v3.1.1/aisp/statements
```

```
{
    "Data": {
        "Statement": [
            {
                "AccountId": "12a03d84-1a34-46c6-acd1-53065e80d282",
                "StatementId": "8a163bbc-bbc0-4de5-bb12-ee46f49814b5",
                "StatementReference": "2018-10",
                "Type": "RegularPeriodic",
                "StartDateTime": "2018-10-01T11:00:16.352Z",
                "EndDateTime": "2018-10-31T11:00:16.352Z",
                "StatementDescription": [
                    "Oct 2018"
                ],
                "StatementAmount": [
                    {
                        "CreditDebitIndicator": "Debit",
                        "Type": "PreviousClosingBalance",
                        "Amount": {
                            "Amount": "5935.24",
                            "Currency": "GBP"
                        }
                    },
                    {
                        "CreditDebitIndicator": "Debit",
                        "Type": "ClosingBalance",
                        "Amount": {
                            "Amount": "5327.98",
                            "Currency": "GBP"
                        }
                    }
                ]
            },
            ...
        ]
    },
    "Links": {
        "Self": "https://matls.rs.aspsp.ob.forgerock.financial/open-banking/v3.1.1/aisp/accounts/12a03d84-1a34-46c6-acd1-53065e80d282/statements",
        "First": "https://matls.rs.aspsp.ob.forgerock.financial/open-banking/v3.1.1/aisp/accounts/12a03d84-1a34-46c6-acd1-53065e80d282/statements?page=0",
        "Next": "https://matls.rs.aspsp.ob.forgerock.financial/open-banking/v3.1.1/aisp/accounts/12a03d84-1a34-46c6-acd1-53065e80d282/statements?page=1",
        "Last": "https://matls.rs.aspsp.ob.forgerock.financial/open-banking/v3.1.1/aisp/accounts/12a03d84-1a34-46c6-acd1-53065e80d282/statements?page=1"
    },
    "Meta": {
        "TotalPages": 2
    }
}
```

### Get statement transactions

This one is a first of its kind: You can get the transactions for a specific account *and* a specific statement. 
The same way the account was filtered via the path, the statement can also be filtered: it's a double filtering.

```
GET /open-banking/v3.1.1/aisp/accounts/12a03d84-1a34-46c6-acd1-53065e80d282/statements/8a163bbc-bbc0-4de5-bb12-ee46f49814b5/transactions
```

```
{
    "Data": {
        "Transaction": [
            {
                "AccountId": "12a03d84-1a34-46c6-acd1-53065e80d282",
                "TransactionId": "d0487833-e980-4a45-a44e-a6aa68521f01",
                "TransactionReference": "Ref 7871",
                "CreditDebitIndicator": "Debit",
                "Status": "Booked",
                "BookingDateTime": "2018-10-01T21:06:37.352Z",
                "ValueDateTime": "2018-10-01T21:09:59.352Z",
                "Amount": {
                    "Amount": "264.37",
                    "Currency": "GBP"
                },
                "BankTransactionCode": {
                    "Code": "ReceivedCreditTransfer",
                    "SubCode": "DomesticCreditTransfer"
                },
                "ProprietaryBankTransactionCode": {
                    "Code": "Transfer",
                    "Issuer": "AlphaBank"
                },
                "TransactionInformation": "Cash to Snider, Linda L.",
                "Balance": {
                    "Amount": {
                        "Amount": "4078.75",
                        "Currency": "GBP"
                    },
                    "CreditDebitIndicator": "Debit",
                    "Type": "InterimBooked"
                }
            },
            ...
        ]
    },
    "Links": {
        "Self": "https://matls.rs.aspsp.ob.forgerock.financial/open-banking/v3.1.1/aisp/accounts/12a03d84-1a34-46c6-acd1-53065e80d282/statements/8a163bbc-bbc0-4de5-bb12-ee46f49814b5/transactions",
        "First": "https://matls.rs.aspsp.ob.forgerock.financial/open-banking/v3.1.1/aisp/accounts/12a03d84-1a34-46c6-acd1-53065e80d282/statements/8a163bbc-bbc0-4de5-bb12-ee46f49814b5/transactions?page=0",
        "Next": "https://matls.rs.aspsp.ob.forgerock.financial/open-banking/v3.1.1/aisp/accounts/12a03d84-1a34-46c6-acd1-53065e80d282/statements/8a163bbc-bbc0-4de5-bb12-ee46f49814b5/transactions?page=1",
        "Last": "https://matls.rs.aspsp.ob.forgerock.financial/open-banking/v3.1.1/aisp/accounts/12a03d84-1a34-46c6-acd1-53065e80d282/statements/8a163bbc-bbc0-4de5-bb12-ee46f49814b5/transactions?page=2"
    },
    "Meta": {
        "TotalPages": 3
    }
}
```


## Payments

The payments APIs offers you to trigger payments from the user account.
There are different kinds of payments:
- Domestic
- International
- File payments

Domestic and international payments share some similarity.
In particular, they both offers single payments, schedule payment and standing orders.

### Domestic payments

A domestic payment is a payment in the same currency, as opposed to an international payment.

Domestic payment offers three kind of payments:

- single payments
- schedule payment
- standing orders


_They are pretty self explanatory. To illustrate, I will only give you the payment consent request payload for now. How works payment would be details in a dedicated article.
You will see that already the consent request is revealing a lot from the API._

#### Domestic single payment

Single payment is a way to make an instant payment to someone, from the same currency. It's the API to use if you want to offer payments between friends.

```
POST /open-banking/v3.1.1/pisp/domestic-payment-consents

{
  "Data": {
    "Initiation": {
      "InstructionIdentification": "ACME412",
      "EndToEndIdentification": "FRESCO.21302.GFX.20",
      "InstructedAmount": {
        "Amount": "165.88",
        "Currency": "GBP"
      },
      "CreditorAccount": {
        "SchemeName": "UK.OBIE.SortCodeAccountNumber",
        "Identification": "08080021325698",
        "Name": "ACME Inc",
        "SecondaryIdentification": "0002"
      },
      "RemittanceInformation": {
        "Reference": "FRESCO-101",
        "Unstructured": "Internal ops code 5120101"
      }
    }
  },
  "Risk": {
    "PaymentContextCode": "EcommerceGoods",
    "MerchantCategoryCode": "5967",
    "MerchantCustomerIdentification": "053598653254",
    "DeliveryAddress": {
      "AddressLine": [
        "Flat 7",
        "Acacia Lodge"
      ],
      "StreetName": "Acacia Avenue",
      "BuildingNumber": "27",
      "PostCode": "GU31 2ZZ",
      "TownName": "Sparsholt",
      "CountrySubDivision": [
        "Wessex"
      ],
      "Country": "UK"
    }
  }
}
```

### Domestic schedule payment

This payment allows you to offer schedule payment in the same currency.

```
POST /open-banking/v3.1.1/pisp/domestic-scheduled-payment-consents 

{
  "Data": {
    "Permission":"Create",
    "Initiation": {
      "RequestedExecutionDateTime": "2018-11-02T00:00:00+00:00",
      "InstructedAmount": {
        "Amount": "200.00",
        "Currency": "GBP"
      },
      "DebtorAccount": {
        "SchemeName": "SortCodeAccountNumber",
        "Identification": "73815621781678",
        "Name": "test_b0ee1d08-3e58-49cb-b5f2-24fae6fe399b"
      },
      "CreditorAccount": {
        "SchemeName": "UK.OBIE.SortCodeAccountNumber",
        "Identification": "08080021325698",
        "Name": "Tom Kirkman"
      },
      "RemittanceInformation": {
        "Reference": "DSR-037",
        "Unstructured": "Internal ops code 5120103"
      },
      "InstructionIdentification": "2876d876876",
      "EndToEndIdentification": "4535353535"
    }
  },
  "Risk": {
    "PaymentContextCode": "PartyToParty"
  }
}
```

### Domestic standing orders

This payments allows you to offer regular payments from the same currency.

```
POST /open-banking/v3.1.1/pisp/domestic-standing-order-consents 

{
  "Data": {
  	"Permission":"Create",
    "Initiation": {
      "Frequency": "EvryDay",
      "Reference": "Pocket money for Damien",
      "FirstPaymentDateTime": "2018-12-06T06:06:06+00:00",
      "FirstPaymentAmount": {
        "Amount": "6.66",
        "Currency": "GBP"
      },     
      "RecurringPaymentAmount": {
        "Amount": "7.00",
        "Currency": "GBP"
      },    
      "FinalPaymentDateTime": "2020-11-20T06:06:06+00:00",
      "FinalPaymentAmount": {
        "Amount": "7.00",
        "Currency": "GBP"
      },
      "DebtorAccount": {
        "SchemeName": "SortCodeAccountNumber",
        "Identification": "73815621781678",
        "Name": "test_b0ee1d08-3e58-49cb-b5f2-24fae6fe399b"
      },
      "CreditorAccount": {
        "SchemeName": "UK.OBIE.SortCodeAccountNumber",
        "Identification": "08080021325698",
        "Name": "Bob Clements"
      }
    }
  },
  "Risk": {
    "PaymentContextCode": "PartyToParty"
  }
}
```


### International payments

Like domestic payments but between two currencies.

#### International single payment

Single payment is a way to make an instant payment to someone, between two different currencies.

```
POST /open-banking/v3.1.1/pisp/international-payment-consents 

{
  "Data": {
    "Initiation": {
      "InstructionIdentification": "ACME412",
      "EndToEndIdentification": "FRESCO.21302.GFX.20",
      "InstructionPriority": "Normal",
      "CurrencyOfTransfer":"USD",
      "InstructedAmount": {
        "Amount": "165.88",
        "Currency": "GBP"
      },
      "CreditorAccount": {
        "SchemeName": "UK.OBIE.SortCodeAccountNumber",
        "Identification": "08080021325698",
        "Name": "ACME Inc",
        "SecondaryIdentification": "0002"
      },
      "RemittanceInformation": {
        "Reference": "FRESCO-101",
        "Unstructured": "Internal ops code 5120101"
      },
      "ExchangeRateInformation": {
        "UnitCurrency": "GBP",
        "RateType": "Actual"
      }
    }
  },
  "Risk": {
    "PaymentContextCode": "PartyToParty"
  }
}
```

### International schedule payment

This payment allows you to offer schedule payment between two different currencies.

```
POST /open-banking/v3.1.1/pisp/international-scheduled-payment-consents

{
  "Data": {
    "Permission":"Create",
    "Initiation": {
      "RequestedExecutionDateTime": "2018-11-02T00:00:00+00:00",
      "InstructedAmount": {
        "Amount": "200.00",
        "Currency": "GBP"
      },
      "DebtorAccount": {
        "SchemeName": "SortCodeAccountNumber",
        "Identification": "73815621781678",
        "Name": "test_b0ee1d08-3e58-49cb-b5f2-24fae6fe399b"
      },
      "CreditorAccount": {
        "SchemeName": "UK.OBIE.SortCodeAccountNumber",
        "Identification": "08080021325698",
        "Name": "Tom Kirkman"
      },
      "RemittanceInformation": {
        "Reference": "DSR-037",
        "Unstructured": "Internal ops code 5120103"
      },
      "InstructionIdentification": "2876d876876",
      "EndToEndIdentification": "4535353535"
    }
  },
  "Risk": {
    "PaymentContextCode": "PartyToParty"
  }
}
```

### International standing orders

This payments allows you to offer regular payments between two different currencies.

```
POST /open-banking/v3.1.1/pisp/international-standing-order-consents

{
  "Data": {
  	"Permission":"Create",
    "Initiation": {
      "Purpose": "Test",
      "Frequency": "EvryWorkgDay",
      "Reference": "Rent",
      "FirstPaymentDateTime": "2018-12-06T06:06:06+00:00",
      "InstructedAmount": {
        "Amount": "7.00",
        "Currency": "EUR"
      },  
      "CurrencyOfTransfer": "EUR", 
      "FinalPaymentDateTime": "2020-11-20T06:06:06+00:00",
      "DebtorAccount": {
        "SchemeName": "SortCodeAccountNumber",
        "Identification": "73815621781678",
        "Name": "test_b0ee1d08-3e58-49cb-b5f2-24fae6fe399b"
      },
      "CreditorAccount": {
        "SchemeName": "UK.OBIE.SortCodeAccountNumber",
        "Identification": "08080021325698",
        "Name": "Bob Clements"
      }
    }
  },
  "Risk": {
    "PaymentContextCode": "PartyToParty"
  }
} 
```

## File payments

Files payments are more advance APIs. They allow you to submit a bulk of payment. Those APIs accept files as input, which is quite convenient. 
The standard defines the format for two types of files, JSON and XML.

### JSON

The JSON one looks very much like the other payments payload. The only difference is that it's accepting multiple one at the same time.

```
POST /open-banking/v3.1.1/pisp/file-payment-consents/PDC_f85d8c8a-1fe5-4f7d-b8b6-1d7cc9ad6ef6/file

{
  "Data": {
    "DomesticPayments": [
      {
        "InstructionIdentification": "ANSM020",
        "EndToEndIdentification": "FRESCO.21302.GFX.01",
        "LocalInstrument": "UK.OBIE.CHAPS",
        "InstructedAmount": {
          "Amount": "21.00",
          "Currency": "GBP"
        },
        "DebtorAccount": {
          "SchemeName": "UK.OBIE.SortCodeAccountNumber",
          "Identification": "11280001234567",
          "Name": "Andrea Smith"
        },
        "CreditorAccount": {
          "SchemeName": "UK.OBIE.SortCodeAccountNumber",
          "Identification": "08080021325698",
          "Name": "Bob Clements"
        },
        "CreditorPostalAddress": {
          "AddressType": "Correspondence",
          "StreetName": "Liberty",
          "BuildingNumber": "1",
          "PostCode": "AB1 2CD",
          "TownName": "London",
          "Country": "UK"
        },
        "RemittanceInformation": {
          "Reference": "FRESCO-037",
          "Unstructured": "Internal ops code 5120103"
        }
      },
      ...
    ]
  }
} 
```

### XML

If you are an XML fan, you can send the same in XML.

## Event notification

The event notification APIs allows you to receive notification when a payment changes status for example.
It's a very convenient APIs, which help you avoiding implementing a pull mechanism against all your current payment.

The idea of this API is very simple:

1. You register your event callback to the bank. It's the callback that the bank will use to send you notification
2. When an event is happening to one of your payments, the bank calls you via the callback you registered, with a payload containing the payments that changed
3. You call the `Get payment` for each payment referred from the event notifications, so you can update the payment from your side.


### Registering your callback

Registering your callback is very straight forward:

```
POST /open-banking/v3.1.1/callback-urls 

{
  "Data": {
    "Url": "https://tpp-core.ob.forgerock.financial/open-banking/v3.0/event-notifications",
    "Version": "3.0"
  }
}
``` 

### Receiving events

The payload of the events, send by the bank to you, is a JWT.
The payload of this JWT is a JSON that look like this:

```
{
  "iss": "https://matls.as.aspsp.forgerock.financial/",
  "iat": 1516239022,
  "jti": "b460a07c-4962-43d1-85ee-9dc10fbb8f6c",
  "sub": "https://matls.as.aspsp.forgerock.financial/open-banking/v3.1.1/pisp/domestic-payments/pmt-7290-003",
  "aud": "7umx5nTR33811QyQfi",
  "events": {
    "urn:uk:org:openbanking:events:resource-update": {
      "subject": {
        "subject_type": "http://openbanking.org.uk/rid_http://openbanking.org.uk/rty",
        "http://openbanking.org.uk/rid": "PDC_9ddce5f2-52cb-4a3e-b26",
        "http://openbanking.org.uk/rty": "domestic-payment",
        "http://openbanking.org.uk/rlk": [
          {
            "version": "v3.1",
            "link": "https://matls.as.aspsp.forgerock.financial/open-banking/v3.1.1/pisp/domestic-payments/PDC_9ddce5f2-52cb-4a3e-b26"
          },
          ...
        ]
      }
    }
  },
  "txn": "dfc51628-3479-4b81-ad60-210b43d02306",
  "toe": 1516239022
}
```

### Get payment

To be complete, here is how you read the payment from the event notification

```
GET /open-banking/v3.1.1/pisp/domestic-payments/PDC_9ddce5f2-52cb-4a3e-b26
```

```
{
    "Data": {
        "DomesticPaymentId": "PDC_9ddce5f2-52cb-4a3e-b261-2d32ce4b5470",
        "ConsentId": "PDC_9ddce5f2-52cb-4a3e-b261-2d32ce4b5470",
        "CreationDateTime": "2019-10-24T11:09:47.452Z",
        "Status": "AcceptedSettlementCompleted",
        "StatusUpdateDateTime": "2019-10-24T11:09:47.419Z",
        "Initiation": {
            "InstructionIdentification": "ACME412",
            "EndToEndIdentification": "FRESCO.21302.GFX.20",
            "InstructedAmount": {
                "Amount": "165.88",
                "Currency": "GBP"
            },
            "CreditorAccount": {
                "SchemeName": "UK.OBIE.SortCodeAccountNumber",
                "Identification": "08080021325698",
                "Name": "ACME Inc",
                "SecondaryIdentification": "0002"
            },
            "RemittanceInformation": {
                "Unstructured": "Internal ops code 5120101",
                "Reference": "FRESCO-101"
            }
        }
    },
    "Links": {
        "Self": "https://matls.rs.aspsp.ob.forgerock.financial/open-banking/v3.1/pisp/domestic-payments/PDC_9ddce5f2-52cb-4a3e-b261-2d32ce4b5470"
    },
    "Meta": {}
}
```


## Confirmation of funds

Confirmation of funds is a set of APIs focusing on a simple task: telling you by a yes and no answer, if the user has enough funds.
Obviously you could say that the `Get balances` would do the job. Thing is, some developers may not have the permissions to access the account and transaction APIs.
We will talk in another article about the trusting model of Open Banking and how permissions are granted. 
As a Fintech, it is easier to get permissions for the confirmation of funds than the account and transactions APIs.
Developers only interested by know if the user has enough funds may just consider only requesting the confirmation of funds permission.

Another interesting usage of the confirmation of funds is during payments. This API has been designed as an extension of the payments APIs, and basically developers allowed to do payments can also use the confirmation of funds APIs in the middle of the payment flow.
I personally imagine that for large payments, you could make sure that the user has enough funds before even bothering trying to continue the rest of the payment flow.

### Stand alone confirmation of funds

You can verify if a user has enough funds by doing 

```
POST /open-banking/v3.1.1/cbpii/funds-confirmations

{
  "Data": {
    "ConsentId": "FCC_269c0ab6-e02f-4010-90b2-2c9c29c7e616",
    "Reference": "Purchase02",
    "InstructedAmount": {
       "Amount": "2000000000.00",
       "Currency": "GBP"
    }
  }
}
```

```
{
    "Data": {
        "FundsConfirmationId": "FCC_269c0ab6-e02f-4010-90b2-2c9c29c7e616",
        "ConsentId": "FCC_269c0ab6-e02f-4010-90b2-2c9c29c7e616",
        "CreationDateTime": "2019-10-24T12:30:12.255Z",
        "FundsAvailable": false,
        "Reference": "Purchase02",
        "InstructedAmount": {
            "Amount": "2000000000.00",
            "Currency": "GBP"
        }
    },
    "Links": {
        "Self": "https://matls.rs.aspsp.ob.forgerock.financial/open-banking/v3.0/cbpii/funds-confirmations"
    },
    "Meta": {}
}
```

It's important to note that you would need to have first the authorisation of the user before calling this API.

### confirmation of funds during a payment

When you are doing a payment flow, once you got the user authorisation of payment, you can call the confirmation of funds apis for this payment.

The API looks like this:

```
GET /open-banking/v3.1.1/pisp/domestic-payment-consents/PDC_615053fb-ae81-430c-8d30-1a50a1d9ccb0/funds-confirmation
```

```
{
    "Data": {
        "FundsAvailableResult": {
            "FundsAvailableDateTime": "2019-10-24T12:33:40.981Z",
            "FundsAvailable": true
        }
    },
    "Links": {
        "Self": "https://matls.rs.aspsp.ob.forgerock.financial/open-banking/v3.1.1/pisp/domestic-payment-consents/PDC_615053fb-ae81-430c-8d30-1a50a1d9ccb0/funds-confirmation"
    },
    "Meta": {}
}
```

As you can see, you don't need to specify the amount. As you are actually referring to the payments via `PDC_615053fb-ae81-430c-8d30-1a50a1d9ccb0`, the bank is able to deduce remember the amount you are interested in.

## Conclusion

We saw in the article the core of the APIs offered by Open Banking UK. The rest of the APIs that I voluntary haven't describe here, are designed to handle the security part. We will talk about them once we reach the security flow.

You should now have a good idea of what kind of functionalities/resources are offers by Open Banking UK, and hope it's giving you new ideas for your future projects!
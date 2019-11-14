# Get the user transactions using the Open Banking APIs
*This article is part of a set of tutorial about Open Banking UK, although, they are all written to be read as a standalone article. If you are interested in learning about Open Banking UK from the beginning, you may want to start by our intro article.*

*If you wish to execute the request at the same than the article, please note that it assumes you are already onboard. If it is not your case, please follow the article X*

In this article, we will see how to consume the transactions APIs. If you haven't follow yet the security flow article, I recommend strongly to have a read at it first. We will base this article on it and concentrate on the differences.

## Create an account consent request

In order to access the user transactions, we need to create an account request with one of those two permissions:

* `ReadTransactionsBasic`
* `ReadTransactionsDetail`

The difference is in the level of details. 

An example of a transaction received with `ReadTransactionsBasic`: 

```
{
    "AccountId": "09000792-bd62-4348-a62d-e7d75f0fca3c",
    "TransactionId": "8ecec17e-764d-4988-ad20-bfb782a9c32c",
    "TransactionReference": "Ref 5573",
    "CreditDebitIndicator": "Credit",
    "Status": "Booked",
    "BookingDateTime": "2018-11-01T12:19:24.511Z",
    "ValueDateTime": "2018-11-01T12:22:40.511Z",
    "Amount": {
        "Amount": "330.25",
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
    "TransactionInformation": ""
}
```

The same transaction but with `ReadTransactionsDetails`: 

```
{
    "AccountId": "09000792-bd62-4348-a62d-e7d75f0fca3c",
    "TransactionId": "8ecec17e-764d-4988-ad20-bfb782a9c32c",
    "TransactionReference": "Ref 5573",
    "CreditDebitIndicator": "Credit",
    "Status": "Booked",
    "BookingDateTime": "2018-11-01T12:19:24.511Z",
    "ValueDateTime": "2018-11-01T12:22:40.511Z",
    "Amount": {
        "Amount": "330.25",
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
    "TransactionInformation": "Cash from Burch, Bruno K.",
    "Balance": {
        "Amount": {
            "Amount": "2142.97",
            "Currency": "GBP"
        },
        "CreditDebitIndicator": "Debit",
        "Type": "InterimBooked"
    }
}
```


You have to specify either or both `Credits` or `Debits` transactions, by using the permissions:
* `ReadTransactionsCredits`
* `ReadTransactionsDebits`

In this article, we will show how to retrieve the transactions from a specific account and from a statement, therefore we need to ask the user the permissions for accounts and statements access:

* `ReadAccountsBasic`
* `ReadStatementsBasic`

At the end, our account consent access request will be:

```
{
  "Data": {
    "Permissions": [
      "ReadAccountsBasic",
      "ReadStatementsBasic",
      "ReadTransactionsCredits",
      "ReadTransactionsDebits",
      "ReadTransactionsDetail"
    ]
  },
  "Risk": {}
}
```

Follow the security flow and use this account consent access payload and get an access token with the user authorisation.

## Get all the transactions

Now that you've got an access token, you can start consuming the transactions APIs!
You can use the bulk API to retrieve the transactions from all the accounts at the same time.
Very simple, all you need to do is:

```
GET /open-banking/v3.1.1/aisp/transactions HTTP/1.1
Host: matls.rs.aspsp.ob.forgerock.financial
Authorization: Bearer eyJ0eXAiOiJKV1QiLCJ6aXAiOiJOT05FIiwia2lkIjoiRTE5N1kzMVFLT05mSk42aTdrQlkyMzFneUFvPSIsImFsZyI6IkVTMjU2In0.eyJzdWIiOiJ0ZXN0XzAxODQ5NDZiLWRlZjMtNGI2MC1iZjQ1LWViNmM5Y2M0ZjlkNiIsImN0cyI6Ik9BVVRIMl9HUkFOVF9TRVQiLCJhdXRoX2xldmVsIjowLCJhdWRpdFRyYWNraW5nSWQiOiIxNDQzNjRmZS1jMTkyLTQzNzYtYWM2Ny0xY2NmMjU3N2M0MGItNDc5ODYiLCJpc3MiOiJodHRwczovL2FzLmFzcHNwLm9iLmZvcmdlcm9jay5maW5hbmNpYWwvb2F1dGgyIiwidG9rZW5OYW1lIjoiYWNjZXNzX3Rva2VuIiwidG9rZW5fdHlwZSI6IkJlYXJlciIsImF1dGhHcmFudElkIjoiVGhleE9hdVRmQzc0alpqU04td0MxQUU0OVkwLkpfTk9oN0djczFQSWRxWDZxOWFtSGQ2S3ZFOCIsIm5vbmNlIjoiMTBkMjYwYmYtYTdkOS00NDRhLTkyZDktN2I3YTVmMDg4MjA4IiwiYXVkIjoiNjc4YzRjYzEtMTJlNC00YzgyLTg0ZmQtNzNjYmY2MDk1MzVhIiwibmJmIjoxNTczNTcyNzUwLCJncmFudF90eXBlIjoiYXV0aG9yaXphdGlvbl9jb2RlIiwic2NvcGUiOlsib3BlbmlkIiwiYWNjb3VudHMiXSwiYXV0aF90aW1lIjoxNTczNTcyNzQ5LCJjbGFpbXMiOiJ7XCJpZF90b2tlblwiOntcImFjclwiOntcInZhbHVlXCI6XCJ1cm46b3BlbmJhbmtpbmc6cHNkMjpzY2FcIixcImVzc2VudGlhbFwiOnRydWV9LFwib3BlbmJhbmtpbmdfaW50ZW50X2lkXCI6e1widmFsdWVcIjpcIkFBQ19kMTY1YjM4OC01ODJjLTQ5MzgtYjhhOS0wYzViNzA1ODc2YWVcIixcImVzc2VudGlhbFwiOnRydWV9fSxcInVzZXJpbmZvXCI6e1wib3BlbmJhbmtpbmdfaW50ZW50X2lkXCI6e1widmFsdWVcIjpcIkFBQ19kMTY1YjM4OC01ODJjLTQ5MzgtYjhhOS0wYzViNzA1ODc2YWVcIixcImVzc2VudGlhbFwiOnRydWV9fX0iLCJyZWFsbSI6Ii9vcGVuYmFua2luZyIsImV4cCI6MTU3MzY1OTE1MCwiaWF0IjoxNTczNTcyNzUwLCJleHBpcmVzX2luIjo4NjQwMCwianRpIjoiVGhleE9hdVRmQzc0alpqU04td0MxQUU0OVkwLjhEV1NpYnJYUnhHZkg1S2hNVmlLaEMtVFFLayJ9.teLlhDPLoobqQAcl7lGoZve1jqF1d1Gj3_ZJoZc9T6_svW5-h6dBY6TNGc_GPmBJ4t3rH4t0mWbOyIthepCovQ
Content-Type: application/json
x-idempotency-key: FRESCO.21302.GFX.20
x-fapi-financial-id: 0015800001041REAAY
x-fapi-customer-last-logged-time: Sun, 10 Sep 2017 19:43:31 UTC
x-fapi-customer-ip-address: 104.25.212.99
x-fapi-interaction-id: 93bac548-d2de-4546-b106-880a5018460d
Accept: application/json
```

As a response:
```
{
    "Data": {
        "Transaction": [
            {
                "AccountId": "09000792-bd62-4348-a62d-e7d75f0fca3c",
                "TransactionId": "8ecec17e-764d-4988-ad20-bfb782a9c32c",
                "TransactionReference": "Ref 5573",
                "CreditDebitIndicator": "Credit",
                "Status": "Booked",
                "BookingDateTime": "2018-11-01T12:19:24.511Z",
                "ValueDateTime": "2018-11-01T12:22:40.511Z",
                "Amount": {
                    "Amount": "330.25",
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
                "TransactionInformation": "Cash from Burch, Bruno K.",
                "Balance": {
                    "Amount": {
                        "Amount": "2142.97",
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
        "Self": "https://matls.rs.aspsp.ob.forgerock.financial/open-banking/v3.1.1/aisp/transactions",
        "First": "https://matls.rs.aspsp.ob.forgerock.financial/open-banking/v3.1.1/aisp/transactions?page=0",
        "Next": "https://matls.rs.aspsp.ob.forgerock.financial/open-banking/v3.1.1/aisp/transactions?page=1",
        "Last": "https://matls.rs.aspsp.ob.forgerock.financial/open-banking/v3.1.1/aisp/transactions?page=63"
    },
    "Meta": {
        "TotalPages": 64
    }
}
```


You get the list of transactions in a JSON array, paginated.
The `links` section is a helper to guide you on how to navigate between pages. You will notice that behind the scene, there is just a `page` GET parameter. 

You can also apply a date filter on top, via the two GET parameters `fromBookingDateTime` and `toBookingDateTime`.

```
GET /open-banking/v3.1.1/aisp/transactions?fromBookingDateTime=2018-11-01T12:19:24&toBookingDateTime=2019-11-01T12:19:24 
``` 

## Get transactions by account

You can filter transactions by account. Instead to be a GET parameter this time, it is a path parameter.

But first, you need to know which account you want to filter transactions from. This is the reason we added the permission `ReadAccountsBasic` so you could get the list of account IDs:

```
GET /open-banking/v3.1.1/aisp/accounts/ HTTP/1.1
Host: matls.rs.aspsp.ob.forgerock.financial
Authorization: Bearer eyJ0eXAiOiJKV1QiLCJ6aXAiOiJOT05FIiwia2lkIjoiRTE5N1kzMVFLT05mSk42aTdrQlkyMzFneUFvPSIsImFsZyI6IkVTMjU2In0.eyJzdWIiOiJ0ZXN0XzAxODQ5NDZiLWRlZjMtNGI2MC1iZjQ1LWViNmM5Y2M0ZjlkNiIsImN0cyI6Ik9BVVRIMl9HUkFOVF9TRVQiLCJhdXRoX2xldmVsIjowLCJhdWRpdFRyYWNraW5nSWQiOiIxNDQzNjRmZS1jMTkyLTQzNzYtYWM2Ny0xY2NmMjU3N2M0MGItNDc5ODYiLCJpc3MiOiJodHRwczovL2FzLmFzcHNwLm9iLmZvcmdlcm9jay5maW5hbmNpYWwvb2F1dGgyIiwidG9rZW5OYW1lIjoiYWNjZXNzX3Rva2VuIiwidG9rZW5fdHlwZSI6IkJlYXJlciIsImF1dGhHcmFudElkIjoiVGhleE9hdVRmQzc0alpqU04td0MxQUU0OVkwLkpfTk9oN0djczFQSWRxWDZxOWFtSGQ2S3ZFOCIsIm5vbmNlIjoiMTBkMjYwYmYtYTdkOS00NDRhLTkyZDktN2I3YTVmMDg4MjA4IiwiYXVkIjoiNjc4YzRjYzEtMTJlNC00YzgyLTg0ZmQtNzNjYmY2MDk1MzVhIiwibmJmIjoxNTczNTcyNzUwLCJncmFudF90eXBlIjoiYXV0aG9yaXphdGlvbl9jb2RlIiwic2NvcGUiOlsib3BlbmlkIiwiYWNjb3VudHMiXSwiYXV0aF90aW1lIjoxNTczNTcyNzQ5LCJjbGFpbXMiOiJ7XCJpZF90b2tlblwiOntcImFjclwiOntcInZhbHVlXCI6XCJ1cm46b3BlbmJhbmtpbmc6cHNkMjpzY2FcIixcImVzc2VudGlhbFwiOnRydWV9LFwib3BlbmJhbmtpbmdfaW50ZW50X2lkXCI6e1widmFsdWVcIjpcIkFBQ19kMTY1YjM4OC01ODJjLTQ5MzgtYjhhOS0wYzViNzA1ODc2YWVcIixcImVzc2VudGlhbFwiOnRydWV9fSxcInVzZXJpbmZvXCI6e1wib3BlbmJhbmtpbmdfaW50ZW50X2lkXCI6e1widmFsdWVcIjpcIkFBQ19kMTY1YjM4OC01ODJjLTQ5MzgtYjhhOS0wYzViNzA1ODc2YWVcIixcImVzc2VudGlhbFwiOnRydWV9fX0iLCJyZWFsbSI6Ii9vcGVuYmFua2luZyIsImV4cCI6MTU3MzY1OTE1MCwiaWF0IjoxNTczNTcyNzUwLCJleHBpcmVzX2luIjo4NjQwMCwianRpIjoiVGhleE9hdVRmQzc0alpqU04td0MxQUU0OVkwLjhEV1NpYnJYUnhHZkg1S2hNVmlLaEMtVFFLayJ9.teLlhDPLoobqQAcl7lGoZve1jqF1d1Gj3_ZJoZc9T6_svW5-h6dBY6TNGc_GPmBJ4t3rH4t0mWbOyIthepCovQ
Content-Type: application/json
x-idempotency-key: FRESCO.21302.GFX.20
x-fapi-financial-id: 0015800001041REAAY
x-fapi-customer-last-logged-time: Sun, 10 Sep 2017 19:43:31 UTC
x-fapi-customer-ip-address: 104.25.212.99
x-fapi-interaction-id: 93bac548-d2de-4546-b106-880a5018460d
Accept: application/json
```

As a response:
```
{
    "Data": {
        "Account": [
            {
                "AccountId": "09000792-bd62-4348-a62d-e7d75f0fca3c",
                "Currency": "GBP",
                "AccountType": "Personal",
                "AccountSubType": "CurrentAccount",
                "Nickname": "Household"
            },
            {
                "AccountId": "83d0b5bc-a834-4963-bb7a-ac65e52b3f8e",
                "Currency": "EUR",
                "AccountType": "Personal",
                "AccountSubType": "CurrentAccount",
                "Nickname": "FR Bills"
            },
            {
                "AccountId": "d9f75395-3038-4862-b0df-4dee456d23dd",
                "Currency": "GBP",
                "AccountType": "Personal",
                "AccountSubType": "CurrentAccount",
                "Nickname": "UK Bills"
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


You can now filter the transactions for the account `09000792-bd62-4348-a62d-e7d75f0fca3c`:

```
GET /open-banking/v3.1.1/aisp/accounts/09000792-bd62-4348-a62d-e7d75f0fca3c/transactions HTTP/1.1
Host: matls.rs.aspsp.ob.forgerock.financial
Authorization: Bearer eyJ0eXAiOiJKV1QiLCJ6aXAiOiJOT05FIiwia2lkIjoiRTE5N1kzMVFLT05mSk42aTdrQlkyMzFneUFvPSIsImFsZyI6IkVTMjU2In0.eyJzdWIiOiJ0ZXN0XzAxODQ5NDZiLWRlZjMtNGI2MC1iZjQ1LWViNmM5Y2M0ZjlkNiIsImN0cyI6Ik9BVVRIMl9HUkFOVF9TRVQiLCJhdXRoX2xldmVsIjowLCJhdWRpdFRyYWNraW5nSWQiOiIxNDQzNjRmZS1jMTkyLTQzNzYtYWM2Ny0xY2NmMjU3N2M0MGItNDc5ODYiLCJpc3MiOiJodHRwczovL2FzLmFzcHNwLm9iLmZvcmdlcm9jay5maW5hbmNpYWwvb2F1dGgyIiwidG9rZW5OYW1lIjoiYWNjZXNzX3Rva2VuIiwidG9rZW5fdHlwZSI6IkJlYXJlciIsImF1dGhHcmFudElkIjoiVGhleE9hdVRmQzc0alpqU04td0MxQUU0OVkwLkpfTk9oN0djczFQSWRxWDZxOWFtSGQ2S3ZFOCIsIm5vbmNlIjoiMTBkMjYwYmYtYTdkOS00NDRhLTkyZDktN2I3YTVmMDg4MjA4IiwiYXVkIjoiNjc4YzRjYzEtMTJlNC00YzgyLTg0ZmQtNzNjYmY2MDk1MzVhIiwibmJmIjoxNTczNTcyNzUwLCJncmFudF90eXBlIjoiYXV0aG9yaXphdGlvbl9jb2RlIiwic2NvcGUiOlsib3BlbmlkIiwiYWNjb3VudHMiXSwiYXV0aF90aW1lIjoxNTczNTcyNzQ5LCJjbGFpbXMiOiJ7XCJpZF90b2tlblwiOntcImFjclwiOntcInZhbHVlXCI6XCJ1cm46b3BlbmJhbmtpbmc6cHNkMjpzY2FcIixcImVzc2VudGlhbFwiOnRydWV9LFwib3BlbmJhbmtpbmdfaW50ZW50X2lkXCI6e1widmFsdWVcIjpcIkFBQ19kMTY1YjM4OC01ODJjLTQ5MzgtYjhhOS0wYzViNzA1ODc2YWVcIixcImVzc2VudGlhbFwiOnRydWV9fSxcInVzZXJpbmZvXCI6e1wib3BlbmJhbmtpbmdfaW50ZW50X2lkXCI6e1widmFsdWVcIjpcIkFBQ19kMTY1YjM4OC01ODJjLTQ5MzgtYjhhOS0wYzViNzA1ODc2YWVcIixcImVzc2VudGlhbFwiOnRydWV9fX0iLCJyZWFsbSI6Ii9vcGVuYmFua2luZyIsImV4cCI6MTU3MzY1OTE1MCwiaWF0IjoxNTczNTcyNzUwLCJleHBpcmVzX2luIjo4NjQwMCwianRpIjoiVGhleE9hdVRmQzc0alpqU04td0MxQUU0OVkwLjhEV1NpYnJYUnhHZkg1S2hNVmlLaEMtVFFLayJ9.teLlhDPLoobqQAcl7lGoZve1jqF1d1Gj3_ZJoZc9T6_svW5-h6dBY6TNGc_GPmBJ4t3rH4t0mWbOyIthepCovQ
Content-Type: application/json
x-idempotency-key: FRESCO.21302.GFX.20
x-fapi-financial-id: 0015800001041REAAY
x-fapi-customer-last-logged-time: Sun, 10 Sep 2017 19:43:31 UTC
x-fapi-customer-ip-address: 104.25.212.99
x-fapi-interaction-id: 93bac548-d2de-4546-b106-880a5018460d
Accept: application/json
```

*The response is exactly in the same format that the bulk and for avoiding you to scroll too much, I won't print it.*

## Filter transactions by statement

You can apply a second filter, for the statement, on top of the account filter. It is also a path parameter.
You will need to get first the list of statements for the account filtered:

```
GET /open-banking/v3.1.1/aisp/accounts/09000792-bd62-4348-a62d-e7d75f0fca3c/statements HTTP/1.1
Host: matls.rs.aspsp.ob.forgerock.financial
Authorization: Bearer eyJ0eXAiOiJKV1QiLCJ6aXAiOiJOT05FIiwia2lkIjoiRTE5N1kzMVFLT05mSk42aTdrQlkyMzFneUFvPSIsImFsZyI6IkVTMjU2In0.eyJzdWIiOiJ0ZXN0XzAxODQ5NDZiLWRlZjMtNGI2MC1iZjQ1LWViNmM5Y2M0ZjlkNiIsImN0cyI6Ik9BVVRIMl9HUkFOVF9TRVQiLCJhdXRoX2xldmVsIjowLCJhdWRpdFRyYWNraW5nSWQiOiIxNDQzNjRmZS1jMTkyLTQzNzYtYWM2Ny0xY2NmMjU3N2M0MGItNDc5ODYiLCJpc3MiOiJodHRwczovL2FzLmFzcHNwLm9iLmZvcmdlcm9jay5maW5hbmNpYWwvb2F1dGgyIiwidG9rZW5OYW1lIjoiYWNjZXNzX3Rva2VuIiwidG9rZW5fdHlwZSI6IkJlYXJlciIsImF1dGhHcmFudElkIjoiVGhleE9hdVRmQzc0alpqU04td0MxQUU0OVkwLkpfTk9oN0djczFQSWRxWDZxOWFtSGQ2S3ZFOCIsIm5vbmNlIjoiMTBkMjYwYmYtYTdkOS00NDRhLTkyZDktN2I3YTVmMDg4MjA4IiwiYXVkIjoiNjc4YzRjYzEtMTJlNC00YzgyLTg0ZmQtNzNjYmY2MDk1MzVhIiwibmJmIjoxNTczNTcyNzUwLCJncmFudF90eXBlIjoiYXV0aG9yaXphdGlvbl9jb2RlIiwic2NvcGUiOlsib3BlbmlkIiwiYWNjb3VudHMiXSwiYXV0aF90aW1lIjoxNTczNTcyNzQ5LCJjbGFpbXMiOiJ7XCJpZF90b2tlblwiOntcImFjclwiOntcInZhbHVlXCI6XCJ1cm46b3BlbmJhbmtpbmc6cHNkMjpzY2FcIixcImVzc2VudGlhbFwiOnRydWV9LFwib3BlbmJhbmtpbmdfaW50ZW50X2lkXCI6e1widmFsdWVcIjpcIkFBQ19kMTY1YjM4OC01ODJjLTQ5MzgtYjhhOS0wYzViNzA1ODc2YWVcIixcImVzc2VudGlhbFwiOnRydWV9fSxcInVzZXJpbmZvXCI6e1wib3BlbmJhbmtpbmdfaW50ZW50X2lkXCI6e1widmFsdWVcIjpcIkFBQ19kMTY1YjM4OC01ODJjLTQ5MzgtYjhhOS0wYzViNzA1ODc2YWVcIixcImVzc2VudGlhbFwiOnRydWV9fX0iLCJyZWFsbSI6Ii9vcGVuYmFua2luZyIsImV4cCI6MTU3MzY1OTE1MCwiaWF0IjoxNTczNTcyNzUwLCJleHBpcmVzX2luIjo4NjQwMCwianRpIjoiVGhleE9hdVRmQzc0alpqU04td0MxQUU0OVkwLjhEV1NpYnJYUnhHZkg1S2hNVmlLaEMtVFFLayJ9.teLlhDPLoobqQAcl7lGoZve1jqF1d1Gj3_ZJoZc9T6_svW5-h6dBY6TNGc_GPmBJ4t3rH4t0mWbOyIthepCovQ
Content-Type: application/json
x-idempotency-key: FRESCO.21302.GFX.20
x-fapi-financial-id: 0015800001041REAAY
x-fapi-customer-last-logged-time: Sun, 10 Sep 2017 19:43:31 UTC
x-fapi-customer-ip-address: 104.25.212.99
x-fapi-interaction-id: 93bac548-d2de-4546-b106-880a5018460d
Accept: application/json
```
As a response:
```
{
    "Data": {
        "Statement": [
            {
                "AccountId": "09000792-bd62-4348-a62d-e7d75f0fca3c",
                "StatementId": "1901e1d6-755e-41f9-a6e5-1154540b6771",
                "StatementReference": "2018-11",
                "Type": "RegularPeriodic",
                "StartDateTime": "2018-11-01T08:16:04.511Z",
                "EndDateTime": "2018-11-30T08:16:04.511Z",
                "StatementDescription": [
                    "Nov 2018"
                ]
            },
            ...
        ]
    },
    "Links": {
        "Self": "https://matls.rs.aspsp.ob.forgerock.financial/open-banking/v3.1.1/aisp/accounts/09000792-bd62-4348-a62d-e7d75f0fca3c/statements",
        "First": "https://matls.rs.aspsp.ob.forgerock.financial/open-banking/v3.1.1/aisp/accounts/09000792-bd62-4348-a62d-e7d75f0fca3c/statements?page=0",
        "Next": "https://matls.rs.aspsp.ob.forgerock.financial/open-banking/v3.1.1/aisp/accounts/09000792-bd62-4348-a62d-e7d75f0fca3c/statements?page=1",
        "Last": "https://matls.rs.aspsp.ob.forgerock.financial/open-banking/v3.1.1/aisp/accounts/09000792-bd62-4348-a62d-e7d75f0fca3c/statements?page=1"
    },
    "Meta": {
        "TotalPages": 2
    }
}
```

You can now filter the transactions for the statement `1901e1d6-755e-41f9-a6e5-1154540b6771`:

```
GET /open-banking/v3.1.1/aisp/accounts/09000792-bd62-4348-a62d-e7d75f0fca3c/statements/1901e1d6-755e-41f9-a6e5-1154540b6771/transactions HTTP/1.1
Host: matls.rs.aspsp.ob.forgerock.financial
Authorization: Bearer eyJ0eXAiOiJKV1QiLCJ6aXAiOiJOT05FIiwia2lkIjoiRTE5N1kzMVFLT05mSk42aTdrQlkyMzFneUFvPSIsImFsZyI6IkVTMjU2In0.eyJzdWIiOiJ0ZXN0XzAxODQ5NDZiLWRlZjMtNGI2MC1iZjQ1LWViNmM5Y2M0ZjlkNiIsImN0cyI6Ik9BVVRIMl9HUkFOVF9TRVQiLCJhdXRoX2xldmVsIjowLCJhdWRpdFRyYWNraW5nSWQiOiIxNDQzNjRmZS1jMTkyLTQzNzYtYWM2Ny0xY2NmMjU3N2M0MGItNDc5ODYiLCJpc3MiOiJodHRwczovL2FzLmFzcHNwLm9iLmZvcmdlcm9jay5maW5hbmNpYWwvb2F1dGgyIiwidG9rZW5OYW1lIjoiYWNjZXNzX3Rva2VuIiwidG9rZW5fdHlwZSI6IkJlYXJlciIsImF1dGhHcmFudElkIjoiVGhleE9hdVRmQzc0alpqU04td0MxQUU0OVkwLkpfTk9oN0djczFQSWRxWDZxOWFtSGQ2S3ZFOCIsIm5vbmNlIjoiMTBkMjYwYmYtYTdkOS00NDRhLTkyZDktN2I3YTVmMDg4MjA4IiwiYXVkIjoiNjc4YzRjYzEtMTJlNC00YzgyLTg0ZmQtNzNjYmY2MDk1MzVhIiwibmJmIjoxNTczNTcyNzUwLCJncmFudF90eXBlIjoiYXV0aG9yaXphdGlvbl9jb2RlIiwic2NvcGUiOlsib3BlbmlkIiwiYWNjb3VudHMiXSwiYXV0aF90aW1lIjoxNTczNTcyNzQ5LCJjbGFpbXMiOiJ7XCJpZF90b2tlblwiOntcImFjclwiOntcInZhbHVlXCI6XCJ1cm46b3BlbmJhbmtpbmc6cHNkMjpzY2FcIixcImVzc2VudGlhbFwiOnRydWV9LFwib3BlbmJhbmtpbmdfaW50ZW50X2lkXCI6e1widmFsdWVcIjpcIkFBQ19kMTY1YjM4OC01ODJjLTQ5MzgtYjhhOS0wYzViNzA1ODc2YWVcIixcImVzc2VudGlhbFwiOnRydWV9fSxcInVzZXJpbmZvXCI6e1wib3BlbmJhbmtpbmdfaW50ZW50X2lkXCI6e1widmFsdWVcIjpcIkFBQ19kMTY1YjM4OC01ODJjLTQ5MzgtYjhhOS0wYzViNzA1ODc2YWVcIixcImVzc2VudGlhbFwiOnRydWV9fX0iLCJyZWFsbSI6Ii9vcGVuYmFua2luZyIsImV4cCI6MTU3MzY1OTE1MCwiaWF0IjoxNTczNTcyNzUwLCJleHBpcmVzX2luIjo4NjQwMCwianRpIjoiVGhleE9hdVRmQzc0alpqU04td0MxQUU0OVkwLjhEV1NpYnJYUnhHZkg1S2hNVmlLaEMtVFFLayJ9.teLlhDPLoobqQAcl7lGoZve1jqF1d1Gj3_ZJoZc9T6_svW5-h6dBY6TNGc_GPmBJ4t3rH4t0mWbOyIthepCovQ
Content-Type: application/json
x-idempotency-key: FRESCO.21302.GFX.20
x-fapi-financial-id: 0015800001041REAAY
x-fapi-customer-last-logged-time: Sun, 10 Sep 2017 19:43:31 UTC
x-fapi-customer-ip-address: 104.25.212.99
x-fapi-interaction-id: 93bac548-d2de-4546-b106-880a5018460d
Accept: application/json
```

*The response is also similar than the bulk so I won't print it here.*


## Conclusion

Consuming the transactions APIs is straight forwards once you know how to get an access token with the user authorisation.
The APIs usage and data model are very comprehensive and shouldn't be a problem for you.
In our next article, we will see how to execute a payment with Open Banking UK.

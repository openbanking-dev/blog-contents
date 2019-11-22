# Making Open Banking UK Payments

*This article is part of a set of tutorial about Open Banking UK, although, they are all written to be read as a standalone article. If you are interested in learning about Open Banking UK from the beginning, you may want to start by our intro article.*

*If you wish to execute the request at the same than the article, please note that it assumes you are already onboard. If it is not your case, please follow the article X*

In this article, we will see how to consume the payments APIs. If you haven't follow yet the security flow article, I recommend strongly to have a read at it first. We will base this article on it and concentrate on the differences.

## Types Of Payments

### Domestic Payment
Domestic payments are payments made in the countries home currency. This will be GBP for the examples we will be doing. There are 3 types of domestic payments that can be made; Single Payment, Scheduled Payment and Standing order. We will go into details of each later.

### International Payment
International payments are designed for making foreign payments. You can define the `CurrencyOfTransfer` being the currency you will what to exchange to. Again there are 3 types of domestic payments that can be made; Single Payment, Scheduled Payment and Standing order.

### File Payment
File payments allow you to make many payments from one users account. The contents of these file types can either be in [XML or JSON](https://openbanking.atlassian.net/wiki/spaces/DZ/pages/645367011/Payment+Initiation+API+Specification+-+v3.0#PaymentInitiationAPISpecification-v3.0-FileType) format and must have a SHA256 `FileHash` to act as a checksum. It is not required to know the account the payment will come from ahead of time.

## Ways Making Payments
Given the payment types we've already seen there are different ways of making those payments.

### Single Payment
A way of a user giving consent to make a payment only once. It is not required to know the account the payment will come from ahead of time.

### Scheduled Payment
A way of a user giving consent to make a payment at a future time. The users account will be debited at the time defined in `RequestedExecutionDateTime`. It is not required to know the account the payment will come from ahead of time.

### Standing Order Payment
A recurring payment made by the user. The payment is made up with a first payment followed by recurring payments which will reoccur at a defined [frequency](https://openbanking.atlassian.net/wiki/spaces/DZ/pages/641795981/Domestic+Standing+Orders+v3.0#DomesticStandingOrdersv3.0-FrequencyExamples) for a number of times or until a given date. The recurring amount can differ from the first payment amount. There can also be a final payment with a different amount. It is not required to know the account the payment will come from ahead of time.

## Making a payment via Open Banking

So before describing how we go about making a payment, let's see an example. In this example we'll use Alice who wants to buy a raspberry pi from a shop.

1. Alice is shopping on a tech website and decides she wants to buy a raspberry pi, so she adds it to her basket and checks out.
2. The shop creates initiates the payment by creating a payment request or consent in other words.
3. After creating the consent, the shop redirects Alice to her bank.
  ![Consent](https://i.imgur.com/K3iWGtr.png)
4. Alice will login to her bank, review payment and approve it.
5. Alice is redirected back to the shop with an exchange code
6. The shop uses the exchange code to get an access token
7. The shop will submit the payment
  
  ![Submit Payment](https://i.imgur.com/7mEaouy.png)
  
### Pre-requistes
To become a payment initiation services (PISPs) which is required to be able to make payments you need to have some capital behind you which might not be realistic for a developer just starting off with a cool idea. No need to worry as we can use the [Forgerock Reference Implementation Sandbox](https://openbanking4.dev/open-banking-uk-tutorials-menu/) to get going.

#### On-boarding
As a developer you'll need to onboard. You can follow the [on-boarding guide](https://openbanking4.dev/ob-uk-on-boarding/) to do this

### Request For Payment
In order for Alice to make a payment we'll need to make a payment request to the bank also known as creating the consent. There are two stages to this:

1. Getting an access token
2. Creating the payment request

#### Getting an access token
To get an access token we'll need to authenticate. For this we'll use MATLS using our transport certificates. You can find more information in [security flow](https://openbanking4.dev/ob-uk-security-flow/)

https://postman.ob.forgerock.financial/?version=latest#2dcc2464-71b0-4c4b-bd76-690a1efcc3d0

#### Creating the payment request
Now we've received an access token we'll be able to create the payment request. To do this we will use our access token as a bearer token in the `Authorization` HTTP header 
of a HTTP post to https://rs.aspsp.ob.forgerock.financial/open-banking/v3.1.1/pisp/domestic-payment-consents.

Request
```
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

Response
```
{
    "Data": {
        "ConsentId": "PDC_18d835cc-fbb6-4345-81f9-c9fe54c78594",
        "CreationDateTime": "2019-11-22T11:00:52.106Z",
        "Status": "AwaitingAuthorisation",
        "StatusUpdateDateTime": "2019-11-22T11:00:52.105Z",
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
    },
    "Links": {
        "Self": "https://matls.rs.aspsp.ob.forgerock.financial/open-banking/v3.1/pisp/domestic-payment-consents/PDC_18d835cc-fbb6-4345-81f9-c9fe54c78594"
    },
    "Meta": {}
}
```

You will notice the response is the request with some extra data. Most importantly the `ConsentId` which is the unique identifier for this payment and the `Status` which lets us know the payment has not been submitted yet.

https://postman.ob.forgerock.financial/?version=latest#afbd5831-459a-45e8-b7fa-3a2ab99e6559

### User consent
The payment request has been created and now it's time for Alice to review and approve the payment. During this next step we'll be simulating the developer redirecting Alice to her bank to approve the payment.

There will be 2 steps to this. 

1. Creating the [request parameter](https://openbanking.atlassian.net/wiki/spaces/DZ/pages/7046134/Open+Banking+Security+Profile+-+Implementer+s+Draft+v1.1.0#OpenBankingSecurityProfile-Implementer'sDraftv1.1.0-HybridGrantParameters) which contains the consent ID. 

2. Approve the payment as Alice

#### Creating the request parameter
The request parameters will be be a signed JWT which you would sign with your signing private key but we'll use the utility API and it will be signed on your behalf. You can read more about this in the [security flow](https://openbanking4.dev/ob-uk-security-flow/)

The request parameters would look like when it's not a signed JWT

```
{
  "aud": "https://as.aspsp.ob.forgerock.financial/oauth2",
  "scope": "openid accounts payments",
  "iss": "9ab94a1b-dc81-4212-9294-94035f8f84a7",
  "claims": {
    "id_token": {
      "acr": {
        "value": "urn:openbanking:psd2:sca",
        "essential": true
      },
      "openbanking_intent_id": {
        "value": "PDC_18d835cc-fbb6-4345-81f9-c9fe54c78594",
        "essential": true
      }
    },
    "userinfo": {
      "openbanking_intent_id": {
        "value": "PDC_18d835cc-fbb6-4345-81f9-c9fe54c78594",
        "essential": true
      }
    }
  },
  "response_type": "code id_token",
  "redirect_uri": "https://www.google.com",
  "state": "10d260bf-a7d9-444a-92d9-7b7a5f088208",
  "exp": 1572357975.752,
  "nonce": "10d260bf-a7d9-444a-92d9-7b7a5f088208",
  "client_id": "9ab94a1b-dc81-4212-9294-94035f8f84a7"
}
```

When the request parameter has been signed it will look like this

```
eyJraWQiOiIwMDAzOTE1YWIyYmI3OTc5ODE0YWNhOWM1NzY2MDc3NWEyNDA0NmU0IiwiYWxnIjoiUFMyNTYifQ.eyJhdWQiOiJodHRwczpcL1wvYXMuYXNwc3Aub2IuZm9yZ2Vyb2NrLmZpbmFuY2lhbFwvb2F1dGgyIiwic2NvcGUiOiJvcGVuaWQgYWNjb3VudHMgcGF5bWVudHMiLCJpc3MiOiI0MGZmZjJiZC1jNTFjLTQ1MTgtYjNjYy0zODJkY2RmMDQ4ODgiLCJjbGFpbXMiOnsiaWRfdG9rZW4iOnsiYWNyIjp7InZhbHVlIjoidXJuOm9wZW5iYW5raW5nOnBzZDI6c2NhIiwiZXNzZW50aWFsIjp0cnVlfSwib3BlbmJhbmtpbmdfaW50ZW50X2lkIjp7InZhbHVlIjoiUERDXzE4ZDgzNWNjLWZiYjYtNDM0NS04MWY5LWM5ZmU1NGM3ODU5NCIsImVzc2VudGlhbCI6dHJ1ZX19LCJ1c2VyaW5mbyI6eyJvcGVuYmFua2luZ19pbnRlbnRfaWQiOnsidmFsdWUiOiJQRENfMThkODM1Y2MtZmJiNi00MzQ1LTgxZjktYzlmZTU0Yzc4NTk0IiwiZXNzZW50aWFsIjp0cnVlfX19LCJyZXNwb25zZV90eXBlIjoiY29kZSBpZF90b2tlbiIsInJlZGlyZWN0X3VyaSI6Imh0dHBzOlwvXC93d3cuZ29vZ2xlLmNvbSIsInN0YXRlIjoiMTBkMjYwYmYtYTdkOS00NDRhLTkyZDktN2I3YTVmMDg4MjA4IiwiZXhwIjoxNTc0NDIwODI4LCJub25jZSI6IjEwZDI2MGJmLWE3ZDktNDQ0YS05MmQ5LTdiN2E1ZjA4ODIwOCIsImlhdCI6MTU3NDQyMDUyOCwiY2xpZW50X2lkIjoiNDBmZmYyYmQtYzUxYy00NTE4LWIzY2MtMzgyZGNkZjA0ODg4IiwianRpIjoiN2E2Yjc4ZDYtMmZlYy00MGY5LThkYzktNDIzNDYzMzIwNGYzIn0.D00i5c1ydP4mB-zM9r8dF93Zvdxr9Q5L5JIkOM3BRlhONxELRRf8IBzxLuRVG4gmdHoWPZhmKhiIf6-EfoyPpOTtBmiSvXb9wIsaaYbXSLNfxhyp-tq1gHgERjpY0KopbLtgtIKAnCjxRVO0clDNTZAKyc5rHDHAhljBlxiZOk4rixGPbvhH7kd8N28TkIme27QSNsCzGvT4bphjN6MWHT5Gl67W9cxyATq4noOPFPdhzOerA3lsLRb2fw8apnTD2pPSnhiV3B0x-14SkWTQecE_xq-_VY5u-ZATBYGlxH9n9jo85ozuZPXEnMnKHDMq8pM2ilSoHwi8R_qcYAcCaw
```

https://postman.ob.forgerock.financial/?version=latest#93b4ff1a-6c78-479e-bd2e-f89dbd890f4d

#### Approve the payment as Alice
Here we'll be redirecting Alice from the developers shop website to the bank to approve the payment request.

Here's an example of the redirect URL Alice will need to be redirected to. Some of the query parameters will be different in your case.

```
https://as.aspsp.ob.forgerock.financial/oauth2/authorize?response_type=code%20id_token&client_id=40fff2bd-c51c-4518-b3cc-382dcdf04888&state=10d260bf-a7d9-444a-92d9-7b7a5f088208&nonce=10d260bf-a7d9-444a-92d9-7b7a5f088208&scope=openid%20payments%20accounts&redirect_uri=https://www.google.com&request=eyJraWQiOiIwMDAzOTE1YWIyYmI3OTc5ODE0YWNhOWM1NzY2MDc3NWEyNDA0NmU0IiwiYWxnIjoiUFMyNTYifQ.eyJhdWQiOiJodHRwczpcL1wvYXMuYXNwc3Aub2IuZm9yZ2Vyb2NrLmZpbmFuY2lhbFwvb2F1dGgyIiwic2NvcGUiOiJvcGVuaWQgYWNjb3VudHMgcGF5bWVudHMiLCJpc3MiOiI0MGZmZjJiZC1jNTFjLTQ1MTgtYjNjYy0zODJkY2RmMDQ4ODgiLCJjbGFpbXMiOnsiaWRfdG9rZW4iOnsiYWNyIjp7InZhbHVlIjoidXJuOm9wZW5iYW5raW5nOnBzZDI6c2NhIiwiZXNzZW50aWFsIjp0cnVlfSwib3BlbmJhbmtpbmdfaW50ZW50X2lkIjp7InZhbHVlIjoiUERDXzE4ZDgzNWNjLWZiYjYtNDM0NS04MWY5LWM5ZmU1NGM3ODU5NCIsImVzc2VudGlhbCI6dHJ1ZX19LCJ1c2VyaW5mbyI6eyJvcGVuYmFua2luZ19pbnRlbnRfaWQiOnsidmFsdWUiOiJQRENfMThkODM1Y2MtZmJiNi00MzQ1LTgxZjktYzlmZTU0Yzc4NTk0IiwiZXNzZW50aWFsIjp0cnVlfX19LCJyZXNwb25zZV90eXBlIjoiY29kZSBpZF90b2tlbiIsInJlZGlyZWN0X3VyaSI6Imh0dHBzOlwvXC93d3cuZ29vZ2xlLmNvbSIsInN0YXRlIjoiMTBkMjYwYmYtYTdkOS00NDRhLTkyZDktN2I3YTVmMDg4MjA4IiwiZXhwIjoxNTc0NDIwODI4LCJub25jZSI6IjEwZDI2MGJmLWE3ZDktNDQ0YS05MmQ5LTdiN2E1ZjA4ODIwOCIsImlhdCI6MTU3NDQyMDUyOCwiY2xpZW50X2lkIjoiNDBmZmYyYmQtYzUxYy00NTE4LWIzY2MtMzgyZGNkZjA0ODg4IiwianRpIjoiN2E2Yjc4ZDYtMmZlYy00MGY5LThkYzktNDIzNDYzMzIwNGYzIn0.D00i5c1ydP4mB-zM9r8dF93Zvdxr9Q5L5JIkOM3BRlhONxELRRf8IBzxLuRVG4gmdHoWPZhmKhiIf6-EfoyPpOTtBmiSvXb9wIsaaYbXSLNfxhyp-tq1gHgERjpY0KopbLtgtIKAnCjxRVO0clDNTZAKyc5rHDHAhljBlxiZOk4rixGPbvhH7kd8N28TkIme27QSNsCzGvT4bphjN6MWHT5Gl67W9cxyATq4noOPFPdhzOerA3lsLRb2fw8apnTD2pPSnhiV3B0x-14SkWTQecE_xq-_VY5u-ZATBYGlxH9n9jo85ozuZPXEnMnKHDMq8pM2ilSoHwi8R_qcYAcCaw
```

You'll be able to review and approve the payment and be redirected to the developers shop (https://www.google.com for this example). In the URL you'll see and exchange code in the form `code=<exchange code>`, copy it and put it somewhere for use later.

https://postman.ob.forgerock.financial/?version=latest#a1537aad-f5dc-4bb1-9a8a-e2323b48e59b

### Submit Payment
Alice has approved the payment for her raspberry pi but we still haven't submitted the payment. The payment has still only be requested at this point. The next part of this will be submitting the payment to the bank. We'll walk through 4 different steps here of which 2 are optional but would be good practice in developing a payment system.

1. Getting an access token
2. Confirming Alice has the money to make the payment (Optional)
3. Submitting the payment
4. Checking the payment has been processed

#### Getting an access token
This time we'll need to get an access token for Alice's payment. We'll use that exchange code we got earlier in the [Approve the payment as Alice](Approve-the-payment-as-Alice) step. Follow the steps in [security flow](https://openbanking4.dev/ob-uk-security-flow/) to exchange your code for an access token.

https://postman.ob.forgerock.financial/?version=latest#602bfd06-d86f-49bf-aa00-bffff4c4d8d4

#### Confirming Alice has the money to make the payment
Before submitting the payment, we may want to check Alice has the money to be able to make this payment. This can be an important step as Alice could have been spending before we've submitted the payment and she may have no money left.

To do this you can make a HTTP get request to https://rs.aspsp.ob.forgerock.financial/open-banking/v3.1.1/pisp/domestic-payment-consents/PDC_18d835cc-fbb6-4345-81f9-c9fe54c78594/funds-confirmation with your consent ID and our access token as a bearer token in the `Authorization` HTTP header.

The response will tell you the funds are available or not.

```
{
    "Data": {
        "FundsAvailableResult": {
            "FundsAvailableDateTime": "2019-11-22T11:03:26.427Z",
            "FundsAvailable": true
        }
    },
    "Links": {
        "Self": "https://rs.aspsp.ob.forgerock.financial/open-banking/v3.1.1/pisp/domestic-payment-consents/PDC_18d835cc-fbb6-4345-81f9-c9fe54c78594"
    },
    "Meta": {}
}
```

https://postman.ob.forgerock.financial/?version=latest#9738a801-b2e9-4f3a-a837-c8432df2a6a5

#### Submitting the payment
Given funds are available you'll be able to successfully submit the payment.

We'll need to make a HTTP post to https://matls.rs.aspsp.ob.forgerock.financial/open-banking/v3.1/pisp/domestic-payments with our access token as a bearer token in the `Authorization` HTTP header now we're authorised to submit Alice's payment. We'll need to send the same information as we did for the payment request but this time with the `ConsentId`. This is so the bank can do some validation.

```
{
    "ConsentId": "PDC_18d835cc-fbb6-4345-81f9-c9fe54c78594",
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
      "CountySubDivision": [
        "Wessex"
      ],
      "Country": "UK"
    }
  }
}
```

https://postman.ob.forgerock.financial/?version=latest#98330827-0edf-49c4-b1b3-20455da95e0d

#### Checking the payment has been processed
Given the payment may be processed asynchronously, as the shop, you may want to check the payment has been processed before dispatching the goods. There is an API to get the payment and we can inspect the `Status` field we mentioned when creating the [payment request](Creating-the-payment-request).

Make a HTTP get to https://matls.rs.aspsp.ob.forgerock.financial/open-banking/v3.1/pisp/domestic-payments/PDC_8de73abd-3e3c-4b60-90eb-20aef8d85a43 with our access token as a bearer token in the `Authorization` HTTP header and you'll see whether the payment has been processed or not.

```
{
    "Data": {
        "DomesticPaymentId": "PDC_8de73abd-3e3c-4b60-90eb-20aef8d85a43",
        "ConsentId": "PDC_8de73abd-3e3c-4b60-90eb-20aef8d85a43",
        "CreationDateTime": "2019-10-29T12:34:47.567Z",
        "Status": "AcceptedSettlementCompleted",
        "StatusUpdateDateTime": "2019-10-29T12:34:47.566Z",
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
        "Self": "https://matls.rs.aspsp.ob.forgerock.financial/open-banking/v3.1/pisp/domestic-payments/PDC_8de73abd-3e3c-4b60-90eb-20aef8d85a43"
    },
    "Meta": {}
}
```

#### Being notified the payment has been processed
It's not very effective to keep polling an API for an indefinite amount of time until the payment has been processed so there is a mechanism for subscribing for event notifications. More can be found at the Open Banking [Confluence](https://openbanking.atlassian.net/wiki/spaces/DZ/pages/1077806674/Event+Notification+Subscription+API+-+v3.1.2#EventNotificationSubscriptionAPI-v3.1.2-RealTimeEventNotifications)

## Conclusion
It's not so trivial to become a PISP but given we can use the sandbox we can still innovate before officially becoming a PISP. There are many different types of payments you can make but the flow for all of them are very similar once you understand the flow it's only a matter of changing the payload and the URL and you can fully leverage the power of being a PISP.

![Payment](https://i.imgur.com/nqphqTK.gifv)

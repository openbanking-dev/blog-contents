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

### Ways Making Payments
Given the payment types we've already seen there are different ways of making those payments.

#### Single Payment
A way of a user giving consent to make a payment only once. It is not required to know the account the payment will come from ahead of time.

#### Scheduled Payment
A way of a user giving consent to make a payment at a future time. The users account will be debited at the time defined in `RequestedExecutionDateTime`. It is not required to know the account the payment will come from ahead of time.

#### Standing Order Payment
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

*Q comment: We can point to the trust model article*
To become a payment initiation services (PISPs) which is required to be able to make payments you need to have some capital behind you which might not be realistic for a developer just starting off with a cool idea. No need to worry as we can use the [Forgerock Reference Implementation Sandbox](https://openbanking4.dev/2019/10/27/learn-how-to-use-open-banking-uk-standard/) to get going.

#### On-boarding
*Q comment: same but with on-boarding*
As a developer you'll need to onboard. You can follow the [on-boarding guide](https://openbanking4.dev/2019/10/27/learn-how-to-use-open-banking-uk-standard/) to do this

#### Postman
As a part of on-boarding you would have used Postman. As you'll already be familiar with it we'll follow on with this tool for the payment example. We'll be using [Domestic single payment](https://postman.ob.forgerock.financial/?version=latest#a1537aad-f5dc-4bb1-9a8a-e2323b48e59b) from the Forgerock OBRI Sample collection.

### Request For Payment
In order for Alice to make a payment we'll need to make a payment request to the bank also known as creating the consent. There are two stages to this:

1. Getting an access token
2. Creating the payment request

#### Getting an access token
As open banking is built on OIDC so we'll need to authenticate as the developer with the supported authentication types. We'll use the client credential flow with a client assertion here.

To get an access token we'll need to authenticate. For this we'll use a client assertion; a JWT which has been signed with your signing private key which will be enough to prove who you are. To make this easier the Forgerock Reference Implementation Sandbox has a utility API to do this on your behalf.

Once we have our client assertion we can use it as authentication to get an access token.

During the next few steps the postman collection is set up to set and share variables between requests to make it easier to use. This will continue throughout the example.

*Q comment: we will switch to MATLS auth method, easier for the TPP*

1. Open ForgeRock OBRI Sample
2. Open `Payment flows` -> `Payment API V3.1` -> `Domestic Payments` -> `Domestic single payment` -> `Payment preparation`
4. Select `Client credentials` and send the request where you'll receive an access token

#### Creating the payment request
Now we've received an access token we'll be able to create the payment request. To do this we will use our access token as a bearer token in the `Authorization` HTTP header.

The next few steps have predefined payloads to make usage easier but you'll be able to play and tweak these.

*Q comment: I did made the previous articles a bit less dependent of postman. I think potentially a middle ground. I will add the link to the postman corresponding to the request and perhaps you can quote the requests each time?*

1. Select `Create domestic payment consent` and send the request to create the payment request
2. Now you've created the request you can get it to check the status by selecting `Get domestic payment consent` and clicking send

### User consent
The payment request has been created and now it's time for Alice to review and approve the payment. During this next step we'll be simulating the developer redirecting Alice to her bank to approve the payment.

There will be 2 steps to this step. 

1. Creating the [request parameter](https://openbanking.atlassian.net/wiki/spaces/DZ/pages/7046134/Open+Banking+Security+Profile+-+Implementer+s+Draft+v1.1.0#OpenBankingSecurityProfile-Implementer'sDraftv1.1.0-HybridGrantParameters) which most importantly contains the payment ID. 

2. Approve the payment as Alice

#### Creating the request parameter
The next step have predefined payloads which will be populated with some values but more specifically the ID of the payment consent we previously created. The request parameters will again be a signed JWT which you would sign with your signing private key but we'll use the utility API and it will be signed on your behalf.

1. Open `Auth & Consent` 
2. Select `Generate request parameter` and send and in response you'll get the signed request parameter

#### Approve the payment as Alice
Here we'll simulate being redirected from the developers shop website to the bank to approve the payment request.

1. Select `Hybrid flow - OB official API`
2. Press the `Code` link
3. Select cURL
4. Select the URL part of the request and copy it
5. Paste the URL into a browser
6. You'll be able to review and approve the payment and be redirected to the developers shop
7. In the URL you'll see and exchange code in the form `code=<exchange code>`, copy it and put it somewhere for use later


### Submit Payment
Alice has approved the payment for her raspberry pi but we still haven't submitted the payment. The payment has still only be requested at this point. The next part of this will be submitting the payment to the bank. We'll walk through 4 different steps here of which 2 are optional but would be good practice in developing a payment system.

1. Getting an access token
2. Confirming Alice has the money to make the payment (Optional)
3. Submitting the payment
4. Checking the payment has been processed

#### Getting an access token
Here we'll be using the exchange code we put somewhere earlier for later use. The exchange code can be exchanged for an access token. It's worth noting that Alice couldn't use this exchange code to get an access token herself or anyone else for that matter.
*Q comment: we probably overlap with the security article*

The next step again stores and shares variables between steps for ease of use.

1. Open `Exchange code - OB official API`
2. Select `Exchange code - OB official API` and send the request. In response you'll get an access token


#### Confirming Alice has the money to make the payment
Before submitting the payment, we may want to check Alice has the money to be able to make this payment.

1. Open `Data access`
2. Select `Get confirmation of fund` and click send and the response will determine whether funds are available

#### Submitting the payment
Given funds are available you'll be able to successfully submit the payment.

Select `Domestic Payment` and click send. In the response will be a status which will indicate if the payment has been processed yet. It's likely that the payment will be processed asynchronously

#### Checking the payment has been processed
Given the payment may be processed asynchronously, as the shop, you may want to check the payment has been processed before dispatching the goods. There is an API to get the payment.

Select `Get Domestic Payment` and click send. You can keep polling this API until the payment has been processed

#### Being notified the payment has been processed
It's not very effective to keep polling an API for an indefinite amount of time until the payment has been processed so there is a mechanism for subscribing for event notifications. More can be found at the Open Banking [Confluence](https://openbanking.atlassian.net/wiki/spaces/DZ/pages/1077806674/Event+Notification+Subscription+API+-+v3.1.2#EventNotificationSubscriptionAPI-v3.1.2-RealTimeEventNotifications)

## Conclusion

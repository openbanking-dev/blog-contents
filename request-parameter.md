# The request parameter in Open Banking UK

*This article is written as an appendix of our Open Banking UK tutorial. We recommend reading first the security flow article, that will cover the global flow that involved the request parameter. However it still can be read as a standalone article if you got already enough background in Open Banking to jump in the request parameter world!*


## Introduction

Open Banking involves three parties: the developer (you), Alice and the bank. The Open Banking UK is based on a redirection model, meaning that you will use a redirection to make Alice in contact with her the bank, with a context attached to it.
It will allow the bank to understand the reason Alice was redirect to them and will load the login page and consent page accordingly.
This imply you can easily transmit a context to the bank, in a secured way. This context is nothing else than the request parameter.
A confusing name, reason being is it comes from the OIDC standard and was seen as a way to bundle the `GET` request parameters all togethers and sign them.

## Why the request parameter over Get parameters?

You probably used to redirect users to other parties, using the GET parameters.
An example of it could be a simple link in your app to an offer from a partner. The link would look like:

`https://awesomepartner.com?offer=123&promo=WINTER_10`

The `awesomepartner` will be able to understand it has to loads the offer `123` to the user with the promo code `WINTER_10` corresponding to 10%.

It works well but obviously like me, you already thinking: 

> mmmh, what about I change it to WINTER_20, should I get 20%?

As a user, you can change the `GET` parameter on the way in and try to access things you were not supposed to initially.

You can understand that in the case of Open Banking, where we want to share account data and even makes payments, we are trying to prevent the user to be able to play with those parameters.

The idea of the request parameter comes from there: Let's bundle all the parameters together and sign them. This way, if the user is modifying the values, the signature will be broken and the server will be able to reject the request.


## What is the request parameter

Continuing on our example above, if we were to do the same URI but with a request parameter, we would end up with a URI:

```
https://awesomepartner.com?request=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJwcm9tbyI6IldJTlRFUl8xMCIsIm9mZmVyIjoiMTIzIn0.O1AuKtSZqRZZ84RPVl7angL-d1rWEcFLS8wO0lXfH-E
```

You may be thinking: `what the F*** is this ?!`. Let me introduce you to a JWS. JWS stands for JSON Web Token Signed.
In other word, it's just a JSON signed and base64 encoded to make it more web compatible.

### How to read the request parameter?

You can use some free services like jwt.io.
For our token above, you can go to [https://jwt.io/#debugger-io?token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJwcm9tbyI6IldJTlRFUl8xMCIsIm9mZmVyIjoiMTIzIn0.O1AuKtSZqRZZ84RPVl7angL-d1rWEcFLS8wO0lXfH-E](https://jwt.io/#debugger-io?token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJwcm9tbyI6IldJTlRFUl8xMCIsIm9mZmVyIjoiMTIzIn0.O1AuKtSZqRZZ84RPVl7angL-d1rWEcFLS8wO0lXfH-E).

You will see that as a payload, we got:

```
{
  "promo": "WINTER_10",
  "offer": "123"
 }
```

I haven't lie, the previous parameters are now bundle together, in a JSON format. The value really of all of this, it's because it is signed. Here, I signed it with HS256, an algorithm that sign things using a password. For this JWS, I used `password` as a password.

You can actually verify that signature with jwt.io:

If you look at just under the JWT on the left side, you probably notice the `Invalid Signature`. It's because by default, jwt.io is trying to verify the signature with `your-256-bit-secret`, which is not the right one. You can change it on the right side to `password` and you will see `Signature Verified` instead.

You should already feel the potential of using JWS over GET parameters. It doesn't make sense in all scenarios but certainly in cases like Open Banking or even more generic, Open API, using a request parameter is an elegant way to secure this redirection.
 

## How to build the request parameter for Open Banking

Building a JWS from a JSON is something well-known and covered by plenty of libraries. In JWT.io, you can even see the list of libraries for each languages.
Instead of showing you how to use all those libraries, in this article, we will concentrate our efforts on describing the format of the JSON for Open Banking.

As a developer, you need to agree with the bank on the format of this JSON. We touch briefly that this was used by OIDC, and has it's importance now as OIDC is actually standardise the format of the request parameter.
If you are interested to read the specification in details, this is cover by [https://openid.net/specs/openid-connect-core-1_0.html#JWTRequests](https://openid.net/specs/openid-connect-core-1_0.html#JWTRequests).

Instead of boring you with all the details, we will give you directly a template for the request parameter:

```
{
  "iss": "{{CLIENT_ID}}",
  "aud": "{{AS_ISSUER_ID}}",
  "scope": "openid {{SCOPES}}",
  "claims": {
    "id_token": {
      "acr": {
        "value": "urn:openbanking:psd2:sca",
        "essential": true
      },
      "openbanking_intent_id": {
        "value": "{{CONSENT_ID}}",
        "essential": true
      }
    },
    "userinfo": {
      "openbanking_intent_id": {
        "value": "{{CONSENT_ID}}",
        "essential": true
      }
    }
  },
  "response_type": "code id_token",
  "redirect_uri": "{{CLIENT_REDIRECT_URI}}",
  "state": "{{STATE}}",
  "exp": {{EXP}},
  "nonce": "{{STATE}}",
  "client_id": "{{CLIENT_ID}}"
}
```

Let's describe all those variables:

- `CLIENT_ID`: It's basically your ID. Before reaching the point of building this request parameter, you must have go through the on-boarding process. This will give you some credentials, including this client ID. If you want to know more about on-boarding, you can read this article.
- `AS_ISSUER_ID`: It's the bank ID. As the request parameter is designed for a specific bank, you are specifying as 'audience' of your JWT Alice's bank ID. In the case of the ForgeRock mock bank, this ID would look be `https://as.aspsp.ob.forgerock.financial/oauth2`.
- `SCOPES`: The list of scopes you need for this request. If you are doing a payments, the value would be `payments` or for an account sharing, `accounts`.
- `CONSENT_ID`: You must have done the `consent preparation` step first. Part of this process have return you a `consent id`, which you can refer here.
- `CLIENT_REDIRECT_URI`: Your redirect URI, the one that will be used by the bank to redirect Alice back to your application.
- `STATE`: A state value that will be carry over the flow. It's a value design for you, the bank does nothing with it. The idea is to allow you to attach a value `state` that will help you reload the context of the request. As it is an un-synchronous flow, you will need it to retrieve state in your application.
- `EXP`: The expiration of your request parameter. Usually a few minutes, as Alice would consume this request parameter straight away. The format is a number in second elapse since `1970-01-01`. Example: `1573573045.354` -> `Tue Nov 12 2019 15:37:25 GMT+0000 (Greenwich Mean Time)`

## A step back

Let's take a minute to step back, and talk a bit about the magic behind this request parameter. 
We saw the advantage of signing the requests parameters this way in the first section. If you look closer, you will find another trick hide inside this request parameter: we attached a `CONSENT_ID` to the request.
Indeed, even if the request parameter can be generalised like we did in our previous example, it's best to remind us that this is not any kind of request parameter: It's the one defined by OIDC,  a standard on top of OAUTH2 that allows different parties to share Alice data securely. 
If you know a bit about OAuth2/OIDC, you probably know about scopes. Scopes are not been designed initially to carry a context ID, but are more global, like `accounts` or `payments`.
What Open Banking just did by finding a way to attach this consent ID to the request parameter in a standard way, is binding the authorisation flow to a specific context.
As a developer consuming this service, it may not change things much but behind the scene, this was a real trick to make Open Banking relying on standards like OIDC and OAuth2, and still provide a rich consent journey.
The fact the JSON is a bit convoluted to attach this consent ID, as you probably noticed it is under `claims` -> `id_token` -> `openbanking_intent_id` -> `value`, is just to make it OIDC standard compliant and been able to re-use a standard with years of experiences. A good way to not re-invent the wheel if you want my opinion. 
It probably highlights that OIDC needs to adapt and adopt this new requirements of attaching context to an authorisation flow.
I hope this section gives you a taste of why this `claims` is all about and you appreciate the reason of having a template!

## Sign the request parameter

Signing the request can be done with a library but still, it will requires you to specify which algorithm to use, which keys etc.
In Open Banking, it's been decided that the algorithm recommended would be `PS256`. It requires an RSA key pair. 
At this point, you should already have sorted your keys in your backend. All you need to know is that it's your signing key you should used. If you are using a directory, it would be the one you uploaded the CSR and got the PEM from it. If you are using EIDAS certificates, it's the QSEAL.

You will need to specify the KID of this signing key. In the case of the Open Banking directory or ForgeRoc directory, the value can be found in the directory UI.

## Conclusion

The request parameter is a key component of the Open Banking flow. It provides a layer of security and add a nice way to attach a context to the authorisation flow.
In fact, it's probably one of the main reason that we use OIDC and not just OAuth 2, just because this request parameter lives in the OIDC standard specification. Plenty of other advantages of using OIDC over just OAuth 2, which would probably the subject a future article!
In the meantime, we hope you learned a bit more about this request parameter and the value it brings to a world of Open API build on top of OIDC/OAuth2 standards.
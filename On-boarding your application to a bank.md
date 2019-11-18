# On-boarding your application to the bank

*This article is part of a set of tutorial about Open Banking UK, although, they are all written to be ready as a standalone article. If you are interested in learning about Open Banking UK, you may want to start by our intro article.*

In order to use the Open Banking APIs, you will need to register your application to each of the banks. We will see in this article how the on-boarding process with each bank works.

For the moment, you are an anonymous developer to the eco-system. There is an Open Banking circle of trust system and basically your first mission is to join it.
Different one exist in parallel, for our set of tutorials, we are going to use the ForgeRock model bank and their ForgeRock Open Banking directory
The advantage of using this one is that anyone can register, which is ideal for learning Open Banking.

## Register in the ForgeRock Open Banking directory

### Create an account
Go to https://directory.ob.forgerock.financial/ and register a new developer account.
Once login, you would see a dashboard, with your organisation and the list of your software statements.

### Edit your organisation
You can edit your organisation to reflect a bit more who you are. Some of the values will be used during the Open Banking flow, especially during the user authorise request.
I would definitively complete this section if you intent to do a demo of Open Banking.

### Create a software statement
A software statement is another word to say application. You can choose to have one application register and use it for everything, or you can choose to create multiples applications, up to you.
Start by clicking on `Create a new software statement`. 


You will see on the top that there are a few tabs:

- `General`: it's the global information of your application
- `Transport/Signing/Encryption keys`: cryptography is fundamental for Open Banking. This is the section where you will get your tests keys.
- `Software Statement Assertions (SSA)`: It's your financial passport if you like. It is a JSON signed (JWS) by the directory, that will attest that you are part of the circle of trust. This tab allows you to generate a new SSA on demand.
- `On Boarding`: the directory can onboard for you to the ForgeRock bank. Although, we kind of want how the on-boarding works so we are going to do that manually.

Cool fact: the directory also offers APIs, we will be able to do the same things with the APIs than the UI. Very handy especially for generating a new SSA.

#### General

You can complete the entire form but if you feel a bit lazy like me, just complete the essential:

- `Name`: It will displayed in the consent screen, it's nice to show a friendly name than a blank
- `redirect URIs`: we don't want to host an app for exploring Open Banking, we are going to use google. Put `https://google.com`
- `logo`: It's also display in the consent. You can use ours for the exercise if you like. 


#### Transport/Signing/Encryption keys

The way you are going to authenticate, when doing a call the APIs, is via MATLS. It means that server is going to present a certificates, called server certificate (basically what is httpS for) but you are going to present one too, which we call client certificate.
Instead of naming them server and client certificates, the directory is offering you transport certificate. It's because the directory doesn't care if you are a server or a client, so from its point of view, they are TLS certificates, which we also call transport certificate.
You will basically needs to download your transport certificates and use it as client certificates.
All of that was just to say to download your current transport key :) 
A certificate is the representation of an asymmetric key. Asymmetric implies a public and a private key.
You will need to download the .pem and .key, in order to use this key in the future.

Note that you can also download your signing key the same way. In this tutorial, we are going to use the JWKMS APIs, which would allow us to use this signing key indirectly, by authenticating with our transport certificate. 
This means you don't need to download your signing key to follow this tutorial.

#### Software Statement Assertions

You can try generating an SSA via the UI. It's a good way to explore.
For our tutorial, we will do the same but via the directory API. No need to download an SSA now.

#### On-boarding

We are not going to onboard using the directory. Instead, we will show you how to do it via APIs. You can skip this tab for today.

## Register your application to the bank

You are now ready to register your application to the bank. As a reminder, you are registered in the directory and recognised now as part of the Open Banking community for the sandbox, but the Forgerock mock bank itself doesn't know you yet.

Under the cover, Open Banking uses OIDC and technically, you are not registered as an OIDC client for the bank yet.

### Setting up Postman

If you are interested to follow this tutorial and execute the requests at the same time, you can follow this article that will help you setting up Postman.
For each request example, we will point to the postman request corresponding so you can execute it on Postman.

### Dynamic registration

The nice things with Open Banking, is once you managed to be register into the directory, you can do a dynamic registration to the bank. What this means is that there isn't any manual process involved, no approval requires etc.
By calling the dynamic registration APIs the right way, you directly get your OIDC clients register to the bank and you can start using the Open Banking APIs.

Some banks are still not on top of the art on that subject and still offer manual registration. Unfortunately for those, you would have to follow their process, which would be unique for each banks. They are not a majority though. Not ideal for you but let's hope they all move on and adopt dynamic registration. 
For this tutorial, we will show you dynamic way.

#### The AS discovery

Before we go in building this registration request, it's important to first read the supported features from the bank. 
The AS (the Bank) discovery gives you the list of supported security features from this bank.
What is nice about this is that it's the only endpoint you need to know to retrieve all the rest of the information from the bank.
It's also standardise, which is awesome as you would be able to automatise the discovery of the banks and maintain one piece of code!
The standard is there: https://openid.net/specs/openid-connect-discovery-1_0.html

Here is an example of what ForgeRock mock bank supports:

```
GET /oauth2/.well-known/openid-configuration HTTP/1.1
Host: as.aspsp.ob.forgerock.financial
```

```
  {
    "version": "3.1.2",
    "issuer": "https://as.aspsp.ob.forgerock.financial/oauth2",
    "authorization_endpoint": "https://as.aspsp.ob.forgerock.financial/oauth2/authorize",
    "token_endpoint": "https://matls.as.aspsp.ob.forgerock.financial/oauth2/access_token",
    "userinfo_endpoint": "https://matls.as.aspsp.ob.forgerock.financial/oauth2/userinfo",
    "introspection_endpoint": "https://matls.as.aspsp.ob.forgerock.financial/oauth2/introspect",
    "jwks_uri": "https://as.aspsp.ob.forgerock.financial/api/jwk/jwk_uri",
    "registration_endpoint": "https://matls.as.aspsp.ob.forgerock.financial/open-banking/register/",
    "scopes_supported": [
        "openid",
        "payments",
        "fundsconfirmations",
        "accounts"
    ],
    "response_types_supported": [
        "code token id_token",
        "code",
        "code id_token",
        "device_code",
        "id_token",
        "code token",
        "token",
        "token id_token"
    ],
    "grant_types_supported": [
        "refresh_token",
        "client_credentials",
        "authorization_code"
    ],
    "acr_values_supported": [
        "urn:openbanking:psd2:sca",
        "urn:openbanking:psd2:ca"
    ],
    "subject_types_supported": [
        "public",
        "pairwise"
    ],
    "id_token_signing_alg_values_supported": [
        "RS256",
        "PS256"
    ],
    "id_token_encryption_alg_values_supported": [
        "RSA-OAEP",
        "RSA-OAEP-256",
        "A128KW",
        "A256KW",
        "RSA1_5",
        "dir",
        "A192KW"
    ],
    "id_token_encryption_enc_values_supported": [
        "A256GCM",
        "A192GCM",
        "A128GCM",
        "A128CBC-HS256",
        "A192CBC-HS384",
        "A256CBC-HS512"
    ],
    "userinfo_signing_alg_values_supported": [
        "ES384",
        "HS256",
        "HS512",
        "ES256",
        "RS256",
        "HS384",
        "ES512"
    ],
    "userinfo_encryption_alg_values_supported": [
        "RSA-OAEP",
        "RSA-OAEP-256",
        "A128KW",
        "A256KW",
        "RSA1_5",
        "dir",
        "A192KW"
    ],
    "userinfo_encryption_enc_values_supported": [
        "A256GCM",
        "A192GCM",
        "A128GCM",
        "A128CBC-HS256",
        "A192CBC-HS384",
        "A256CBC-HS512"
    ],
    "request_object_signing_alg_values_supported": [
        "RS256",
        "PS256"
    ],
    "request_object_encryption_alg_values_supported": [
        "RSA-OAEP",
        "RSA-OAEP-256",
        "A128KW",
        "RSA1_5",
        "A256KW",
        "dir",
        "A192KW"
    ],
    "request_object_encryption_enc_values_supported": [
        "A256GCM",
        "A192GCM",
        "A128GCM",
        "A128CBC-HS256",
        "A192CBC-HS384",
        "A256CBC-HS512"
    ],
    "token_endpoint_auth_methods_supported": [
        "client_secret_post",
        "private_key_jwt",
        "client_secret_basic",
        "tls_client_auth"
    ],
    "token_endpoint_auth_signing_alg_values_supported": [
        "RS256",
        "PS256"
    ],
    "claims_supported": [
        "acr",
        "zoneinfo",
        "openbanking_intent_id",
        "address",
        "profile",
        "name",
        "phone_number",
        "given_name",
        "locale",
        "family_name",
        "email"
    ],
    "claims_parameter_supported": true,
    "request_parameter_supported": true,
    "request_uri_parameter_supported": true,
    "require_request_uri_registration": true
}
```

**Postman: ** You can find this request in the ForgeRock Postman collection under `Setup your environment` > `Discovery` > `AS Discovery`. The link from the online documentation directly to this request is https://postman.ob.forgerock.financial/?version=latest#69195c1d-6d7a-4971-b9bb-51c0e7dff2f2


It's a lot, probably too much information to go through now.
Please refer to the standard to get all the details about all the claims but you don't need to do that now.
Let's concentrate on the essentials claims for our tutorial:

- `issuer`: it's the id to use when referring to the bank, when signing/receiving messages from it. Doing the analogy with sending a postcard, basically it's like the official name to put in a letter, if you wanted to send it to the bank. It's also the name you will see in a message you received from the bank. You will be able to tell that this message from the bank A this way.
- `authorization_endpoint`: we will need to redirect the user to the bank at some point. It's this endpoint that the bank is telling us to redirect the user to when requesting the user authorisation.
- `token_endpoint`: The token endpoint will be useful for us, to retrieve an access token. 
- `registration_endpoint`: The registration endpoint, very handy for this article as we are going to use now.
- `scopes_supported`: A very useful one to checkout, as you may realise that the bank doesn't supports payments for example. May save you time if you are just interested in providing payment service and this bank is not offering those APIs yet.
- `token_endpoint_auth_methods_supported`: List the different authentication that you will be able to use with this bank. The four mains one describe by Open Banking are:

	- `client_secret_post` and `client_secret_basic`: Not recommended but still used by Open Banking, just to allow more flexibilities at the beginning. It's basically a password authentication. Main reason of not recommending it is if you compromise the password, well, it's not easy to get a new one. Password-less is pretty cool and you will see that the two next one allows you to do that.
	- `private_key_jwt`: The idea is to sign a JSON payload with your keys, and by agreeing on the payload of the JSON, this JWT becomes a form of ID (called client assertions). It's a nice usage of the signing concept to provide a way of authenticating people. We will see in more details how this work in this article.
	- `tls_client_auth`: More standard in the industry but not very popular to developers, this method allows you authenticating using a certificate. Commonly called MATLS in the rest of the industry, this method consist of sending a client certificate as part of TLS connection.


##### Which authentication method to use?

It's nice to have the choice but which one should you use? First, it's important to note that as some banks would not support all of the authentication methods yet, you probably to end up supporting all of them, just so you can consume all the banks APIs available.
The question becomes, once you support all the methods, what is the preferred order of the authentication methods?
If I had to say, here is the order I would say for Open Banking:

- `tls_client_auth`
- `private_key_jwt`
- `client_secret_basic`
- `client_secret_post`

I would choose `tls_client_auth` first, as for Open Banking you need to setup your client certificates anyway, this one is the most convenient for you.

#### Dynamic registration request JWT

Coming back to on boarding, let's talk about the registration request.

The dynamic registration request takes as a payload a JWS, which is a JSON signed.

The format of this JSON is standardise by two standards:

- OIDC dynamic registration https://openid.net/specs/openid-connect-registration-1_0.html
- OAuth 2 dynamic registration https://tools.ietf.org/html/rfc7591

I would recommend reading the OIDC dynamic registration first, it's easier to read in my opinion. The important things to note between the two is OAuth 2 dynamic registration defines the concept of SSA (it's our financial passport, see Directory article).

For this article, we are going to use the following JSON as a base line:

```
{
  "exp": 1572287747.598,
  "scope": "openid accounts payments fundsconfirmations",
  "redirect_uris": [
    "https://www.google.com"
    ],
  "grant_types": [
    "authorization_code",
    "refresh_token",
    "client_credentials"
  ],
  "response_types": [
    "code id_token"
  ],
  "subject_type": "pairwise",
  "software_statement": "{{SSA_JWT}}",
  "token_endpoint_auth_method": "tls_client_auth",
  "token_endpoint_auth_signing_alg": "PS256",
  "id_token_signed_response_alg": "PS256",
  "request_object_signing_alg": "PS256",
  "request_object_encryption_alg": "RSA-OAEP-256",
  "request_object_encryption_enc": "A128CBC-HS256"
}
```


Lets describe the essential claims of this JSON:

- `exp`: It's the expiration time of your JWS. For security reason, it's good to keep it short. No real reason to having it valid for days, when you are going to consume it just after creating it.
- `scope`: it's the list of scopes you are requesting. Scopes allow you to access specific kind of resources. 
	- `openid`: this one just need to be there, to say we are going to use not only OAuth 2 but also OpenID. 
	- `accounts`: required to access accounts APIs
	- `payments`: required to access payments APIs.
	- `fundsconfirmations`: required to access confirmation of funds APIs.
  It's important to note that the requested scopes would be double checked by the SSA.
- `redirect_uris`: Redirect URIs used during the security flow, which we will cover in the next article, you will see that the flow is based on what we call a redirection model flow. At some point, you will need to receive a callback, with the response from the bank. For this tutorial, we are going to use google.com. You will see that it's fine to use google for this tutorial, as we will manual extract the response from the parameters. 
  An important things to note, that comes back to me regularly as a question, is if you happen to change your redirection Uris for a reason, you will need to update your OIDC client in each of the bank. What this means is that you will need to do the dynamic registration again with all the banks you already onboard. Fortunately there is a PUT method for the dynamic registration but still, if you are onboard with 1000 banks, it can be a long process.
  People usually things that by updating it in the directory, it's enough. It's the first step to do but unfortunately not sufficient. It will change the redirect Uris in your financial passport, which is necessary, but the banks are not aware of this change yet, until you do a PUT in the dynamic registration.
- `software_statement`: this is where you copy-paste your SSA from the directory. The SSA is a JWS as well, it's a bit of a JWS inception we are doing here but note that the SSA is signed by the directory, and the dynamic registration JWS signed by you. If you think of it, it's a nice way to make sure you are authorised to register, certified by a central authority, the directory in our case, and also verifying your identity at the same time.
- `token_endpoint_auth_method`: choose the authentication method to use with this bank. As mentioned in the previous section, this method needs to be supported by the bank first.

##### Getting your SSA

You can either go to the directory UI and download the SSA from the API, or for this tutorial, use the ForgeRock directory APIs to get it programmatically.

Note: The ForgeRock directory is an online services that used MATLS to authenticate you and generate an SSA corresponding to your software statement. For that, you need to setup MATLS properly, which is the dedicated subject on this article.

The API is very simple, all you need to do is:

```
POST /api/software-statement/current/ssa HTTP/1.1
Host: matls.service.directory.ob.forgerock.financial
```

```
eyJraWQiOiIwYTNlNGJhYzVmMTg2OTFlNTcxNGJkNjM2ZWY4YzhjYWI2MmJlY2Q3IiwiYWxnIjoiUFMyNTYifQ.eyJvcmdfandrc19lbmRwb2ludCI6IlRPRE8iLCJzb2Z0d2FyZV9tb2RlIjoiVEVTVCIsInNvZnR3YXJlX3JlZGlyZWN0X3VyaXMiOltdLCJvcmdfc3RhdHVzIjoiQWN0aXZlIiwic29mdHdhcmVfY2xpZW50X25hbWUiOiJGb3JUZXN0XzcwN2U2MDgxLThhYzItNDNiOC05YTlmLWZhNzViZWUyMmRjOSIsInNvZnR3YXJlX2NsaWVudF9pZCI6IjcwN2U2MDgxLThhYzItNDNiOC05YTlmLWZhNzViZWUyMmRjOSIsImlzcyI6IkZvcmdlUm9jayIsInNvZnR3YXJlX2p3a3NfZW5kcG9pbnQiOiJodHRwczpcL1wvc2VydmljZS5kaXJlY3Rvcnkub2IuZm9yZ2Vyb2NrLmZpbmFuY2lhbDo0NDNcL2FwaVwvc29mdHdhcmUtc3RhdGVtZW50XC83MDdlNjA4MS04YWMyLTQzYjgtOWE5Zi1mYTc1YmVlMjJkYzlcL2FwcGxpY2F0aW9uXC9qd2tfdXJpIiwic29mdHdhcmVfaWQiOiI3MDdlNjA4MS04YWMyLTQzYjgtOWE5Zi1mYTc1YmVlMjJkYzkiLCJvcmdfY29udGFjdHMiOltdLCJvYl9yZWdpc3RyeV90b3MiOiJodHRwczpcL1wvZGlyZWN0b3J5Lm9iLmZvcmdlcm9jay5maW5hbmNpYWw6NDQzXC90b3NcLyIsIm9yZ19pZCI6IjVjNDVmODJkYTkzYjc1MDEyNWM3YWRlYiIsInNvZnR3YXJlX2xvZ29fdXJpIjoiaHR0cHM6XC9cL2kucG9zdGltZy5jY1wvaHRoUUNKaFJcL2ZyLWxvZ28tc3F1YXJlLTFjLWJsYWNrLnBuZyIsInNvZnR3YXJlX2p3a3NfcmV2b2tlZF9lbmRwb2ludCI6IlRPRE8iLCJzb2Z0d2FyZV9yb2xlcyI6WyJQSVNQIiwiREFUQSIsIkNCUElJIiwiQUlTUCJdLCJleHAiOjE1NzMzOTM3MzMsIm9yZ19uYW1lIjoiRm9yZ2VSb2NrIiwib3JnX2p3a3NfcmV2b2tlZF9lbmRwb2ludCI6IlRPRE8iLCJpYXQiOjE1NzI3ODg5MzMsImp0aSI6IjFlMGYyNGE2LTc2OWQtNGNjMi1hMWUxLTg1NDdmNmYyZjAxMiJ9.QC3NBjFhlHSl00wYfTWfgmpA0qLr_hV_5Urn2bsq5CJG2ZLxD52n9cBDnxgE41zorVQnzwDhcTipQ6W5t8hU9Da9HH4q8GhsV2W5uVFtpUy3M_oBbosmpHu55T7wKZpfoMGdEQ-zMVGvCsPuUqHXPlmL1TvC6uD1QLORCnwJORpH87d6DyqJ15a5yv8BjXiz2r1EKgoKr2k9jYoH8KpGAc3JA5XW5iCB1urLGb6nxHyPXjRuMEtTiVA-kybmkPoSClxLNijUPPSnxcL1WvMBswpHysOEUQtTZYa2MNS0xxoXh3KJIy5CU-q_zt8Xr474NFQV1dSO0T9JDjIl1NtgiQ
```

**Postman: ** You can find this request in the ForgeRock Postman collection under `Dynamic Registration` > `Onboarding your TPP` > `Generate SSA`. The link from the online documentation directly to this request is https://postman.ob.forgerock.financial/?version=latest#33781875-cf87-4990-8f90-3577467869b7

You can store this SSA into a variable to re-use it in the next section

##### Sign the registration request into a JWS

What you need to do now is signing this JSON with your signing key. This JSON then becomes a JWS.
There is plenty of libraries around to do this, which will not cover in that article.
In order to make the learning curve easier for you and to not have to worry yet about how to sign a JSON as JWS, ForgeRock offers a service, called JWKMS, which will do that for you.
It's not intent to be used for production, but just as a way for you to not have to worry about signing, at the beginning of your Open Banking learning journey.

Note: the JWKMS is an online service, which uses MATLS to authenticate you and sign messages on your behalf. For that, you need to setup MATLS properly, which is the dedicated subject on this article.

Before calling the JWKMS to sign our JSON, just one detail to sort up: Getting your software statement ID. The specification says that this JWS needs to be signed, using your software statement ID as an issuer ID. You can either go to the directory UI and get the software statement ID from there, or use again the directory APIs:

```
GET /api/software-statement/current/ HTTP/1.1
Host: matls.service.directory.ob.forgerock.financial
```

```
{
    "id": "707e6081-8ac2-43b8-9a9f-fa75bee22dc9",
    "name": "ForTest_707e6081-8ac2-43b8-9a9f-fa75bee22dc9",
    "logoUri": "https://i.postimg.cc/hthQCJhR/fr-logo-square-1c-black.png",
    "mode": "TEST",
    "roles": [
        "PISP",
        "DATA",
        "CBPII",
        "AISP"
    ],
    "status": "ACTIVE",
    "redirectUris": [],
    "applicationId": "5dbed5691c9100001a75abc2"
}
```

**Postman: ** You can find this request in the ForgeRock Postman collection under `Dynamic Registration` > `Onboarding your TPP` > `Current software statement`. The link from the online documentation directly to this request is https://postman.ob.forgerock.financial/?version=latest#4f916da5-a872-4b0c-8fe1-895a67c03b1d

Plenty of information in this response but we will only keep the `id` for our next request: signing the registration request!

For us today, we will use the JWKMS APIs. For signing our JSON payload, we call:

```
POST /api/crypto/signClaims HTTP/1.1
Host: jwkms.ob.forgerock.financial
Content-Type: application/json
issuerId: 71a6b418-089b-47d7-ab31-703f7beca9fe

{
  "exp": 1572787852.957,
  "scope": "openid accounts payments fundsconfirmations",
  "redirect_uris": [
    "https://www.google.com"
    ],
  "grant_types": [
    "authorization_code",
    "refresh_token",
    "client_credentials"
  ],
  "response_types": [
    "code id_token"
  ],
  "subject_type": "pairwise",
  "software_statement": "eyJraWQiOiIwYTNlNGJhYzVmMTg2OTFlNTcxNGJkNjM2ZWY4YzhjYWI2MmJlY2Q3IiwiYWxnIjoiUFMyNTYifQ.eyJvcmdfandrc19lbmRwb2ludCI6IlRPRE8iLCJzb2Z0d2FyZV9tb2RlIjoiVEVTVCIsInNvZnR3YXJlX3JlZGlyZWN0X3VyaXMiOltdLCJvcmdfc3RhdHVzIjoiQWN0aXZlIiwic29mdHdhcmVfY2xpZW50X25hbWUiOiJGb3JUZXN0XzcwN2U2MDgxLThhYzItNDNiOC05YTlmLWZhNzViZWUyMmRjOSIsInNvZnR3YXJlX2NsaWVudF9pZCI6IjcwN2U2MDgxLThhYzItNDNiOC05YTlmLWZhNzViZWUyMmRjOSIsImlzcyI6IkZvcmdlUm9jayIsInNvZnR3YXJlX2p3a3NfZW5kcG9pbnQiOiJodHRwczpcL1wvc2VydmljZS5kaXJlY3Rvcnkub2IuZm9yZ2Vyb2NrLmZpbmFuY2lhbDo0NDNcL2FwaVwvc29mdHdhcmUtc3RhdGVtZW50XC83MDdlNjA4MS04YWMyLTQzYjgtOWE5Zi1mYTc1YmVlMjJkYzlcL2FwcGxpY2F0aW9uXC9qd2tfdXJpIiwic29mdHdhcmVfaWQiOiI3MDdlNjA4MS04YWMyLTQzYjgtOWE5Zi1mYTc1YmVlMjJkYzkiLCJvcmdfY29udGFjdHMiOltdLCJvYl9yZWdpc3RyeV90b3MiOiJodHRwczpcL1wvZGlyZWN0b3J5Lm9iLmZvcmdlcm9jay5maW5hbmNpYWw6NDQzXC90b3NcLyIsIm9yZ19pZCI6IjVjNDVmODJkYTkzYjc1MDEyNWM3YWRlYiIsInNvZnR3YXJlX2xvZ29fdXJpIjoiaHR0cHM6XC9cL2kucG9zdGltZy5jY1wvaHRoUUNKaFJcL2ZyLWxvZ28tc3F1YXJlLTFjLWJsYWNrLnBuZyIsInNvZnR3YXJlX2p3a3NfcmV2b2tlZF9lbmRwb2ludCI6IlRPRE8iLCJzb2Z0d2FyZV9yb2xlcyI6WyJDQlBJSSIsIlBJU1AiLCJEQVRBIiwiQUlTUCJdLCJleHAiOjE1NzI4OTIxNjcsIm9yZ19uYW1lIjoiRm9yZ2VSb2NrIiwib3JnX2p3a3NfcmV2b2tlZF9lbmRwb2ludCI6IlRPRE8iLCJpYXQiOjE1NzIyODczNjcsImp0aSI6Ijc3MDU5NDU5LThkODYtNDAyNS05ZmIyLWIwMjE4NDFlMzNjMyJ9.c0prwaiLGDvUXO4KU1zTKZEONdCMwcW_xZNk_3q858ezaZtpw4AZFEEIAUNPch89m3-W4XL95cNRPtkvKkb-A_hX63ffyAWu1ZlHszjPDWg5tAHQ-fosxMBfchFJHtMxp0FVHNY5BHpMWDtVAosSRE8oWGseWDdlDQnA-I8OpNqJZLTHBZrRh4XKLYf7HsUSudG7LC8bBeh7CeVMhSYmEDZEaj52t4W087knM9lR4GW6HYOXb8UYpslJq5jTzM2HnVxpXUWyokbMyBLsIVsaOc8H7bXIH8jVlQu7YQXuB2M7riXtghM6a6OHgUJcRrhkQN4FZ46GrA2BP79KbpXi0Q",
  "token_endpoint_auth_method": "tls_client_auth",
  "token_endpoint_auth_signing_alg": "PS256",
  "id_token_signed_response_alg": "PS256",
  "request_object_signing_alg": "PS256",
  "request_object_encryption_alg": "RSA-OAEP-256",
  "request_object_encryption_enc": "A128CBC-HS256"
}
```

As a result:

```
eyJraWQiOiIzODhiNjJjOTBlNzAzMDg4MjQwNTQ1ZjM2ZmNmMTRkM2Q2N2EwMTliIiwiYWxnIjoiUFMyNTYifQ.eyJ0b2tlbl9lbmRwb2ludF9hdXRoX3NpZ25pbmdfYWxnIjoiUFMyNTYiLCJyZXF1ZXN0X29iamVjdF9lbmNyeXB0aW9uX2FsZyI6IlJTQS1PQUVQLTI1NiIsImdyYW50X3R5cGVzIjpbImF1dGhvcml6YXRpb25fY29kZSIsInJlZnJlc2hfdG9rZW4iLCJjbGllbnRfY3JlZGVudGlhbHMiXSwic3ViamVjdF90eXBlIjoicGFpcndpc2UiLCJpc3MiOiI3MDdlNjA4MS04YWMyLTQzYjgtOWE5Zi1mYTc1YmVlMjJkYzkiLCJyZWRpcmVjdF91cmlzIjpbImh0dHBzOlwvXC93d3cuZ29vZ2xlLmNvbSJdLCJ0b2tlbl9lbmRwb2ludF9hdXRoX21ldGhvZCI6InRsc19jbGllbnRfYXV0aCIsInNvZnR3YXJlX3N0YXRlbWVudCI6ImV5SnJhV1FpT2lJd1lUTmxOR0poWXpWbU1UZzJPVEZsTlRjeE5HSmtOak0yWldZNFl6aGpZV0kyTW1KbFkyUTNJaXdpWVd4bklqb2lVRk15TlRZaWZRLmV5SnZjbWRmYW5kcmMxOWxibVJ3YjJsdWRDSTZJbFJQUkU4aUxDSnpiMlowZDJGeVpWOXRiMlJsSWpvaVZFVlRWQ0lzSW5OdlpuUjNZWEpsWDNKbFpHbHlaV04wWDNWeWFYTWlPbHRkTENKdmNtZGZjM1JoZEhWeklqb2lRV04wYVhabElpd2ljMjltZEhkaGNtVmZZMnhwWlc1MFgyNWhiV1VpT2lKR2IzSlVaWE4wWHpjd04yVTJNRGd4TFRoaFl6SXRORE5pT0MwNVlUbG1MV1poTnpWaVpXVXlNbVJqT1NJc0luTnZablIzWVhKbFgyTnNhV1Z1ZEY5cFpDSTZJamN3TjJVMk1EZ3hMVGhoWXpJdE5ETmlPQzA1WVRsbUxXWmhOelZpWldVeU1tUmpPU0lzSW1semN5STZJa1p2Y21kbFVtOWpheUlzSW5OdlpuUjNZWEpsWDJwM2EzTmZaVzVrY0c5cGJuUWlPaUpvZEhSd2N6cGNMMXd2YzJWeWRtbGpaUzVrYVhKbFkzUnZjbmt1YjJJdVptOXlaMlZ5YjJOckxtWnBibUZ1WTJsaGJEbzBORE5jTDJGd2FWd3ZjMjltZEhkaGNtVXRjM1JoZEdWdFpXNTBYQzgzTURkbE5qQTRNUzA0WVdNeUxUUXpZamd0T1dFNVppMW1ZVGMxWW1WbE1qSmtZemxjTDJGd2NHeHBZMkYwYVc5dVhDOXFkMnRmZFhKcElpd2ljMjltZEhkaGNtVmZhV1FpT2lJM01EZGxOakE0TVMwNFlXTXlMVFF6WWpndE9XRTVaaTFtWVRjMVltVmxNakprWXpraUxDSnZjbWRmWTI5dWRHRmpkSE1pT2x0ZExDSnZZbDl5WldkcGMzUnllVjkwYjNNaU9pSm9kSFJ3Y3pwY0wxd3ZaR2x5WldOMGIzSjVMbTlpTG1admNtZGxjbTlqYXk1bWFXNWhibU5wWVd3Nk5EUXpYQzkwYjNOY0x5SXNJbTl5WjE5cFpDSTZJalZqTkRWbU9ESmtZVGt6WWpjMU1ERXlOV00zWVdSbFlpSXNJbk52Wm5SM1lYSmxYMnh2WjI5ZmRYSnBJam9pYUhSMGNITTZYQzljTDJrdWNHOXpkR2x0Wnk1alkxd3ZhSFJvVVVOS2FGSmNMMlp5TFd4dloyOHRjM0YxWVhKbExURmpMV0pzWVdOckxuQnVaeUlzSW5OdlpuUjNZWEpsWDJwM2EzTmZjbVYyYjJ0bFpGOWxibVJ3YjJsdWRDSTZJbFJQUkU4aUxDSnpiMlowZDJGeVpWOXliMnhsY3lJNld5SkRRbEJKU1NJc0lsQkpVMUFpTENKRVFWUkJJaXdpUVVsVFVDSmRMQ0psZUhBaU9qRTFOekk0T1RJeE5qY3NJbTl5WjE5dVlXMWxJam9pUm05eVoyVlNiMk5ySWl3aWIzSm5YMnAzYTNOZmNtVjJiMnRsWkY5bGJtUndiMmx1ZENJNklsUlBSRThpTENKcFlYUWlPakUxTnpJeU9EY3pOamNzSW1wMGFTSTZJamMzTURVNU5EVTVMVGhrT0RZdE5EQXlOUzA1Wm1JeUxXSXdNakU0TkRGbE16TmpNeUo5LmMwcHJ3YWlMR0R2VVhPNEtVMXpUS1pFT05kQ013Y1dfeFpOa18zcTg1OGV6YVp0cHc0QVpGRUVJQVVOUGNoODltMy1XNFhMOTVjTlJQdGt2S2tiLUFfaFg2M2ZmeUFXdTFabEhzempQRFdnNXRBSFEtZm9zeE1CZmNoRkpIdE14cDBGVkhOWTVCSHBNV0R0VkFvc1NSRThvV0dzZVdEZGxEUW5BLUk4T3BOcUpaTFRIQlpyUmg0WEtMWWY3SHNVU3VkRzdMQzhiQmVoN0NlVk1oU1ltRURaRWFqNTJ0NFcwODdrbk05bFI0R1c2SFlPWGI4VVlwc2xKcTVqVHpNMkhuVnhwWFVXeW9rYk15QkxzSVZzYU9jOEg3YlhJSDhqVmxRdTdZUVh1QjJNN3JpWHRnaE02YTZPSGdVSmNScmhrUU40Rlo0NkdyQTJCUDc5S2JwWGkwUSIsInNjb3BlIjoib3BlbmlkIGFjY291bnRzIHBheW1lbnRzIGZ1bmRzY29uZmlybWF0aW9ucyIsInJlcXVlc3Rfb2JqZWN0X3NpZ25pbmdfYWxnIjoiUFMyNTYiLCJleHAiOjE1NzI3ODg5NzIsInJlcXVlc3Rfb2JqZWN0X2VuY3J5cHRpb25fZW5jIjoiQTEyOENCQy1IUzI1NiIsImlhdCI6MTU3Mjc4ODY3MiwianRpIjoiODM5OGI5NzgtMTI3YS00ZjFlLWFiNzUtNmE0NzY2MTA3YTYzIiwicmVzcG9uc2VfdHlwZXMiOlsiY29kZSBpZF90b2tlbiJdLCJpZF90b2tlbl9zaWduZWRfcmVzcG9uc2VfYWxnIjoiUFMyNTYifQ.Uy5sOXSXCrkJ_n_wjU6YkdAYxZGPvJNncTGaUdskA7TlGOtUi1_n4VJvxycB_-1j43dagawbS5IQhA3dG_7yyVV4PguZdmnB31N5VOLnNv0QMevFdXLT_bIU904LYZSyBbPYAQZvtiAx8Fq4vPXdhbvpxt5T2quw5LHSlb3HStePb6S7t1KmZ3-UBecia5ZYDQ0r1XeJLwuGGZumFWZ6TmAc-A3MymVglf9Ks-w16-_ZB0QY0K1CpcKJHaM5JS70_wgcpwLy3ehE8BJlDzN-QOsTGQuf-E0c78-mA_xsHZFtG1U5IYStenOC995rxa7nnqh7PSn7noUqKkGCaOj6vg
```

**Postman: ** You can find this request in the ForgeRock Postman collection under `Dynamic Registration` > `Onboarding your TPP` > `Generate registration JWT`. The link from the online documentation directly to this request is https://postman.ob.forgerock.financial/?version=latest#3cdba696-f950-4066-8c7d-084d99dd6d1e

A lot of efforts to get this JWS, now it's time to send it to the bank.

#### Send the registration request

Once you got the registration request in the right format and ready, sending the registration request is a piece of cake.
All you need to do is a POST to the bank dynamic registration request:

```
POST /open-banking/register/ HTTP/1.1
Host: matls.as.aspsp.ob.forgerock.financial
Content-Type: application/jwt

eyJraWQiOiIzODhiNjJjOTBlNzAzMDg4MjQwNTQ1ZjM2ZmNmMTRkM2Q2N2EwMTliIiwiYWxnIjoiUFMyNTYifQ.eyJ0b2tlbl9lbmRwb2ludF9hdXRoX3NpZ25pbmdfYWxnIjoiUFMyNTYiLCJyZXF1ZXN0X29iamVjdF9lbmNyeXB0aW9uX2FsZyI6IlJTQS1PQUVQLTI1NiIsImdyYW50X3R5cGVzIjpbImF1dGhvcml6YXRpb25fY29kZSIsInJlZnJlc2hfdG9rZW4iLCJjbGllbnRfY3JlZGVudGlhbHMiXSwic3ViamVjdF90eXBlIjoicGFpcndpc2UiLCJpc3MiOiI3MDdlNjA4MS04YWMyLTQzYjgtOWE5Zi1mYTc1YmVlMjJkYzkiLCJyZWRpcmVjdF91cmlzIjpbImh0dHBzOlwvXC93d3cuZ29vZ2xlLmNvbSJdLCJ0b2tlbl9lbmRwb2ludF9hdXRoX21ldGhvZCI6InRsc19jbGllbnRfYXV0aCIsInNvZnR3YXJlX3N0YXRlbWVudCI6ImV5SnJhV1FpT2lJd1lUTmxOR0poWXpWbU1UZzJPVEZsTlRjeE5HSmtOak0yWldZNFl6aGpZV0kyTW1KbFkyUTNJaXdpWVd4bklqb2lVRk15TlRZaWZRLmV5SnZjbWRmYW5kcmMxOWxibVJ3YjJsdWRDSTZJbFJQUkU4aUxDSnpiMlowZDJGeVpWOXRiMlJsSWpvaVZFVlRWQ0lzSW5OdlpuUjNZWEpsWDNKbFpHbHlaV04wWDNWeWFYTWlPbHRkTENKdmNtZGZjM1JoZEhWeklqb2lRV04wYVhabElpd2ljMjltZEhkaGNtVmZZMnhwWlc1MFgyNWhiV1VpT2lKR2IzSlVaWE4wWHpjd04yVTJNRGd4TFRoaFl6SXRORE5pT0MwNVlUbG1MV1poTnpWaVpXVXlNbVJqT1NJc0luTnZablIzWVhKbFgyTnNhV1Z1ZEY5cFpDSTZJamN3TjJVMk1EZ3hMVGhoWXpJdE5ETmlPQzA1WVRsbUxXWmhOelZpWldVeU1tUmpPU0lzSW1semN5STZJa1p2Y21kbFVtOWpheUlzSW5OdlpuUjNZWEpsWDJwM2EzTmZaVzVrY0c5cGJuUWlPaUpvZEhSd2N6cGNMMXd2YzJWeWRtbGpaUzVrYVhKbFkzUnZjbmt1YjJJdVptOXlaMlZ5YjJOckxtWnBibUZ1WTJsaGJEbzBORE5jTDJGd2FWd3ZjMjltZEhkaGNtVXRjM1JoZEdWdFpXNTBYQzgzTURkbE5qQTRNUzA0WVdNeUxUUXpZamd0T1dFNVppMW1ZVGMxWW1WbE1qSmtZemxjTDJGd2NHeHBZMkYwYVc5dVhDOXFkMnRmZFhKcElpd2ljMjltZEhkaGNtVmZhV1FpT2lJM01EZGxOakE0TVMwNFlXTXlMVFF6WWpndE9XRTVaaTFtWVRjMVltVmxNakprWXpraUxDSnZjbWRmWTI5dWRHRmpkSE1pT2x0ZExDSnZZbDl5WldkcGMzUnllVjkwYjNNaU9pSm9kSFJ3Y3pwY0wxd3ZaR2x5WldOMGIzSjVMbTlpTG1admNtZGxjbTlqYXk1bWFXNWhibU5wWVd3Nk5EUXpYQzkwYjNOY0x5SXNJbTl5WjE5cFpDSTZJalZqTkRWbU9ESmtZVGt6WWpjMU1ERXlOV00zWVdSbFlpSXNJbk52Wm5SM1lYSmxYMnh2WjI5ZmRYSnBJam9pYUhSMGNITTZYQzljTDJrdWNHOXpkR2x0Wnk1alkxd3ZhSFJvVVVOS2FGSmNMMlp5TFd4dloyOHRjM0YxWVhKbExURmpMV0pzWVdOckxuQnVaeUlzSW5OdlpuUjNZWEpsWDJwM2EzTmZjbVYyYjJ0bFpGOWxibVJ3YjJsdWRDSTZJbFJQUkU4aUxDSnpiMlowZDJGeVpWOXliMnhsY3lJNld5SkRRbEJKU1NJc0lsQkpVMUFpTENKRVFWUkJJaXdpUVVsVFVDSmRMQ0psZUhBaU9qRTFOekk0T1RJeE5qY3NJbTl5WjE5dVlXMWxJam9pUm05eVoyVlNiMk5ySWl3aWIzSm5YMnAzYTNOZmNtVjJiMnRsWkY5bGJtUndiMmx1ZENJNklsUlBSRThpTENKcFlYUWlPakUxTnpJeU9EY3pOamNzSW1wMGFTSTZJamMzTURVNU5EVTVMVGhrT0RZdE5EQXlOUzA1Wm1JeUxXSXdNakU0TkRGbE16TmpNeUo5LmMwcHJ3YWlMR0R2VVhPNEtVMXpUS1pFT05kQ013Y1dfeFpOa18zcTg1OGV6YVp0cHc0QVpGRUVJQVVOUGNoODltMy1XNFhMOTVjTlJQdGt2S2tiLUFfaFg2M2ZmeUFXdTFabEhzempQRFdnNXRBSFEtZm9zeE1CZmNoRkpIdE14cDBGVkhOWTVCSHBNV0R0VkFvc1NSRThvV0dzZVdEZGxEUW5BLUk4T3BOcUpaTFRIQlpyUmg0WEtMWWY3SHNVU3VkRzdMQzhiQmVoN0NlVk1oU1ltRURaRWFqNTJ0NFcwODdrbk05bFI0R1c2SFlPWGI4VVlwc2xKcTVqVHpNMkhuVnhwWFVXeW9rYk15QkxzSVZzYU9jOEg3YlhJSDhqVmxRdTdZUVh1QjJNN3JpWHRnaE02YTZPSGdVSmNScmhrUU40Rlo0NkdyQTJCUDc5S2JwWGkwUSIsInNjb3BlIjoib3BlbmlkIGFjY291bnRzIHBheW1lbnRzIGZ1bmRzY29uZmlybWF0aW9ucyIsInJlcXVlc3Rfb2JqZWN0X3NpZ25pbmdfYWxnIjoiUFMyNTYiLCJleHAiOjE1NzI3ODc4NTIsInJlcXVlc3Rfb2JqZWN0X2VuY3J5cHRpb25fZW5jIjoiQTEyOENCQy1IUzI1NiIsImlhdCI6MTU3Mjc4NzU2MiwianRpIjoiZGE0NDViZjQtOGY3My00MzRkLWFkMTctMWY0OGMyZTU1ZmY0IiwicmVzcG9uc2VfdHlwZXMiOlsiY29kZSBpZF90b2tlbiJdLCJpZF90b2tlbl9zaWduZWRfcmVzcG9uc2VfYWxnIjoiUFMyNTYifQ.PfHlB1k2wIsDiv9GgNX0nUHlZh5aVd0moJeMrL6mTZrrZNdGWvt1Vq0DKabImz51_maTMi3WIwkS7Y5CBGovozwig4MlTCiOO8nVP9UHK9JW28IEZqSUEU5b3OO26hdmFEJftTc_iyHiCxvq72zyhoJ0aYpgIkASyoBvkjP3X9Xf0scBYGxuo8S1cqVfS65AVHlFpJlug7QjGA1jGyAQVAAUDZ20W47gqXauVMqqXsOdHh_KQ8n0-hjVyVdPEj5JDS3-Rup5OC7d7EDRRoRoKfmyd5_9W2gSQw48D71cHlwlhZ3jmh4Ga5F30wKjeXd-Yn5_5F8my1z_-xstkS-d1A
  ```

As a result:

```
{
    "scopes": [
        "openid",
        "payments",
        "fundsconfirmations",
        "accounts"
    ],
    "scope": "openid payments fundsconfirmations accounts",
    "client_id": "8c40fcdb-a9e4-453e-8a54-2b733307bb21",
    "redirect_uris": [
        "https://www.google.com"
    ],
    "response_types": [
        "code id_token"
    ],
    "grant_types": [
        "authorization_code",
        "refresh_token",
        "client_credentials"
    ],
    "application_type": "web",
    "client_name": "ForTest_4691fc6d-16a2-4fec-9e74-25bccaed31e1",
    "jwks_uri": "https://service.directory.ob.forgerock.financial:443/api/software-statement/4691fc6d-16a2-4fec-9e74-25bccaed31e1/application/jwk_uri",
    "subject_type": "pairwise",
    "id_token_signed_response_alg": "PS256",
    "id_token_encrypted_response_alg": "RSA-OAEP-256",
    "id_token_encrypted_response_enc": "A128CBC-HS256",
    "userinfo_signed_response_alg": "",
    "userinfo_encrypted_response_alg": "",
    "userinfo_encrypted_response_enc": "",
    "request_object_signing_alg": "PS256",
    "request_object_encryption_alg": "RSA-OAEP-256",
    "request_object_encryption_enc": "A128CBC-HS256",
    "token_endpoint_auth_method": "tls_client_auth",
    "token_endpoint_auth_signing_alg": "PS256",
    "default_max_age": "1",
    "software_statement": "eyJraWQiOiIwYTNlNGJhYzVmMTg2OTFlNTcxNGJkNjM2ZWY4YzhjYWI2MmJlY2Q3IiwiYWxnIjoiUFMyNTYifQ.eyJvcmdfandrc19lbmRwb2ludCI6IlRPRE8iLCJzb2Z0d2FyZV9tb2RlIjoiVEVTVCIsInNvZnR3YXJlX3JlZGlyZWN0X3VyaXMiOltdLCJvcmdfc3RhdHVzIjoiQWN0aXZlIiwic29mdHdhcmVfY2xpZW50X25hbWUiOiJGb3JUZXN0XzQ2OTFmYzZkLTE2YTItNGZlYy05ZTc0LTI1YmNjYWVkMzFlMSIsInNvZnR3YXJlX2NsaWVudF9pZCI6IjQ2OTFmYzZkLTE2YTItNGZlYy05ZTc0LTI1YmNjYWVkMzFlMSIsImlzcyI6IkZvcmdlUm9jayIsInNvZnR3YXJlX2p3a3NfZW5kcG9pbnQiOiJodHRwczpcL1wvc2VydmljZS5kaXJlY3Rvcnkub2IuZm9yZ2Vyb2NrLmZpbmFuY2lhbDo0NDNcL2FwaVwvc29mdHdhcmUtc3RhdGVtZW50XC80NjkxZmM2ZC0xNmEyLTRmZWMtOWU3NC0yNWJjY2FlZDMxZTFcL2FwcGxpY2F0aW9uXC9qd2tfdXJpIiwic29mdHdhcmVfaWQiOiI0NjkxZmM2ZC0xNmEyLTRmZWMtOWU3NC0yNWJjY2FlZDMxZTEiLCJvcmdfY29udGFjdHMiOltdLCJvYl9yZWdpc3RyeV90b3MiOiJodHRwczpcL1wvZGlyZWN0b3J5Lm9iLmZvcmdlcm9jay5maW5hbmNpYWw6NDQzXC90b3NcLyIsIm9yZ19pZCI6IjVjNDVmODJkYTkzYjc1MDEyNWM3YWRlYiIsInNvZnR3YXJlX2xvZ29fdXJpIjoiaHR0cHM6XC9cL2kucG9zdGltZy5jY1wvaHRoUUNKaFJcL2ZyLWxvZ28tc3F1YXJlLTFjLWJsYWNrLnBuZyIsInNvZnR3YXJlX2p3a3NfcmV2b2tlZF9lbmRwb2ludCI6IlRPRE8iLCJzb2Z0d2FyZV9yb2xlcyI6WyJQSVNQIiwiREFUQSIsIkNCUElJIiwiQUlTUCJdLCJleHAiOjE1NzMzOTQwMzQsIm9yZ19uYW1lIjoiRm9yZ2VSb2NrIiwib3JnX2p3a3NfcmV2b2tlZF9lbmRwb2ludCI6IlRPRE8iLCJpYXQiOjE1NzI3ODkyMzQsImp0aSI6ImQ5ZjhkY2FiLTNjNGUtNDJiZi05OTc2LWY2YmExODA4YzgwZSJ9.ZimU5CmeEZNe-ZD0zF3xXlPhij0toKpvL0ld8enjGmg056nX8RjZcdsnihHmZgCO-ZDg9WQucb71_bb4SsrDENQ1R6vgTnXpp8aWtfQFG3JfjiP46Du-oyvQqZ7VWbX7V8hX-DDVLS59j-sT1aetnzmneKpIGbuuLGW0xhvstx-pDRKnpnArXz7YJAT0mFUsngPwKX75DdQ4HOObFb9gAHgYLB4tvnhXrHl0e3tyCW5etGOIH1Co70EHtBHhkOPWpg-z4ZAD9Ch-WFgMiBoxNGDSbNe_ZsFffw55rePWPnONQF4HC727Qs_pIrAuAQHuHSAJ8fuOy16vhg2CLjpE3Q",
    "tls_client_auth_subject_dn": "2.5.4.97=#131e50534447422d356334356638326461393362373530313235633761646562,C=UK,ST=Avon,L=Bristol,O=ForgeRock,OU=5c45f82da93b750125c7adeb,CN=4691fc6d-16a2-4fec-9e74-25bccaed31e1",
    "registration_access_token": "eyJ0eXAiOiJKV1QiLCJ6aXAiOiJOT05FIiwia2lkIjoiRTE5N1kzMVFLT05mSk42aTdrQlkyMzFneUFvPSIsImFsZyI6IkVTMjU2In0.eyJzdWIiOiI4YzQwZmNkYi1hOWU0LTQ1M2UtOGE1NC0yYjczMzMwN2JiMjEiLCJjdHMiOiJPQVVUSDJfR1JBTlRfU0VUIiwiYXVkaXRUcmFja2luZ0lkIjoiOTgyNWY3NjctOGVmNC00Y2FlLTg3OGEtY2U3NmQ3NGJkZDQ2LTk5MDg2IiwiaXNzIjoiaHR0cHM6Ly9hcy5hc3BzcC5vYi5mb3JnZXJvY2suZmluYW5jaWFsL29hdXRoMiIsInRva2VuTmFtZSI6ImFjY2Vzc190b2tlbiIsInRva2VuX3R5cGUiOiJCZWFyZXIiLCJhdXRoR3JhbnRJZCI6IldRYk1rT2lQR0ZJcjFoNFR0WWFETlhYSUp1Yy5LcE13c3dGX05wSjlSSHROMlk0YzluV21QSkUiLCJhdWQiOiI4YzQwZmNkYi1hOWU0LTQ1M2UtOGE1NC0yYjczMzMwN2JiMjEiLCJuYmYiOjE1NzI3ODkyNDEsInNjb3BlIjpbXSwiYXV0aF90aW1lIjoxNTcyNzg5MjQxLCJyZWFsbSI6Ii9vcGVuYmFua2luZyIsImV4cCI6MTU3Mjg3NTY0MSwiaWF0IjoxNTcyNzg5MjQxLCJleHBpcmVzX2luIjo4NjQwMCwianRpIjoiV1FiTWtPaVBHRklyMWg0VHRZYUROWFhJSnVjLnlrR0Vjdk5IbzRpaF9SaWM2VFR2b3M1cE5nSSJ9.i0Atv2DWs03cSOcbyP5-9qJCYx8Y9x0tvk_NnuNLsDjOHoWfhYb3G58z4OMb5sNw9zdIndJrZkiBLjZEmB0WIA",
    "registration_client_uri": "https://as.aspsp.ob.forgerock.financial/oauth2/register?client_id=8c40fcdb-a9e4-453e-8a54-2b733307bb21"
}
```

**Postman: ** You can find this request in the ForgeRock Postman collection under `Dynamic Registration` > `Onboarding your TPP` > `Dynamic Registration`. The link from the online documentation directly to this request is https://postman.ob.forgerock.financial/?version=latest#5db4a2c1-40a7-4caa-ab4e-b934aade8411

Important things to note is that, even if you requested some specific values for your OIDC client like `"token_endpoint_auth_method": "tls_client_auth"`, the specification says that the bank can override them if they want to. A bit annoying you may think but that's part of the game. It's always best to verify that it's not been the case. From a code point of view, store the response of the registration but not necessarily the request you did.

The response is standardise, we will only cover the main one you should know about:

- `client_id`: The one to not miss and store preciously somewhere. This client ID is the proof that you successfully registered and can use the bank APIs. The bank is going to request it in different places of the flow, it is best to keep it accessible easily. This client ID is also refer as the OIDC client ID. 
- `redirect_uris`: remember that this is the list of redirect URIs that the bank will authorise to redirect to. If you need to change it, you need to do a PUT again.
- `registration_access_token`: don't miss that one, it's important to store it somewhere precious too. It's the only time it will be returned to you so best not loosing it. It's mandatory for PUT/DELETE/GET your OIDC client information via the dynamic registration endpoint.  


## Conclusion

Congratulations! Quite a lot of efforts just to on-board into the bank. The real value of this complexity is to be full programmable. 
We will only on-board with ForgeRock mock bank for this tutorial but in the reality, you would on-board with all the banks following a similar pattern. One day, we may see some libraries dedicated to on-boarding.
Tips: It's probably a good idea to abstract the on-boarding to a dedicated micro-services, to isolate this complexity from the rest of your system.
Now that you are on-board, we can see how to consume the APIs but first, we need to talk about the flow from a theory aspect, especially the security side of it.

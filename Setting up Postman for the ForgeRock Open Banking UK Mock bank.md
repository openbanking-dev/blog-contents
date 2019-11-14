# Setting up Postman for the ForgeRock Open Banking UK Mock bank

ForgeRock offers you a Postman collections to help you use their sandbox.
Postman is a collaboration platform for API development. It allows you to execute REST request simplify.
The advantages of Postman is to be code agnostic and will therefore be a good learning tool before implementing Open Banking UK in your favourite language.
Postman is self-sufficient, it means you don't have to implement any code. You can explore every flow from an end to end just by using Postman.
In this article, we will see how to setup Postman for the ForgeRock mock bank.

## Download Postman

Postman is an application that you install on your OS. You can download it here: https://www.getpostman.com/downloads/


## Download the ForgeRock Open Banking UK collections

Postman has the concept of collections. It's a set of pre-configured REST requests, ready to play.
ForgeRock offers you a collection dedicated to their mock bank, that you will be able to use during the tutorial.
The collection is available here:
https://postman.ob.forgerock.financial
This web-site is a Postman service, that allows ForgeRock to publish their collection online and make it easier for you to explore it and download it.
On the top right, you will see a button 'Run in Postman'. This will open your Postman application and load the collection into the application.
It also best noticing that a collection comes with an environment file. You can see the collection as the template of the REST requests and the environment files as a set of environment variables that complete the template to point to the ForgeRock mock bank.
The environment file will be download at the same time than the collection. The name of the environment for the ForgeRock mock bank is 'OBRI ob (Generated)'.
If everything went well, you should have in your Postman application, a collection called 'ForgeRock OBRI Sample (Generated)' and in the top right of Postman, an environment `OBRI ob (Generated)`.
*You may need to select the environment*

## Testing the collection

Let's do a quick test to make sure you got the collection up and running.
If you click on the collection, you will see it's organised into  different folders. The first one is called 'Setup your environment'. 
Open the request 'Discovery' > 'RS discovery - OB Official API'.
You will see that the URL is `{{RS_MATLS}}/open-banking/discovery` which contains an environment variable `RS_MATLS`. If you selected correctly the environment 'OBRI ob', this variable will be replace by the right value when you click 'Send'.
If everything is okay, you would have received a JSON response with a 200.

## Test your MATLS setup before setting up a client certificate

Go back to the collection and test the request: `Setup your environment` > `Test MATLS` > `RS test MATLS`.

You should get the following:

```
GET /open-banking/mtlsTest HTTP/1.1
Host: matls.rs.aspsp.ob.forgerock.financial
```

```
{
    "issuerId": "anonymous",
    "authorities": [
        "ROLE_ANONYMOUS"
    ]
}
```

As you can see, Postman hasn't send your client certificate and therefore, the bank didn't manage to authenticate you. You are an anonymous user from the point of view of the bank.

## Setup your client certificate in Postman

The Open Banking APIs are protected by MATLS `Mutual Authentication TLS`.
You probably used to the httpS, which basically means the server is presenting to you a certificate that you can verify.
MATLS is about doing the same but both side, meaning that you will also present a certificate to the server. This certificate is called client certificate.

Postman supports providing client certificates but you need to configure it accordingly.
1. Open `Preferences` > `Certificates`.
2. In the `Client Certificates` section, click on `Add Certificate`.
3. Complete the form:
	4. `Host` : `https://*.ob.forgerock.financial`
	5. `CRT file`: upload your transport key PEM file (.pem)
	6. `KEY file`: upload your transport key private key file (.key)
	7. Ignore `PFX file`
	8. Ignore `Passphrase`
	9. Click `add`


Your client certificate should be setup and send to every ForgeRock mock bank endpoint.

## Test your MATLS setup after setting up a client certificate

Go back to the collection and test the request: `Setup your environment` > `Test MATLS` > `RS test MATLS`.

You should get the following:

```
GET /open-banking/mtlsTest HTTP/1.1
Host: matls.rs.aspsp.ob.forgerock.financial
```

```
{
    "issuerId": "7d43c1e5-2b14-40b5-90f3-362a73953f8c",
    "authorities": [
        "UNREGISTERED_TPP"
    ]
}
```

As you can see, now Postman has send your client certificate. The bank was able to authenticate you as `7d43c1e5-2b14-40b5-90f3-362a73953f8c` which for the moment, is not on-board yet. `UNREGISTERED_TPP` meaning that the bank realises that you are part of the Open Banking community as a TPP, but you are not registered yet with this bank.

## Conclusion

You should now be ready to follow the rest of the tutorial and use Postman on the side to reproduce all the flows!




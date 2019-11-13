# Learn how to use Open Banking UK standard

## Introduction

Open Banking is a set of APIs, to consume bank data, but not any  kind of data: the user data. As a developer, you will be able to offer financial services to your users, by directly connecting to their bank and accessing their financial data.

It's really a game changer in the financial industry, as developers have now a new source of innovation: a simple set of secured APIs, allowing them to the bank data of their users without been a bank themselves.

As a developer, I really like Open Banking for what it represents: a step forwards Open APIs. The fact that the financial industry is showing the way, is for me a strong message. We are now heading to an API Colorado, where each industry sector will offers secured APIs. An unlimited source of innovation, empowered to us, developers. Is that not ideal?

Before talking about Open APIs, let's talk first with what we have today in our plate: Open Banking.

By a set of articles, we are going to see how works Open Banking in the UK. 
Security is obviously in the center of this standard, allowing you to access the user financial data, with his consent, without any security risk.
Open Banking UK may sounds complex at the first approach but hopefully you will see after those articles that it's not. It's just a lot to learn in one go, and this is what those articles will help you with: We are going to learn step by step, on how to use Open Banking UK.

## Expected audience

You are a developer and you want to learn more about the Open Banking UK APIs. You heard about the concept and looking for getting your hands dirty, as it's really the only way to learns things.
You are working for a bank and your company is following the trend of Open Banking. You want to understand a bit more what you are supposed to provide as APIs and how the full flow works from an end to end. 
We will take the angle of the developer using the APIs for this set of articles.

## Glossary

We will try to not use the specification vocabularies, as it doesn't make things easy at the beginning.
Although, it is best we do introduce some of it now, in case you end up checking the specification and want to do the correlation with those tutorials:
* TPP: Third Party Provider. It's you, the developer
* ASPSP: Account Servicing Payment Service Provider, it's the bank!
* AS: Authorisation Server, it's the bank as well, just the authorisation bit of it.
* RS: Resource Server, it's also the bank! It's a generic term to designate the party storing the resources.
* PSU: Payment Services User. It's the bank customer that is owning the data. It's the user basically

If you are interested to have a lot more glossary, the one from the standard is pretty good: https://www.openbanking.org.uk/about-us/glossary/

You understand a bit better now why we will try to not use those terms in this tutorial!

## Le menu

### Starters

#### Overview of the Open Banking APIs.

It's good to start by selling you the APIs. Making you dream a bit so you stick with me for this set of articles!
We are going to do an over view of the APIs, to see what they have to offer to you today.
If you are just looking to see the potential of Open Banking, as a source of inspiration, it's the article for you.

#### Open Banking trusting model and how to register yourself

The first things to start with, is understanding the trusting model of Open Banking. By trusting model, I mean how all those different parties are going to trust each others.
Because yes, if you thought you could just show up and make payment, access account data, without identifying you to the banks, I am sorry to disappoint you!
It's not big deal, it was expected if you think of the security aspect. This article will be about how the circle of trust in Open Banking works and how you can join the community.


### The mains

#### ForgeRock Open Banking Reference Implementation

In order to discover the APIs, we will use the model bank offered by ForgeRock. It's a sandbox environment that everyone can access and play with.
It is security conformant mock bank that provides all the Open Banking UK APIs available today. It would be a perfect for us for our Open Banking UK learning journey.

#### On-boarding your application to the bank

We will then talk about how to on-board with a bank. Even if you are part of the Open Banking community from the previous article now, you still need to do this on-boarding step with each bank.
Basically at this point, the bank needs you to have an identity in their system, so they can verify you are authorised to use their APIs in the future. 
In Open Banking, it's called on-boarding. You basically need to register your application with them, simple as that.

#### The security flow
Once on-board done, you are read to start using the APIs! It's now a good time to start talking about the security around them.
Security is no doubt what would slow you down in your APIs consumption adventure. 
The usual API token model would not be good enough in Open Banking. It is due that it's not just you and the bank, there is also the user. That's like a bummer for you at this point. It's what makes those APIs so attractive in one way, but it's also what make them more complex to consume from a security point of view.
The next article would be around understanding this security. 

#### Get the user transactions using the Open Banking APIs

Once you understand the reason for all this security and how it works in theory, it's time to put that in practice.
We are going to access the account APIs of the user, in particular the transactions APIs. It's one of the most popular API enabled by Open Banking and it's maybe why you are here right now!

#### Making a payment using Open Banking (soon)

Another appealing APIs are the payments APIs. We are going to see the different payments available to you, like international payment, and how they works.
It's slightly different than the accounts APIs, just because the idea is not to access a resource but more to trigger an action, in our case a payment.

### The deserts

#### Event notifications (soon)

One thing you will notice once you start to look at payments more seriously, is that you need a way to be notified about the payment completion.
Pulling VS Pushing, here is the debate. Today, the Open Banking APIs offers both, meaning you can either:
* Pull the bank about a specific payment and ask them the status of it
* Receive a notification from the bank, to notify you that the payment has a new status

Pull is probably the most commonly implemented by the bank at this stage, but I am sure as a developer like me, you are already not liking it. Pulling is more work for the developer consuming the API, it's better to push that work on the bank side isn't it?
We are going to see the event notification APIs, which when available by the bank, allows you to be notified by your payments.


#### Confirmation of funds (soon)

The last APIs defined by Open Banking, is called confirmation of funds. Not my favourite APIs, probably why I am putting at last.
Although it does offer you a very specific functionality, which may be spot on for your business.
Those APIs are about verifying that the user has enough funding.
You can do it during a payment, which is handy, or you can just do it any point of time.



## Conclusion

Still here? Time for your Open Banking dinner. Bon appétit!






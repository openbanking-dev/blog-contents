# Open Banking UK trusting model

*This article is part of a set of tutorial about Open Banking UK. If you are interested to learn about Open Banking UK, you may want to check-out first our intro article.*

For security reason, you can imagine you would need to be authorise before consuming those APIs, especially payments.

You probably used to the model where you register and get an API key. It's how works the google APIs for example.

For Open Banking, things have been designed differently. What we want to avoid really, is having each developer registering to each bank with a complex process, having to show their passport, their financial licenses, etc.
That would be a major bummer and would make your dream to integrate with 100 banks a nightmare.

The way it works is based on certificates and JWTs, but I come back on that. First, let's see how it works in general.

There is two parallel trusting model:

* A central one, based on what we call an Open Banking UK directory.
* A certificate based model called EIDAS


## Open Banking directory

Open Banking directory is an entity in charge of registering every parties involved in the Open Banking system, to build up a circle of trust.
It's the model adopted in the United-Kingdom at the moment.
Easiest model so far in my opinion, as they do the heavy lifting of verifying the identity of each parties, providing certificates for each of them etc.
You can find the directory here: https://directory.openbanking.org.uk/s/login/

The idea is that you register with the Open Banking directory, which is trust by every participants including banks. Instead of having every participants verifying your company identity and accreditation, you do it once with the directory and the other participant will trust you by delegation.
The directory duty is to provide you the necessary documents, which you will present to each bank, that will allows them to trust you.

### How it works

The enrolling process is not a simple journey, I prefer to warn you in advance. It requires a certain commitment from your part and would not recommend it if you just fancy playing with the APIs. For that, I would recommend another way that I will talk in the next section.

Once you are enrolled, you can register your application, which is named software statements.
For each applications that needs to use Open Banking APIs, you will register a new software statement.
Once you got your software statement, you will be able to get certificates and a Software Statement Assertion (SSA).
That's basically all you need to then go to each bank and register with them.

### ForgeRock Open Banking directory

For this set of tutorials, we will use the ForgeRock sandbox. In addition to accepting the official Open Banking directory, ForgeRock also provide their own version of the directory.
The reason for it is to simplify the enrolment to their sandbox.
The official Open Banking directory, even for accessing sandboxes, will do all the background verifications on you and your company. At this point, you probably don't even want to get your company involved.
The ForgeRock directory removes this background check and provides you a direct access to their directory. The ForgeRock directory is only trust by the ForgeRock mock bank. If you want to use the sandboxes of the other UK banks, you would have to go through the Open Banking directory.

You can find the ForgeRock Open Banking directory here: https://directory.ob.forgerock.financial/



## EIDAS
The EIDAS trusting model is based on the principal that each party get issued a certificate, called EIDAS certificate.
Inside those certificates, you find information about the financial institution, in particular what permissions is granted to it.
By presenting your certificate to banks, you get granted access to the APIs.
It is the model adopted by the rest of Europe but even if the Open Banking directory is present in the UK, UK banks have been asked to also accept EIDAS certificate to facilitate access to non-UK developers.

In the paper, it's nice, in the reality, probably not that much.

Let's talk about the few challenges/concern you would have to face.

### Getting your EIDAS certificate

Getting your EIDAS certificate is the first challenge. You would need to find a CA that is able to issue EIDAS certificate, prove your identity etc, pay a bit of money, and you get your certificate. There is a few in Europe, like in Portugal, that would do that.
Not an easy things to do and take a few weeks to get. For now, we will imagine that you managed to get your EIDAS certificates, which is a good step already. Let's go to the next concern.

### EIDAS standard implementation varies

Your EIDAS certificates is issued by a CA, and even if it is standardise, differences exist between those EIDAS certificates issued. It may have a period of adaptation before every banks is accepting every format of EIDAS issued in Europe.
Again, let's imagine that every banks accepts EIDAS nicely.

### Using your precious EIDAS certificate

The idea now is that you use your EIDAS certificate to consume the Open Banking APIs. You may be a bit reluctant to use your certificate everywhere and all the time, after all those efforts to get it. You wouldn't want someone to steal it from you, it's kind of your treasure in a way.

Again, let's put a side that concern.

### Revocation
How fast a CA will revoke the EIDAS certificate if the owner of it becomes compromised?
It's still in debate and I haven't the answer to that. Although it's hard to image it would be instantaneous. It's likely it will take weeks, which is quite concerning. Especially if unfortunately someone stole your EIDAS and start making fraudulent usage of it. As it's your responsibility and your insurance taking the hit, you may not be happy to see the attacker using your certificates for days.

### Information inside the certificate

An EIDAS is the only thing you need to get access to the APIs.
At some point during the Open Banking flow, the user will get a consent screen. It will present to the user who you are and what access you are requesting.
We are used to this kind of consent screen as users, so I am sure you know what I am talking about.
In those consent screen, we are used to see the application name  and logo requesting access to our data.
Coming back to EIDAS, if that's the only thing we provide to the bank, then the application name and logo should be in there.
That's where the trouble start.

An EIDAS is basically this:

```
Extension
Identifier:	1.3.6.1.5.5.7.1.3
Value:	30 81 C3 30 08 06 06 04 00 8E 46 01 01 30 09 06 07 04 00 8E 46 01 06 03 30 09 06 07 04 00 8B EC 49 01 02 30 81 A0 06 06 04 00 81 98 27 02 30 81 95 30 6A 30 29 06 07 04 00 81 98 27 01 04 0C 1E 43 61 72 64 20 42 61 73 65 64 20 50 61 79 6D 65 6E 74 20 49 6E 73 74 72 75 6D 65 6E 74 73 30 1E 06 07 04 00 81 98 27 01 03 0C 13 41 63 63 6F 75 6E 74 20 49 6E 66 6F 72 6D 61 74 69 6F 6E 30 1D 06 07 04 00 81 98 27 01 02 0C 12 50 61 79 6D 65 6E 74 20 49 6E 69 74 69 61 74 69 6F 6E 0C 1D 46 6F 72 67 65 52 6F 63 6B 20 46 69 6E 61 6E 63 69 61 6C 20 41 75 74 68 6F 72 69 74 79 0C 08 46 52 2D 41 41 41 41 41
Critical:	No
#SEQUENCE (4 elem)
#  SEQUENCE (1 elem)
#    OBJECT IDENTIFIER 0.4.0.1862.1.1
#  SEQUENCE (1 elem)
#    OBJECT IDENTIFIER 0.4.0.1862.1.6.3
#  SEQUENCE (1 elem)
#    OBJECT IDENTIFIER 0.4.0.194121.1.2
#  SEQUENCE (2 elem)
#    OBJECT IDENTIFIER 0.4.0.19495.2 
#    SEQUENCE (3 elem)
#      SEQUENCE (3 elem)
#        SEQUENCE (2 elem)
#          OBJECT IDENTIFIER 0.4.0.19495.1.4
#          UTF8String Card Based Payment Instruments
#        SEQUENCE (2 elem)
#          OBJECT IDENTIFIER 0.4.0.19495.1.3
#          UTF8String Account Information
#        SEQUENCE (2 elem)
#          OBJECT IDENTIFIER 0.4.0.19495.1.2
#          UTF8String Payment Initiation
#      UTF8String ForgeRock Financial Authority
#      UTF8String FR-AAAAA
```

The first thing we can notice is that there isn't a logo in there. That is a -1 for the user experience.
Then what about the application name, which really is essential.
As the EIDAS certificate is issued to your organisation, there isn't the concept of software statement there.
Therefore banks can't display the application name but instead the organisation that owns it.
Let me take an example: Imagine google were to become a consumer of the Open Banking APIs and wanted to offer payments in google map. They would need an EIDAS certificate, that they are likely to register under their official name 'Alphabet' and use it across all their applications.
That would mean when you do a payment via a google application, the user would be redirected to his bank and ask to pay 'Alphabet'. This is not good, and that's considering we are lucky enough that the CA that issued the EIDAS, didn't put a non-friendly name in the certificate, like AlphabetLTD.

An EIDAS is technically working, just not good enough to be considered as a good user experience. As a developer, I think I prefer going for a central directory solution and be sure that my user would get a good experience my application.


## Conclusion

Building a proper circle of trust in a distributed manner is complex.
The EIDAS came up as an alternative to a central directory but is today not offering the same comfort and user experience than a central directory. 
It's feels like a middle ground would be more satisfying, with a central directory accepting EIDAS as a way to register in it.
As a developer, you can choose to use the Open Banking UK directory or getting an EIDAS. If you are here just to discover Open Banking, your best option is to use the ForgeRock Open Banking directory.

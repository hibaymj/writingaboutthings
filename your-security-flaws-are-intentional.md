# Your API Security Flaws are Intentional
## A secure API is intentionally designed, if you’re having API security issues then your API design is the problem.

Drip. Drip. Drip. Drip. Your eyes spring open. Everything is dark. You frantically look around for whatever woke you up. The faucet is leaking. Again. No matter how new, expensive, or well maintained that faucet just leaks. It’s _designed_ that way. The same is true for security flaws in your API designs.

Don’t worry, this isn’t a conspiracy theory. The faucet companies aren’t out to drive you mad. It’s just a design flaw. Their faucet has specific manufacturing tolerances to keep prices low. An inner seal is a on the small side of acceptable, while the outer seal is on the bigger side. You tighten everything up. But you know it’s going to happen again. The design is flawed, and your faucet leaks.

The design was completely intentional. The consequences to you weren’t. Engineers performed studies on tolerances for the parts to balance costs and quality. Your faucet is a very complex failure in a small package.

Complex failures are maddeningly difficult to prevent. Their preconditions are specific and numerous. Ultimately, this makes them very rare. Is it rare enough?

There are two main factors which contributed in a variety of ways:
* **Assumptions**: The engineers assumed if both parts were within tolerances, the whole faucet would work. But they didn’t take into account relative tolerances for the spaces between. They assumed the design specified enough. It didn’t.
* **Tolerance Objectives**: The primary design goal for the faucet was to keep prices low. The intention was to create a functional, low cost faucet, and price was more important. They created leaky failure scenarios when they compromised that design objective.

Engineers make tolerance assumptions and balance it against costs to manufacture. This design is responsible for the failure even though it’s far removed in both time and space.

## Complex Environments Breed Tolerated Complex Failures

In a recent post Nauman Ali walks us through a great scenario where [well intentioned developers make benign changes to a contact model](https://stoplight.io/blog/conflicting-model-problem/). He shows how over time these models become increasingly incompatible with direct impact to software quality. 

APIs are the digital model of your organization’s business. Organizations are very complex environments. Your API designs _will_ be complex. In this example “Contact” represents a physical mailing address to one group, and an email address to others.

Just like our faucet, the two different “Contact” models are both well designed _in isolation_. The issue arises when you look at the assumptions and tolerances of the API platform as a whole. When you look at your APIs, do you see and recognize the cracks? Suppose those responsible for API security make the (in isolation) entirely reasonable assumption that “Contact” refers to an email address and physical addresses are exposed. What happens to them? Are they entirely responsible for this complex failure? Is it really their job to know every detail about every resource in the APIs? What happens to the platform?


### The Origins of our Leak

Consider both of these services are part of a multi-tenant SaaS provider’s core platform. We have two services referred to as “Contact”, for clarity we’ll refer to the newer one as “Email-Contact” and the other as “Physical-Contact”. Security is usually defined at the URL and sometimes HTTP Method level. Email-Contact will be deployed internally at `https://email-contact-service.example-platform.com/api/contacts` and Physical-Contact at `https://address-contact-service.example-platform.com/api/contacts`.

Our team creating the Email-Contact service works with their security team to define authorization rules. Ultimately, security personnel create a new rule that `/api/contacts` requires an authenticated user with “owner” or “admin” role to access. However, users have the ability to create organizations on the platform and this will grant them “owner” role. This capability is available to free users. We now have all the ingredients for an attacker to retrieve both the emails and physical addresses of all contacts on the platform.

The attacker creates a free organization, granting themselves the “owner” role. Using this they can make calls to both contact services, providing unauthorized access to contact physical addresses.

### Attack Surfaces, and Leaky Faucets

Security professionals refer to the “attack surface” of a system as the exposed part where flaws can lead to unintended or unauthorized access, disclosure, or execution. This is an apt metaphor.

Just like with our faucet, the assumptions and tolerances of individual elements of our API platform allows API security leaks to emerge. We can't place fault in any individual. Many things needed to happen for this failure to occur. It all started with our design.

### The Design Problem

Looking back at our faucet example, the issue we ran into was the unintended consequences of our wide tolerances. We have the same type of failure occurring in our API design, so what is happening?

Our security definitions are designed with less specificity than our resources. We know what happened in this case, and we can add checks to each Contacts authorization processing. This is just a patch. The leaks will come again. The only way to ensure security by design is to define it with equal or greater specificity than our resources. Let's see how we can do that.

## API Security by Design

Every API consumer request will specify the following one way or another:
* The target resource
* The consumer's intent
* Intent data
* Authorization information

This is the data we have to operate and secure our services. However, with this scheme the platform also needs one more critical piece of informaton the consumer shouldn't know; where should the request go? This presents a huge problem. The fundamental reason we build APIs is to decouple the client and server. If we ask the user to provide more information we destroy the value we are trying to create in the first place.

Clearly we must find another solution. Let's take a deeper look into whats causing this contention. To do that we need to consider a little bit of networking in a microservice architecture. Web applications use URLs to identify different security _contexts_, or places to define security rules. This is done to decouple the security definitions from the network hardware level identifiers like _IP Address_ and _port_. This is done to prevent security issues from creeping in when hardware fails or is upgraded, or the application moves to a different environment. This sounds familiar to our API decoupling goals. More on that in a bit.

There is a fundamental assumption with this pattern; you can do anything you want if you have access to a security context. This approach was created to secure large monolithic web applications where all the capabilities live together in the same codebase. In this case the approach fits extremely well. However, its problematic with a microservices architecture because it reintroduces the same fundamental security challenges.

Let's see some examples:
* `https://webhost-dev-12-2:9999/welcome.html` moves to `https://webhost-prod-4-1:9899/welcome.html`. In this case we have an internal DNS name host, the application moves from a `DEV` to a `PROD` environment, and the port changes as well.
* `https://webhost-prod-4-1:9899/welcome.html` is externally exposed at `https://example-platform.org/welcome.html`. Again the hostname changes, and the port is dropped entirely.

The only stable value in these examples is the _relative path_ after the the `TLD` and `port`. This explains why _security contexts_ are defined by the relative URLs. Obviously that's not the whole picture. A small number of networking rules are configured per-application to ensure the traffic finds the correct destination. The `base URL` is identicle for all the security contexts.

How does this look with microservices? Recall our two services, contact and physical-contact, are deployed at `https://email-contact-service.example-platform.com/api/contacts` and `https://address-contact-service.example-platform.com/api/contacts` respectively. Immediately we're faced with a huge problem. the `base URL` is different. Meaningfully different. Each sub-domain is pointing to a unique application!

Worse yet, these sub-domains can only be utilized internally. The naming of our two contact services overlap. We can figure out a functional approach or require external consumers to use our internal implementation details.

This is a simple, perhaps difficult to accept, yet profound truth: a URL is insufficient to define a _security context_. This _simplified_ approach only worked in the monolithic deployment because all the requests had consistent `base URLs` and only required a few DNS routing rules.

This relatively simple case is only exacerbated using containers to further divide traffic on a specific host! This is only going to get worse, so how do we _fix it_?

## Securable Design

To secure APIs we need tighter tolerances, we need to specify something _more_. It's time to talk about consumer intent, and intent data. Consumer intent is the type of the consumer's desired transition or representation expressed to the resource. Intent data is the data necessary to flesh out the specifics of the consumer's desire. 

> [Roy Fielding wrote:](https://roy.gbiv.com/untangled/2008/rest-apis-must-be-hypertext-driven)
> A REST API should spend almost all of its descriptive effort in defining the media type(s) used for representing resources and driving application state, or in defining extended relation names and/or hypertext-enabled mark-up for existing standard media types. Any effort spent describing what methods to use on what URIs of interest should be entirely defined within the scope of the processing rules for a media type (and, in most cases, already defined by existing media types). 

The simplest way to use this to solve our problem is to use metadata outside of the request body and URL to declare consumer intent. Lets see some HTTP requests in action:
```Accept: application/vnd.example-platform.contacts.email+json
GET https://example-platform.org/api/contacts/12345
```
```Accept: application/vnd.example-platform.contacts.mailing-address+json
GET https://example-platform.org/api/contacts/45678
```
What we have done is use Vendor Media Types to tell the server exactly which type of contact we would like. Assuming the schemas for these mediaTypes are known, this is precisely the information we need to resolve our security conflict! By moving the additional information into the `Accept` header, we have made the consumer's life easier AND made our service more secure!

## Wrapping Up!
The security of our APIs is dependent on our design tolerances. If our API design allows ambiguity, depends on overloading the resource locator (URL), or assumes every method requires the same access then we will continue to have API security breaches. Our design ensures it.

If you want to secure your APIs, then tighten up your tolerances. You may notice a large improvement in your developer experience along the way.

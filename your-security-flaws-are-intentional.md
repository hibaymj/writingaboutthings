# Your API Security flaws are intentional.
## A secure API is intentionally designed, if you’re having API security issues then your API design is the problem.

Drip. Drip. Drip. Drip. Your eyes spring open. Everything is dark. You frantically look around for whatever woke you up. The faucet is leaking. Again. No matter how new, expensive, or well maintained that faucet just leaks. It’s designed that way.

Don’t worry, this isn’t a conspiracy theory. The faucet companies aren’t out to drive you mad. It’s just a design flaw. Their faucet has specific manufacturing tolerances to keep prices low. An inner seal is a on the small side of acceptable, while the outer seal is on the bigger side. You tighten everything up. But you know it’s going to happen again. The design is flawed, and you’re faucet leaks.

The design was completely intentional. The consequences to you weren’t. Engineers performed studies on tolerances for the parts to balance costs and quality. Your faucet is a very complex failure in a small package.

Complex failures are maddeningly difficult to prevent. Their preconditions are specific and numerous. Ultimately, this makes them very rare. Is it rare enough?

There are two main factors which contributed in a variety of ways: assumptions, and tolerance objectives.
* The engineers assumed if both parts were within tolerances, the whole faucet would work. But they didn’t take into account relative tolerances for the spaces between. They assumed the design specified enough. It didn’t.
* The primary design goal for the faucet was to keep prices low. The intention was to create a functional, low cost faucet, and price was more important. They created leaky failure scenarios when they compromised that design objective.

Engineers make tolerance assumptions and balance it against costs to manufacture. This design is responsible for the failure even though it’s far removed in both time and space.

### Complex Environments breed tolerated Complex Failures

In a [recent post](https://stoplight.io/conflicting-model-problem/) [Nauman Ali](https://stoplight.io/blog/author/nauman-ali/) walks us through a [great scenario](https://xkcd.com/927/) where well intentioned developers make benign changes to a contact model. He shows how over time these models become increasingly incompatible with direct impact to software quality. 

APIs are the digital model of your organization’s business. Organizations are very complex environments. In this example “Contact” represents a physical mailing address to one group, and an email address to others.

Just like our faucet, the two different “Contact” models are both well designed _in isolation_. The issue arises when you look at the assumptions and tolerances of the API platform as a whole. When you look at your APIs, do you see and recognize the cracks? Suppose those responsible for API security make the (in isolation) entirely reasonable assumption that “Contact” refers to an email address and physical addresses are exposed. What happens to them? Are they entirely responsible for this complex failure? Is it really their job to know every detail about every resource in the APIs? What happens to the platform?


### The origins of our leak

Consider both of these services are part of a multi-tenant SaaS provider’s core platform. We have two services referred to as “Contact”, for clarity we’ll refer to the newer one as “Email-Contact” and the other as “Physical-Contact”. Security is usually defined at the URL and sometimes HTTP Method level. Email-Contact will be deployed internally at `https://email-contact-service.example-platform.com/api/contacts` and Physical-Contact at `https://address-contact-service.example-platform.com/api/contacts`.

Our team creating the Email-Contact service works with their security team to define authorization rules. Ultimately, security personnel create a new rule that `/api/contacts` requires an authenticated user with “owner” or “admin” role to access. However, users have the ability to create organizations on the platform and this will grant them “owner” role. This capability is available to free users. We now have all the ingredients for an attacker to retrieve both the emails and physical addresses of all contacts on the platform.

The attacker creates a free organization, granting themselves the “owner” role. Using this they can make calls to both contact services, providing unauthorized access to contact physical addresses.

### Attack Surfaces, and leaky faucets

Security professionals refer to the “attack surface” of a system as the exposed part where flaws can lead to unintended or unauthorized access, disclosure, or execution.

Just like with our faucet, the assumptions and tolerances of individual elements of our API platform allows API security leaks to emerge.

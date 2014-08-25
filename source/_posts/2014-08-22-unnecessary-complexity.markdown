---
layout: post
title: "Unnecessary Complexity"
date: 2014-08-22 17:20:18 -0700
comments: true
categories: 
---

<div align="center">
"Any intelligent fool can make things bigger, more complex, and more violent. It takes a touch of genius — and a lot of courage — to move in the opposite direction." - Einstein
</div>
<p/>
Software development is hard.  That's a fact.  Not only do we have a product that we must enhance, develop from scratch, 
or even maintain, but we have people's interactions, opinions, emotions, the list goes on.  And then there is design, 
technology and execution.   All these factors, all these variables make software development the fantastic challenge 
that it is.  And I love it.  At the same time, I am on a mission to remove complexity introduced into a system when it 
need not be there.  The actual doing of this, making the decisions about what to include, what to leave out and to do 
this consistently on a day to day basis is difficult.

As Product Owners and Engineers, we are faced with these decisions all the time as we work on a system.  Each decision, 
from those more-complex-than-it-needs-to-be features to irreversible design decisions to that single line of code, all 
have an impact on our productivity and agility.  Everybody talks about keeping it simple and taking a lean approach - 
this is nothing new.  It is how we can achieve this when making those decisions on the ground that is the focus of this 
blog.

There are at least five major areas where we love to introduce complexity: feature development, execution, design, 
technology selection and code.

Here I want to go through a number of approaches that I have used that help prevent complexity being incurred because, 
let's face it, the solution should only be as complex as the problem at hand.

##Unnecessary feature complexity

The basic tenet of unnecessary feature complexity, especially when dealing with new product development, 
is to answer the question "What is the minimum we have to do to figure out if this is going to work?" Again, nothing 
new.  This is part of the Lean movement.  I'm sure you have your own, but allow me to to offer the following approach 
to exploring a new product or service:

* What is the minimum amount of work we need to do in order to have the initial conversation with the first client?
  * Rough idea on features available
  * Rough idea on tech, format, deployment and performance.  Discussion based, no documentation required.
* What is the minimum amount of work we need to do to sell to our first client?
  * Minimum Marketable Product (MMP) determined along with feasibility
  * More information about data required to be supplied and what would be returned.
  * NOT the API contract
  * (Internal only) Rough idea of implementation time to version 0.1 for first client.
* Once we have sold with delivery estimate to client:
  * Lock down API contract or feature set including success/failure scenarios - deliver early so client can start mocking
out from their end and write the integration code
  * Integration approach detailed
  * Deployment architecture detailed
  * Performance characteristics and design to support detailed
  * Implement and deliver MMP
  * Roadmap for any further features created based on initial usage (iterative and incremental)

Pretty straight forward, huh?  Yet we all know when we have done too much design after a product or idea has been put 
on the backburner.  Ever thought "We really didn't need to do that much work to make that decision to defer the product"? Make the hard decisions now to save time later.

Of course, there are times when the above steps do not make sense, or they need to be modified.  For example, 
when doing a legacy migration. There are likely many clients that rely on features already present in the legacy system. In these cases, 
we can take the opportunity to evolve the clients during the migration and do some spring cleaning of 
the legacy system, i.e. answer the questions "What features do we really need?" and "What features have proven to not 
be of value given what we know now about our business?"  Unfortunately, as is most often the case, there are "hidden" 
organic features that have no documentation that can trip us up.

##Unnecessary execution complexity

There are many in this category, so let's just focus on a few:

* Break big problems down into smaller pieces:  Be exact with the acceptance criteria.  This one is self explanatory.
* Business Accepting a story only when it is deployed and verified in production: This is a technique that <a href="http://mikquinlan.com/2013/07/28/no-more-burndown-no-more-definition-of-done/" target="_blank">I've spoken about before.</a> 
It encourages quality and promotes continuous deployment, a part of the Continuous Delivery strategy.
* Keep to 1 week iterations if suitable: As long as there is a long term strategy, <a href="http://mikquinlan.com/2013/03/07/re-platforming-a-system-and-the-value-of-1-week-iterations/" target="_blank">1 week iterations</a>
give more opportunities to retrospect and force breaking down of stories into smaller component parts of business value while keeping the overall mission squarely in the centre of the picture.
* Automate delivery:  Once we have small stories and iterative, incremental development, deploying manually, 
especially if we are practicing BA'ing a story once it is in production, takes a lot of effort. Make the investment in 
Continuous Delivery supported by automation, taking into account the aspects <a href="http://mikquinlan.com/2014/01/27/continuous-delivery-the-missing-piece-of-the-puzzle/" target="_blank">I have blogged about before</a>.
Micro Services are also a viable option (note: Micro Services is a bad name, IMO, a service should just be small enough to fit in our
headspace...but that is a topic for another blog).

##Unnecessary design complexity

Keep it simple. We all know the acronym. Here are some ideas to help focus development efforts:

* Limit the number of business strategies/initiatives an organisation takes on:  This could actually be a section all 
on its own.  I have frequently been in organisations where multiple business strategies are pursued concurrently.  
There is nothing wrong with pursuing multiple business strategies, but when it exhausts the capacity of the organisation, 
then we end up with a lot of half done ideas that never gain the momentum they're supposed to due to not enough attention 
being paid to them (market forces notwithstanding).  This then trickles down into design.  Design is hard.  We must 
make sure we design within our capacity so we build something that will work and work well enough to fulfill the most 
valueable strategic objectives that we carefully chose.
* Keep designs incredibly straight forward wherever possible: Again, not a new concept, yet when on the ground how many 
times have you looked at a design and thought "that is waaaay too complex"?  More than once, maybe?  Not every edge 
case needs to be covered.  Work with the business to understand the acceptable levels of failure a system can have.  
During design, when an edge case comes up, if it's technical weigh up the likelihood of occurrence vs the effort to fix.
Don't just dive in and fix it because it's there.  If it's a business edge case, work with the business to see if it 
needs to be resolved or whether a failure is acceptable.  Again, make hard decisions here.  Overall, taking a little 
longer to decide that a feature will not be implemented will save effort over a quick decision to do it then having 
the entire team take on that complexity.

##Unnecessary technology complexity

Keep the learning curve as low as possible so that engineers have less to learn technologically and can focus on the 
solutions implemented to create the product or service.  Whatever we can do to have them focus more on the problems 
being solved rather than the different libraries we used to, say, create and access files can only be a good thing:

* Organisationally it is important to limit the technologies being used so we can build upon the knowledge acquired by
our fellow engineers.  If we have one NoSQL database, do we really need another?  For a technology selection where 
NoSQL is deemed to be a good fit, it is important to come up with a reason why we cannot use a NoSQL technology that 
is already in production within our organisation, along with all the libraries we have built around it.  Of course, 
if that technology is not a fit, we then must be able to articulate the distinct advantages of the technology we want 
to introduce.  Polyglot solutions are a good thing, applied judiciously.  Again, effort vs value.  In this case, effort 
to de-risk, implement and build libraries/tools around the new technology vs value to the business.
* Within a service it is important to limit the libraries used.  The number of libraries a single service can end up 
using can be huge.  That huge effective pom.xml if you use Maven, that long list of gems.  Limit the libraries used for 
a service so that engineers will not just reach for the library they know when facing a problem, adding to the burgeoning 
list of libraries required for the project.

##Unnecessary code/implementation complexity

Have you ever looked at a codebase and seen multiple ways of achieving the same thing?  For example, file handling, 
filtering specific objects out of lists etc.?  Yeah, me too.  It's important that we do common operations consistently 
within a codebase (and across services...harder but still possible).  Agree within the team how this should be done 
(use the <a href="https://code.google.com/p/guava-libraries/" target="_blank">Guava libraries</a>? <a href="http://commons.apache.org/" target="_blank">Apache Commons</a>? etc.)
and stick to it.  If you are an engineer and unsure, look at other parts of the codebase.  Common methods of handling 
code results in less to learn and a decreased ramp up period for engineers new to it.

##A note on back end infrastructure systems vs front end product development

Front end product development needs rapid delivery.  So we can't always live in a world where everything is consistent.  In these scenarios, agree among the team members the libraries to use, approach etc. and forge ahead developing the new product.  In this case, technical debt will ensue, but this is good technical debt.  In fact, we could say it is "necessary" technical debt. Necessary because not adding a feature can be as key to being first to market as adding a feature.

If the product or feature is successful and is moved from proof to long lived, address the technical debt that has been incurred.  This typically happens when a product gets traction and a company is coming out of the startup phase. A great blog that articulates technical debt is 
<a href="http://blog.crisp.se/2013/10/11/henrikkniberg/good-and-bad-technical-debt" target="_blank">this one</a> by Henrik Kniberg.

##Summary

I hope the above guidance helps.  Each of the bullet points could be a topic in itself, and I wanted this blog to be a collection of areas to consider.  If I was to sum this up in one sentence, it would be

"Take just a little more time to make the hard decisions."

I am always looking for ways to make the development of software and, by extension, products easier, so I'd be interested to know what strategies you use to limit complexity in a system. Feel free to leave a comment, below.

Aaaaand as I completed this, that lovely chap Dan North just released <a href="https://www.youtube.com/watch?v=XqgwHXsQA1g" target="_blank">this presentation</a>. :-)

Note: This is a modified cross post from one that originally appeared <a href="http://mikquinlan.com/2014/08/14/unnecessary-complexity/" target="_blank">here</a>. 

**About the author**: Mik Quinlan is Director of Engineering - Mobile Advertising at Thinknear by Telenav. He has
over 16 years experience architecting and implementing highly scalabale, high throughput, low latency systems and
has led several Agile transformations at various companies. Connect via <a href="https://www.linkedin.com/in/mikquinlan" target="_blank">LinkedIn</a>
or <a href="https://twitter.com/mikquinlan" target="_blank">Twitter</a>.
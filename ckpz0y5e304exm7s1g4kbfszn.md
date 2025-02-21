---
title: "Introduction into Domain-Driven Design (DDD)"
datePublished: Wed Jun 16 2021 05:19:09 GMT+0000 (Coordinated Universal Time)
cuid: ckpz0y5e304exm7s1g4kbfszn
slug: domain-driven-design-introduction
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1623820118835/s5gapCsww.png
tags: programming, tools, developer, ddd

---

The concept of Domain-Driven Design (DDD) was introduced by Eric Evans. He wrote about it in his book Domain-driven Design back in 2004 *(aka "The Big Blue Book")*. 

ℹ️ What is a domain in the context of DDD?
> A **sphere of knowledge, influence, or activity**. The **subject area** to which the user applies a program is the domain of the software.

I will provide you with an overview of what DDD is all about. This post is about the **"Why?" and "What?" of DDD**. I won't deep dive into specific topics here. Nevertheless, I will point out some definitions of important terms like I just did with "domain". As you will realize a shared vocabulary also is part of DDD itself.

First of all, you ask yourself if the concepts are still valid? 

# Relevance of DDD Today
DDD is not about a certain technology, it is a concept. More modern concepts are directly referring to DDD. A quick citation from the very first page of the book Building Microservices by Sam Newman:

> Eric Evan's book DDD helped us understand the importance of representing the real world in our code. and showed us better ways to model our systems.

Also, the [IBM Garage Event-Driven Reference Architecture](https://ibm-cloud-architecture.github.io/refarch-eda/methodology/domain-driven-design/) has a chapter about the methodology DDD.

To answer the question about the relevance: It is indeed very valid until today. With the up-to-dateness out of the way let's move on and explore the "Why?" and "What?"

# Why Does DDD Exist?
Why is there even a need for something like DDD? The subtitle of Eric Evans book provides a really good hint:

> Tackling Complexity in the Heart of Software

Typically the critical complexity of software projects is in the domain itself. You can not change the complexity within the domain. You can try to tell your client in the banking sector that you will focus on payments and ditch lending business because it is too complex.

Additionally, we do not develop software just for the purpose of developing software. We do it in order to solve problems and improve things:

> The heart of software is its ability to solve domain-related problems for its user. All other features, vital though they may be, support this basic purpose.

# What is DDD About?
So what actually is DDD? DDD can't be defined in a sentence or two. It is an approach to the development of complex software consisting of multiple puzzle pieces:

1. Focus on the core complexity and opportunity in the *domain*.
2. Explore the *models* in a collaboration of domain experts and software experts.
3. Speak a *ubiquitous language* within a *bounded context*.

*(Note the italic words. These are specific terms in DDD and I will define them like I already did with the word domain.)*

These three points are already a very high-level summary of DDD but I'll go through them one by one in the following sections.

## Focus on Domain

> Focus on the core complexity and opportunity in the *domain*.

I can keep this one short. It is essentially about the things I already pointed out in the "Why?" chapter above. Software is no self-purpose.

Don't confuse this with "focus on the business". The core complexity and opportunity can be different from what we would call "the business". Think about Twitter. Providing the functionality of Tweets, following each other, etc. wasn't the main complexity. Most probably the main complexity was more in scaling that platform. 

## Explore the Models

> Explore the *models* in a collaboration of domain experts and software experts.

Models are a way to deal with the aforementioned complexity. But, first of all, what even is a model?

ℹ️ What is a model in the context of DDD?
> A **system of abstractions** that **describes selected aspects of a domain** and can be used to solve problems related to that domain.

Another way describing the term (also from Eric Evans himself):
> A model is a **simplification**. It is an **interpretation of reality** that abstracts the aspects relevant to solving the problem at hand and ignores extraneous detail.

I think it does worth it to explicitly point out what it is not: **It is not a diagram** (therefore also not an Entity Relationship Diagram (ERD) - even though *entity* is also a term used in DDD). A diagram just helps to communicate the model. You'll see a lot of diagrams in the context of DDD. There is no specific, one-and-only modeling language. Most often some UML-like diagrams are used or just a handwritten sketch. It is not about the diagram, but about the concept behind it.

The verb "explore" is chosen very intentionally over something like "writing down". It will be an iterative process to get to a model. There is no such thing as finishing the models once and for all. There will be new insights, new or changing problems in the domain or whatsoever. It is an ongoing exploration.

--- 
### Excursion: Example of a Model
Think of a typical map of the world *(somewhat like a diagram)*. The standard map of the world shows the Mercator projection. Most of you will know that this map is not about the sizes of different countries or something like that. It was specifically created for navigation on the sea. You can draw a line between points on the map and know how to navigate with your compass.

This is what is behind the map, the actual model: 
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1623700470766/hbP4Kepgk.png)

This example used by [Eric Evans in his 2019 conference talk](https://youtu.be/pMuiVlnGqjk?t=1787) is a good one. It points out that the model is directly related to the problem at hand.

--- 

> Having a model doesn't actually give you any advantage as such. The advantage doesn't come from having a model. The advantage comes from having a **model that is a very nice fit for a specific set of problems that you are trying to solve**.

There is one crucial part of the opening sentence missing: "collaboration of domain experts and software experts". The model has to be explored in a collaboration. Neither the domain experts create the model nor the developers sell their ERD as the model. The process of collaboratively exploring the model is called "Knowledge Crunching" by Eric Evans.

Closing this section with a last, very important point best expressed through two quotes of Eric Evans:

> The thrust of this book is that **one model should underline implementation, design, and team communication**. Having separate models for these separate purposes poses a hazard.

> If the people who write the code do not feel responsible for the model, or don't understand how to make the model work for an application, then the model has nothing to do with the software. **If developers don't realize that changing the code changes the model, then their refactorings will weaken the model instead of strengthening it.**

This gist of these: **the code and the model are directly related.**

## Speak a Ubiquitous Language

> Speak a *ubiquitous language* within a *bounded context*.

ℹ️ What does ubiquitous language mean in the context of DDD?
> A language **structured around the domain model** and **used by all team members** **within a bounded context** to connect all the activities of the team with the software.

ℹ️ There is one other term in it worth defining: bounded context
 >**Context:**
The setting in which a word or statement appears that determines its meaning.    **Statements about a model can only be understood in a context.**

>**Bounded Context**:
A description of a boundary (typically a subsystem, or the work of a particular team) **within which a particular model is defined and applicable**.   

*(Now we have all definitions of this article out of the way - I promise.)*

It is worth noting that "ubiquitous" language is not ubiquitous in a sense of being applicable system-wide. It is rather about using in code, speech, diagrams, etc. That is also why the restriction "within a bounded context" is added. The language will be applicable within the boundary you have to explicitly define. It is not about defining terms across boundaries, e.g. for the whole company. For example, what makes something like an article could be something very different in the marketing or warehouse context, maybe even within different parts of the warehouse context.

*(Maybe you have realized that I added "in the context of DDD" to the definitions. This is exactly why.)*

In my own words: Ubiquitous language is a common language directly interconnected with the model of the given domain, which is used everywhere within your defined boundaries. The part "directly interconnecting with the model" is revealing that a change to the language should also directly be related to changing the model as well. In fact, the model should be created using terms that are already be used by the domain experts.

> [...] changes to the language will be recognized as changes in the domain model and will lead the team to update class diagrams and rename classes and methods in the code [...] Use the model as the backbone of the language. [...] Use the same language in diagrams, writing, and especially speech.

Great, now we know what the term means, but why is this important? This part of the book gives an impression of why:

> On a project **without a common language**, developers have to **translate** for domain experts. Domain experts **translate** between developers and still other domain experts. Developers even **translate** for each other.

I bet you had encountered situations where you as the developer had no idea what the domain expert was talking about or even worse, you thought you know but he/she was talking about something else. The same goes the other way around.

Make it a habit in the team to ask for the meaning of specific terms being used. The ubiquitous language has to be exercised by the team in all communication relentlessly. The domain expert is talking about a concept in the given domain which is not reflected in the model? The model is probably incomplete.

The ubiquitous language will lead to the code and the actual domain moving closer together. It is not only about the code reflecting the actual domain, it has also quite a few other positive aspects. Think about the maintainability of the code. If another developer gets back to the code after a while there will be terms used in the domain. Domain experts do have the knowledge about the terms being used, they are not just classes with names made up by the developers.

# There is More
This was a rather high-level introduction to DDD. But there is of course more to it.

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1623733629176/pK96QwPuC.png)

This image provides an overview of different concepts within DDD. As you can see, I wrote only about a very small part of it. I will write a follow-up post about concepts like *Entity*, *Value Object*, and *Aggregates*. But this post should be enough to get an understanding of what DDD is all about.

# Conclusion
DDD is still an important concept to have in your toolbelt. It has influenced a lot of other areas already *([have a look at the JavaDoc of the JPA Specification](https://github.com/spring-projects/spring-data-jpa/blob/main/src/main/java/org/springframework/data/jpa/domain/Specification.java))* and still is influencing them.

Solving problems of the domain is what developing software is all about. We have to deal with that complexity. But we can make our lives easier by applying DDD to handle this complexity.

---
Hope you liked it. Happy to hear your feedback. Let us connect on [Twitter](https://twitter.com/JannikWempe) or [LinkedIn](https://www.linkedin.com/in/jannik-wempe/).

# Resources

Of course the main resource: [The book DDD](https://www.amazon.com/Domain-Driven-Design-Tackling-Complexity-Software/dp/0321125215/)

There is a [free Domain-Driven Design Reference](https://www.domainlanguage.com/wp-content/uploads/2016/05/DDD_Reference_2015-03.pdf) from Eric Evans online.

The [IBM Garage Event-Driven Reference architecture](https://ibm-cloud-architecture.github.io/refarch-eda/methodology/domain-driven-design/) also containing information about DDD.

I liked this [conference talk from Eric Evans at the DDD Europe 2019](https://youtu.be/pMuiVlnGqjk). Of course, there are several other great talks available on YouTube.




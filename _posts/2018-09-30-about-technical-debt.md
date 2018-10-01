---
layout: post
title: "About Technical Debt"
desc: "What is technical debt and how to manage it"
keywords: "Technical debt, software development, debt"
tags: [Technical Debt]
excerpt_separator: <!--more-->
---

Software developers communicate by using different terms and metaphors.
Some of those terms and metaphors are obvious for developers, but not that obvious for managers.

To communicate well, developers and managers need to interpret those terms in the same way.

Technical debt is one of those terms.

Let's try to understand what is a technical debt is and what it isn't.

<!--more-->


## What is technical debt

[This Wikipedia page](https://en.wikipedia.org/wiki/Technical_debt) provides the following definition:

> Technical debt is a concept in software development that reflects the implied cost of additional rework caused by choosing an easy solution now instead of using a better approach that would take longer.

For example, a team needs to create a new feature.
We can choose a faster solution which requires rework with the understanding that it can slow down changes in the future.
On the other hand, choosing the better solution now will require more time to implement.

Technical debt would be the cost updating the faster solution to be a more robust one.

If we avoid improving the solution, then technical debt can grow over time.
Unlike financial debt, if you throw that implementation away, you'll throw it together with technical debt.

First, we need to identify debt in the software. We can do that by understanding our system better.
If we don't, we may start mistakenly generating technical debt in another area believing we are paying it off.

Technical debt is not necessarily a bad thing. That's why there is no need trying to avoid it at all costs.
On the contrary, it could be a very useful approach.

Example:
Imagine a lunch you are about to have with your team.
On the way to a restaurant you realize you've forgotten your wallet.
What do you do?
You can skip your lunch, then stay hungry till the end of the day, be in bad mood, and be less productive.
That also affects your work.

On the other hand, you can borrow some money from one of your teammates and enjoy a meal.

Getting into a debt at that moment is a better choice.

The thing is, once you borrow money you need to pay it back.
If you do, you may borrow money again in the future, when you get into a similar situation.
If you don't, your relations with your mates may change. Next time, you would have to stay hungry.

You should also understand when to pay the debt off.

From the team lunch example, you may pay the debt when you are back in the office.
But what if your wallet is left at home?
Do you need to borrow money from another colleague or it can wait till tomorrow?

Most likely, it can wait till tomorrow. But you need to agree on those terms with the person who lent you the money.

Actually, having a small and manageable amount of debt is healthy for your software.
By identifying those areas of technical debt you can decide when and what to pay off first.


## Causes of technical debt

Software developers are always facing the choice of making certain decisions.
The size of those decision is quite diverse.
Choosing a variable name or a technology are examples of some of those decisions.

The point is, we always make decisions.
Regardless of whether they are conscious decisions or not.

The result of those decisions may generate and accumulate technical debt.

Sometimes, even when we want to go for a slower but better approach,
we may be forced to pick a faster solution by stakeholders and managers, because a company had to deliver a feature "yesterday".

It could also be that the current state of the system is flooded with temporary hacks which make impossible to apply a proper solution,
and we end up creating even more technical debt.


## What to expect from technical debt

All the temporary solutions which remain in the system and mix with each other, start to accumulate debt.
It requires more and more time to fix bugs and add even the most simple functionality.

Sometimes, when technical debt is too high, it can block the development of software completely.
That would not only hurt the software itself but the entire company.

Let's face it, it's not that fun to hunt down those pesky bugs.
Especially when changes that fix a bug introduce two more bugs in different places.
Developers will be unhappy, and as a result, they'll quit.


## How to get out of technical debt

It's simple. You need to pay it off. But that's not always easy to do.

### How to pay it off

When you are in the earlier stages of development, you can build a discipline to pay off debts.

The example scenario can sound like:

We shipped an important feature.
We introduced some level of technical debt.
We needed that feature sooner than later.
The feature works fine and brings some profit.

Now, let's spend the next week cleaning up the implementation.
We may want to improve this part or that area.
Also, let's clean up the implementation of a feature we've made before.

### When the debt becomes smaller

The debt will become smaller when you pay it off faster than you generate it.

To do so, we need to build discipline in a certain way.
One way is to follow the scouts rule: “Always leave the campground cleaner than you found it”.

Robert C. Martin (aka Uncle Bob) rephrased it to apply better to programming.

> "Always leave the code you're editing a little better than you found it."


## What is not technical debt


We've seen the definition of technical debt and can analyze a couple of examples to see what is and what is not technical debt.

### Outdated libraries and frameworks

Let's review the definition of technical debt again.

> Technical debt is a concept in software development that reflects the implied cost of additional rework caused by choosing an easy solution now instead of using a better approach that would take longer.

Let's look at an example of an outdated version of a framework of your choice.

We know that frameworks and libraries come with shiny new functionality and security patches.
Thus, it makes sense to keep them up to date.


Does the current, already outdated version of the framework, make development slower? Well, it depends.
It does if we want to use new features from the new version.

If we don't update the version of the framework in constantly growing software, it will become harder to update it.
That's a fact.

Why is it harder? Is it because of accumulated technical debt or because a software is growing bigger?

In bigger software projects every change takes more time to implement than in smaller ones.
Even in a "perfect world" with projects that have zero level of technical debt.

When we started our project from scratch, we probably picked the latest version of a framework.

Let's consider that over the next few months a new version of that framework has been released.

Does that mean we've inherited technical debt?
No, because we made the best choice we could given the present "newest" version of the software.
Because the changes are out of our control, I wouldn't consider this as technical debt.


### Non-scalable solutions

When we start a new project we may also start to think about how will we scale it to work with 1 million users.

We have a choice. We can do a fast solution which would work on a single server,
or we may start to think about perfect architecture and how to deploy it simultaneously on a 100 servers with 99.99999% uptime.

The second choice would require way more time to implement.
We might think: "Yes, that's the right way to do it and that one-server-architecture will start to generate technical debt".

Well, that thought fits into the definition of technical debt.

But is it technical debt? Let's take a closer look.

Even though people might think the scalable solution is a better approach, it's rarely the case.

If we implement a one server approach, we will be able to save a lot of time and push our idea to the market earlier.
Also, we can start to validate our idea earlier.
Who knows, it could be an idea that's not good enough and nobody wants to use the product.
In that case, we won a lot of time. Now, we can either change the course of the project or shut it down.

On the flip side, if we pick the 100-server-approach, we may be able to scale it.
But let's face it, not every project would be able to hit the ceiling and needs to support 1 million users.
That may never happen.

That term already has a name ["premature optimization"](http://wiki.c2.com/?PrematureOptimization).

Based on those conclusions, non-scalable solutions are not technical debt either.

We may think: "What if we would need to support 1 million users though, then what?" Would it be technical debt?


In that case, of course, it may decrease the performance of our application.
But would it make implementations of features any slower?
Probably not. If that does not make us any slower, then is it technical debt?

Once we see the number of users growing up we can prepare for that.
As a temporary solution, we can scale horizontally to win a little bit of time, while we're thinking about scale vertically.
But that's a subject for another article.

### A Mess

I like one of Uncle Bob's discourses: [A Mess is not a Technical Debt](https://sites.google.com/site/unclebobconsultingllc/a-mess-is-not-a-technical-debt).

Getting into technical debt is a decision.
But if we produce unmanageable and messy code with no test coverage, full of design flaws, and code which is hard to change, we shouldn't use the term technical debt as an excuse.

## Wrapping up

Today we figured out what technical debt is, how we generate it, and what we should do to prevent, or pay it off.
We went through a couple of examples of what is and what is not technical debt.

I hope that will help you clear up some your current or future discussions in your team and with your managers.
It is also important that managers and stakeholders understand the term and are ready to work to pay it off.

If you want to learn more, here are a couple of links worth visiting.

* [Technical debt Wiki page](https://en.wikipedia.org/wiki/Technical_debt)
* [Martin Fowler's Technical Debt Quadrant](https://martinfowler.com/bliki/TechnicalDebtQuadrant.html)
* [A Mess is not a Technical Debt](https://sites.google.com/site/unclebobconsultingllc/a-mess-is-not-a-technical-debt)
* [An "Fun Fun Function" episode](https://www.youtube.com/watch?v=YBJFirHSS5Q)
* [Answers on Quora](https://www.quora.com/What-is-technical-debt)

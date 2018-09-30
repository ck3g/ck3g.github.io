---
layout: post
title: "About Technical Debt"
desc: "What is technical debt and how to manage it"
keywords: "Technical debt, software development, debt"
tags: [Technical Debt]
excerpt_separator: <!--more-->
---

Software developers communicate by using different terms and metaphors.
Some of those terms and metaphors are obvious for developers. Some of them could be not that obvious for managers.

To communicate well, developers and managers, need to interpret those terms in the same way.

The technical debt is one of those metaphors.

Let's try to understand what is a technical debt and what isn't.

<!--more-->


## What is technical debt

[The Wikipedia page](https://en.wikipedia.org/wiki/Technical_debt) provides the following definition:

> Technical debt is a concept in software development that reflects the implied cost of additional rework caused by choosing an easy solution now instead of using a better approach that would take longer.

We need to implement some functionality.
We can choose a faster solution which requires rework because it can slow down the further changes.
On the other hand, the better solution will require more time.

The technical debt would be the cost of changing a faster solution to form a better shape.

If we avoid improving the solution, then technical debt can grow over time.
Unlike a financial debt, if you throw that implementation away, you'll throw it together with technical debt.

First, we need to identify the debt in the software. We can do that by understanding our system better.
If we don't, we may start generating debt into another area mistakenly believing we are paying it off.

Technical debt is not necessary a bad thing. That's why there is no need trying to avoid it at all cost.
On the contrary, it could be a very useful approach.

Example:
Imagine a lunch you are about to have with your team.
On the way to a restaurant you realize you've forgotten your wallet.
What do you do?
You can skip your lunch, then stay hungry till the end of the day.
Be in bad mood and be less productive.
That also affects your work.

Or, you can lend some money from one of your teammates and enjoy the meal.

Getting into a debt at that moment is a better choice.

The thing is, once you lend money you need to pay it off.
If you do, you may lend money again in the future, when you get into a similar situation.
If you don't, your relations with your mates may change. Next time, you would have to stay hungry.

You should also understand when to pay the debt off.

From the team lunch example, you may pay the debt when you are back in the office.
But what if your wallet is left at home?
Do you need to lend money from another colleague or it can wait till tomorrow?

Most likely, it can wait till tomorrow. But you need to agree on those terms with your moneylender.

Actually, having a small and manageable amount of debt is healthy for your software.
By identifying those areas of technical debt you can decide when and what to pay off first.


## Causes of technical debt

Software developers are always facing the choice of making certain decisions.
The size of those decision quite diverse.
Choosing a variable name or a technology are one of those decisions.

The point is, we always make decisions.
Regardless of whether those are conscious decisions or not.

The result of those decisions may generate and accumulate technical debt.

Sometimes, even when we want to go for a slower but better approach,
we may be forced to pick a faster solution by stakeholders and managers.
Because a company had to deliver that feature "yesterday".

It could be also that the current state of the system is flooded with temporary hacks which make not possible to apply a proper solution,
and we need to dive deeper into a debt.


## What to expect from technical debt

As well as financial debt, we need to pay off a technical debt.
If we don't it would start to accumulate.

All the temporary solutions which remain in the system and mix with each other, start to accumulate a debt.
In that case, the debt starts to eat its interest.
It requires more and more time to fix bugs or throw in new (even simple) functionality.

A technical debt in one area makes the area more complex to maintain or to change.
It becomes much harder and slower to deliver new features or improve existing ones.
Sometimes, when technical debt is too high, it could block the development of the software completely.
That would not only hurt software itself. It may hurt a whole company.

Let's face it, that is not that fun to hunt down those pesky bugs.
Especially when changes which fix a bug introduce two more bugs in different places.
Developers would be unhappy, and as a result, they'll quit.


## How to get out of technical debt

It's simple. You need to pay it off. But that's not always easy to do.

### How to pay it off

When you are on the earlier stages of development, you can build a discipline to pay off debts.

The example scenario can sound like:

We shipped the important feature.
We introduced some level of technical debt.
We needed that feature the sooner the better.
The feature works fine and brings some profit.

Now, let's spend the next week to clean up the implementation.
We may want to improve this part and that area.
Also, let's clean up the implementation of a feature we've made before.

### When the debt becomes smaller

The debt becomes smaller then you'll pay it off faster then generate it.

To do so, again we need to build a discipline in a certain way.
One way is to follow the boy scouts rule: “Always leave the campground cleaner than you found it”.

Robert C. Martin (aka Uncle Bob) rephrased it to apply better to programming.

> "Always leave the code you're editing a little better than you found it."


## What is not a technical debt


I've seen the definition of technical debt and can analyze a couple of examples to see what is and what is not a technical debt.

### Outdated libraries and frameworks

Let's review the definition of the technical debt again.

> Technical debt is a concept in software development that reflects the implied cost of additional rework caused by choosing an easy solution now instead of using a better approach that would take longer.

Let's look at an example of an outdated version of a framework of your choice.

We know that frameworks and libraries come with shiny new functionality and security patches.
Thus, it makes sense to keep them up to date.


Does the current, already outdated version of the framework, make development slower? Well, it depends.
It does if we want to use new features from the new version.

If we won't update the version of the framework in a constantly growing software, it would become harder to update it.
That's the fact.

Why is that harder? Is it because of accumulated technical debt or because a software grew bigger?


I think that's because the software grew in its size.
In bigger software projects every change takes more time than in smaller ones.
Even in the "perfect world" projects with zero level of technical debt.

When we've started our project from scratch, probably we've picked the latest version of a framework.

Does it qualify as the choice of simpler or quicker solution? Hardly.
Could you choose a newer version of the framework? No, it did not exist back in a day.

If we had no choice to make it faster, then it was the only choice.
I'm not considering the choice of the different framework here. Let's stick to one particular framework of our choice.

The bottom line of that example.

Then we start the project and choose the newest version of a framework.
It does not make us any faster, nor implies more work later.

Based on those conclusions, I think that is not a technical debt.


### Non-scalable solutions

When we start a new project we may also start to think about how would we scale it to work with 1 million users.

We have a choice. We can do a fast solution which would work on a single server.
Or we may start to think about perfect architecture and how to deploy it simultaneously on 100 servers with 99.99999% uptime.

The second choice would require way more time to implement.
We might think: "Yes, that's the right way to do it and that one-server-architecture would start to generate technical debt".

Well, that thought fits into the definition of the technical debt.

But is it a technical debt? Let's take a closer look.

Even though people might think the scalable solution is a better approach, it's rarely the case.

If we implement a one server approach, we will be able to save a lot of time and push our idea to the market earlier.
Also, we can start to validate our idea earlier.
Who knows, it could be an idea that's not good enough and nobody wants to use the product.
In that case, we won a lot of time. Now, we can either change the course of the project or shut it down.

On a flip side, if we pick the 100-server-approach, we may be able to scale it.
But let's face it, not every project would be able to hit the ceiling and needs to support 1 million users.
That may never happen.

That term already has a name ["premature optimization"](http://wiki.c2.com/?PrematureOptimization).

Based on that conclusions, non-scalable solutions are not a technical debt either.

We may think, what if we would need to support 1 million users though.
Then what? Would it be a technical debt?


In that case, of course, it may decrease the performance of our application.
But would it make implementations of features any slower?
Probably not. If that does not make us any slower, then is it a technical debt?

Once we see the number of users growing up we can prepare for that.
As a temporary solution, we can scale horizontally to win a little bit of time, while we're thinking about scale vertically.
But that's the subject for another article.

### A Mess

I like one of Uncle Bob's discourse: [A Mess is not a Technical Debt](https://sites.google.com/site/unclebobconsultingllc/a-mess-is-not-a-technical-debt).

Getting into a technical debt is a decision.
But if we produce unmanageable and messy code with no test coverage, full of design flaws, and the code which is hard to change, we shouldn't use the technical debt term as an excuse.

## Wrapping up

Today we figured out what is the technical debt. How do we generate it and what should we do to prevent, or pay it off.
We went through a couple of examples of what is and what is not a technical debt.

I hope that will help you clear up some your current or future discussions in your team and with your managers.
It is also important that managers and stakeholders understand the term and are ready to work to pay it off.

If you want to learn more, here is a couple of links worth to visit.

* [Technical debt Wiki page](https://en.wikipedia.org/wiki/Technical_debt)
* [Martin Fowler's Technical Debt Quadrant](https://martinfowler.com/bliki/TechnicalDebtQuadrant.html)
* [A Mess is not a Technical Debt](https://sites.google.com/site/unclebobconsultingllc/a-mess-is-not-a-technical-debt)
* [An "Fun Fun Function" episode](https://www.youtube.com/watch?v=YBJFirHSS5Q)
* [Answers on Quora](https://www.quora.com/What-is-technical-debt)

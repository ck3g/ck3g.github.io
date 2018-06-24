---
layout: post
title: "Introduction to Alexa skills development"
desc: "Read the introduction to Amazon Alexa. How does it work and what kind of devices it supports."
keywords: "Amazon, Alexa, Alexa skills, Lambda, AWS, Serverless"
tags: [Alexa]
excerpt_separator: <!--more-->
---

During the last few months as was sticking to topics around Elixir.

But as a developer there are some other areas in software development I am interested in.
So I would like to write about them too and share my experience.

Recently I was playing around with Amazon Alexa skills development.
I've found that fun and interesting. So let's get started.

<!--more-->

## What is Amazon Alexa

Alexa is a voice assistant from Amazon. It is integrated into smart speakers such as [Amazon Echo](https://amzn.to/2tobQcD) and [Echo Dot](https://amzn.to/2tmNiB8).
As well as into [Echo Show](https://amzn.to/2MUkgku) and [Echo Spot](https://amzn.to/2tuGZuc) devices.
Those devices can provide a different kind of information by a voice request.
The examples are news, weather, playing music and even shop online.

Amazon allows developing custom skills for the Alexa which makes the technology more attractive.

Skills are some sort of applications which can be used by Echo devices.
To use those skills a person should say a wake-up word and activate the skill.
All the interaction with the skill is happening by voice.

So basically you get the voice interface no buttons no screens.

Well, Alexa can duplicate the output of some simple information on your phone or other devices such as [Echo Show](https://amzn.to/2MUkgku) and [Echo Spot](https://amzn.to/2tuGZuc). But in general, all the communication is verbal.

All of those points make technology interesting and worth trying.


## Why develop Alexa skills


Alexa has some advantages over Apple's Siri or Google assistant. You can develop your own skills for Alexa.
At the moment Siri provides some sort of API for developers to use her, but that API quite limited.
Capabilities of Alexa skills, on the other hand, are way too broad.


Also, it seems Amazon puts a lot of effort to raise Alexa skill development.
I see a lot of advertisement.
Amazon runs a bunch of workshops and webinars around the world to spread the knowledge and teach people how to develop those skills.
On these workshops, I've seen people who weren't developers. And yet they were interested in Alexa skills development.

Amazon also runs [a skill promotion program](https://developer.amazon.com/alexa-skills-kit/alexa-developer-skill-promotion).
Where everyone can publish a skill to earn a reward. A part of that reward you are guaranteed to get. For example a T-Shirt or a hoodie.
If your skill would be used by 100 unique users you can earn one of the Amazon devices. Most of the time it's Amazon Echo Dot.
Skills with most unique users can earn a bigger reward.

All you need to do is to publish a skill, submit the form on the page and get your reward.

I've got a couple of Echo Dot devices by participating in that program.

This promotion changes every month so you can use it as a motivation to publish your skills.
There is a disadvantage of that program in my opinion.
You are allowed to apply only with your new skill.
Improvements and updates of your existing skills do not count.
By doing that, Amazon increases the number of skills in the store but does not motivate to improve existing skills.
I wish they change these requirements, so we can see more skill with the higher quality instead of lots of junk skills.


## How do Alexa skills work

Let's take a look at how does a skill work.

First, from a user point of view.

A user says something like: "Alexa, ask Scooter how much is the fish?" and Alexa answers back.
As you can see that sentence sounds pretty normal for the human being.

Let's split it and see that every part means.
The sentence above consists of four parts: "wake up word", "a command", "an invocation name" and "an utterance".

The wake-up word "Alexa" is used to trigger device to start listening to a user.
It can be one of the following four words: Alexa, Echo, Computer or Amazon.
You choose that per device and you use it to activate (wake up) that device.

Then we use the command "ask". That is launch word and tells Alexa how to process the request.
"Ask" is not the only command, it can be also "run", "start", "tell" etc.

Then we have the skill invocation name: "Scooter". It is used to point Alexa to one of the installed skills.
The invocation is defined by skill's developer.
It should be recognizable English (or a language of your skill) word, so Alexa would be able to understand that.
Actually, that is not the only requirement, you can see others on [the documentation page](https://developer.amazon.com/docs/custom-skills/choose-the-invocation-name-for-a-custom-skill.html#cert-invocation-name-req).

The last and optional part till the end of the sentence called the utterance. In our case that is "how much is the fish?".
Why is that optional? It depends on the command we use. For the "ask" command it is required because we need to provide the instructions on what to do.
In that case, Alexa will process our response, respond back to us and finish the job.
If we say "Alexa, run Scooter" it will launch the skill and we can interact further. But again, it depends on the skill.


<p align="center">
  <img src="{{ site.url }}/img/posts/alexa-introduction/how_alexa_works.png" />
</p>

Now let's see how does it work under the hood.

A user says something to Alexa and triggers a skill. The skill calls a lambda function (in most of the cases) to process the request.
It uses utterance to figure out what does user want and behaves accordingly.
It can get additional information from third-party resource to provide it back to the user or can pass some information to some sort of IoT device to perform some action. Anything.

We already know, that the skill is a computer program, therefore, it should be running somewhere.
Here comes the AWS Lambda service.

[AWS Lambda](https://aws.amazon.com/lambda/) is a serverless computing solution.
It allows running your applications without the need to think about servers.
There is, of course, a server behind that, but that completely hidden from a user.
All we do is we upload the code to AWS Lambda service as a separate lambda function.
Then we configure a trigger for that lambda function by either HTTP endpoint or (in our case) link it with the Alexa skill.
That's it. Then AWS Lambda takes care about the rest. It hosts the code, scales it if needed. We only pay for the usage.

AWS Lambda supports a bunch of different languages, such as JavaScript (Node.js), Java, Python, Go and C#.

It is also possible to link your skill with your own server and process all the information there instead of a lambda function.
That is up to you.

## Wrapping up

Now we know what Amazon Alexa is and how does it work. Both from a user point of view and behind the scenes.

In the next articles, we will see how to implement our own skill, test it and publish to the store.

Subscribe in the form below to get notified about new articles. See you next time.

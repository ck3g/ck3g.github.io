---
layout: post
title: "How to call external APIs from Alexa skill"
desc: "Learn how to make external API calls from Alexa skill."
keywords: "Amazon, Alexa, Alexa skills, API calls, API"
tags: [Alexa]
excerpt_separator: <!--more-->
---

In the previous articles, we have covered different aspects of developing Alexa skills.
We've learned [how to pass state between intents](http://whatdidilearn.info/2018/09/02/how-to-use-session-attributes-in-alexa-skills.html) and [between sessions](http://whatdidilearn.info/2018/09/16/how-to-keep-state-between-sessions-in-alexa-skill.html).

Sometimes, you may run into a situation when you need to interact with external services.
It could be a service to fetch weather information or local news.
Or it could your private service to interact with IoT devices.
Basically, anything you can interact with over HTTP.

Now, let's learn how we can interact with external APIs from an Alexa skill.

<!--more-->

We are going to implement a skill to provide quotes from famous people.

Once we ask Alexa for an inspirational quote.
The skill would fetch the quotes from API and tell a random quote back to us.

To begin, I would create [a boilerplate Hello World skill](http://whatdidilearn.info/2018/07/22/use-ask-cli-to-create-and-deploy-alexa-skills.html) with the single `QuoteIntent`.

In the example below, you can see the initial implementation of the intent handler.

```js
const randomArrayElement = (array) =>
  array[Math.floor(Math.random() * array.length)];

const fetchQuotes = () => {
  return new Promise((resolve, reject) => {
    resolve([
      { content: "Quote 1", author: "Author 1" },
      { content: "Quote 2", author: "Author 2" }
    ]);
  })
};

const QuoteIntentHandler = {
  canHandle(handlerInput) {
    return handlerInput.requestEnvelope.request.type === 'IntentRequest'
      && handlerInput.requestEnvelope.request.intent.name === 'QuoteIntent';
  },
  async handle(handlerInput) {
    try {
      const quotes = await fetchQuotes();
      const quote = randomArrayElement(quotes);
      const speechText = `${quote.content} ${quote.author}.`;

      return handlerInput.responseBuilder
        .speak(speechText)
        .withSimpleCard('Inspiration', speechText)
        .getResponse();
    } catch (error) {
      console.error(error);
    }
  },
};
```

We have an `async` handler function. Inside the function, we are fetching a list of the quotes.
Then we get a random quote from the list and say it to a user.

Our goal is to update the implementation of `fetchQuote()` function to fetch the data from some sort of API instead of providing us some hardcoded data.

Let's do that.

There are different ways to make an HTTP call in JavaScript.
Some of them are described in [this article](https://www.twilio.com/blog/2017/08/http-requests-in-node-js.html) from Twilio blog.

For our example, I've picked a library called [axios](https://github.com/axios/axios). Mostly because it is based on [Promises](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise).

First, we need to install it as a dependency. Be sure you are doing that from the directory which contains the source code of the lambda function itself.

```
→ cd lambda/custom
→ npm install axios --save
→ cd -
```

Then, on the top of the `index.js` file, we need to require the library.

```js
const axios = require('axios');
```

Now, we are ready to reimplement the `fetchQuotes()` functions to make an API call.

```js
const fetchQuotes = async () => {
  try {
    const { data } = await axios.get(quotesUrl);
    return data;
  } catch (error) {
    console.error('cannot fetch quotes', error);
  }
};
```

All we do here is making a GET HTTP request to fetch the data from `quotesUrl` (we will get to it in a bit).
Once we get the data, we return it back.

The `quotesUrl` constant contains an URL to an external API. In our case, I am using a JSON file described in [the following gist](https://gist.github.com/ck3g/44afbba3a80270167cedad37bb8114e3).
The gist has a similar to our initial `fetchQuotes()` function structure. It contains an array of objects. Each object has a `content` and an `author` keys.

```json
[
  {
    "content": "It does not matter how slowly you go, as long as you do not stop.",
    "author": "Confucius"
  }
]
```

In order to make it work, we need to define a variable containing the path to the `quotes.json` file from the gist.

```js
const quotesUrl = 'https://gist.githubusercontent.com/ck3g/44afbba3a80270167cedad37bb8114e3/raw/quotes.json';
```

After that, we are ready to deploy the skill and test it.

<p align="center">
  <img style="max-width: 50%" src="{{ site.url }}/img/posts/alexa-api-calls/example.png" />
</p>

And it works.

The complete example you can find on [the GitHub page](https://github.com/ck3g/alexa-skill-how-tos/tree/master/how-to-call-external-apis).

## Wrapping up

We can see that making API calls from Alexa skill does not make much difference as API calls in any other JavaScript projects.

Using such a feature in our skills gives a very powerful ability. Now we need to extend the list of the quotes just by updating the API (in our case it's the gist file).

I am using the gist just for the sake of that example. In the end, it does not matter too much what kind of API do you use, as long as it provides you a data you need.

Of course, it is also possible to make a different kind of API calls.
Your skill may collect some data from the user and perform the `POST` requests to update your services.
Or it could be a chain of HTTP requests to fetch the data based on a user's needs.

See you next time.

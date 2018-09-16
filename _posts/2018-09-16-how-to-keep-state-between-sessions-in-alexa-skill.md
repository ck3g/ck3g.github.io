---
layout: post
title: "How to keep state between sessions in Alexa skills"
desc: "Learn how to use DynamoDB to store the state between Alexa skills sessions."
keywords: "Amazon, Alexa, Alexa skills, Attributes, Persistent Attributes, Toy Robot, Toy Robot simulator"
tags: [Alexa, Toy-Robot]
excerpt_separator: <!--more-->
---

[Keeping some state within a single session of Alexa skill](http://whatdidilearn.info/2018/09/02/how-to-use-session-attributes-in-alexa-skills.html) could play an important role of how the skill interacts with a user.
Keeping the state between sessions may bring your skill to another level.

By doing so, the skill can pick up the user from the middle of a conversation and proceed from there.

Sounds great? So let's learn how we can achieve that.

<!--more-->

As we already know from [the previous article](http://whatdidilearn.info/2018/09/02/how-to-use-session-attributes-in-alexa-skills.html), handlers receive a `handlerInput` object as an argument.
We have also learned that the object contains an `attributeManager` object.

Similar to `getSessionAttributes()` and `setSessionAttributes()`, the `attributeManager` object provides us methods to work with persistent attributes: `getPersistentAttributes()`, `setPersistentAttributes()` and `savePersistentAttributes()`.

By using these methods we can save into and retrieve the attributes from some sort of storage (a database for example).

In today's example, we are going to use [Amazon DynamoDB](https://aws.amazon.com/dynamodb/) as a persistence storage.

DynamoDB is a NoSQL database from Amazon. It is a part of Amazon Web Services (AWS).
That means we don't need to have additional accounts to use it.
Once we have AWS account we are good to go.

Let's jump to implementation.

For today's example, I am going to use the Toy Robot Simulator skill from [the previous article](http://whatdidilearn.info/2018/09/02/how-to-use-session-attributes-in-alexa-skills.html).
If you are not familiar with it, you may want to review it.

We are going to extend the Toy Robot Simulator to store its position even after we have finished an interaction with the skill.
When we run the skill for the second time, we are going to restore its previous position and proceed from there.

First, we need to install and configure the persistence adapter for the skill.

To do that, we need to navigate to the directory with the source code of the lambda function and install the dependency.

```
→ cd lambda/custom
→ npm i ask-sdk-dynamodb-persistence-adapter --save
→ cd -
```

Then on the top of our `lambda/custom/index.js` file, we need to define the usage of the adapter.

```js
const { DynamoDbPersistenceAdapter } = require('ask-sdk-dynamodb-persistence-adapter');
const persistenceAdapter = new DynamoDbPersistenceAdapter({
  tableName: 'ToyRobotStates',
  createTable: true
});
```

To use our database we need to specify a table's name.
As an optional argument, we can ask to create a table in case it does not exist. Let's do so.

The advantage of that approach is that Alexa SDK will handle all the dirty work for us.
It would create the table and manage its structure. All the details would be hidden from us.

As the last step of configuration, we need to tell `skillBuilder` to use the adapter.
The custom skill builder has a `withPersistenceAdapter()` method. That's exactly that we need.

```js
const skillBuilder = Alexa.SkillBuilders.custom();

exports.handler = skillBuilder
  .addRequestHandlers(
    // list of the handlers
  )
  .withPersistenceAdapter(persistenceAdapter) // <--
  .addErrorHandlers(ErrorHandler)
  .lambda();
```

Now our skill is configured to use a persistence layer. It's time to apply it.

Unlike `getSessionAttributes()` and `setSessionAttributes()`, the methods working with persistent attributes do not return us attributes directly.
Those methods returns us [the `Promise` object](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise).
That means we need to adjust our code to work in that way.

First, let's update the `LaunchRequestHandler` to fetch the data from DynamoDB and say different messages back to the user.

```js
const LaunchRequestHandler = {
  canHandle(handlerInput) {
    return handlerInput.requestEnvelope.request.type === 'LaunchRequest';
  },
  handle(handlerInput) {
    return new Promise((resolve, reject) => {
      handlerInput.attributesManager.getPersistentAttributes()
        .then((attributes) => {
          const { position } = attributes;

          let speechText = WELCOME_MESSAGE;

          if (typeof position !== 'undefined') {
            const { direction, x, y } = position;

            speechText = `The position has been restored. The robot is in position ${x} ${y} facing ${direction}.`;
          }

          resolve(handlerInput.responseBuilder
            .speak(speechText)
            .reprompt(speechText)
            .withSimpleCard(CARD_TITLE, speechText)
            .getResponse());
        })
        .catch((error) => {
          console.log(error);
          reject(error);
        });
    });
  },
};
```

In that example, we only retrieve attributes by calling `getPersistentAttributes()`.
If there is a position stored we would say the robot's coordinates back to the user.

Next, we need to actually update that position. We can do it once we place the robot.


```js
const PlaceIntentHandler = {
  canHandle(handlerInput) {
    return handlerInput.requestEnvelope.request.type === 'IntentRequest'
      && handlerInput.requestEnvelope.request.intent.name === 'PlaceIntent';
  },
  handle(handlerInput) {
    return new Promise((resolve, reject) => {
      handlerInput.attributesManager.getPersistentAttributes()
        .then((attributes) => {
          const speechText = 'The robot is in the initial position.';

          attributes.position = {
            'direction': 'north',
            'x': 0,
            'y': 0
          };

          handlerInput.attributesManager.setPersistentAttributes(attributes);
          handlerInput.attributesManager.savePersistentAttributes();

          resolve(handlerInput.responseBuilder
            .speak(speechText)
            .reprompt(speechText)
            .withSimpleCard(CARD_TITLE, speechText)
            .getResponse());
        })
        .catch((error) => {
          reject(error);
        });
    });
  },
};
```

Here we retrieve attributes once again. Then we override the position with initial coordinates.
Then we update attributes by calling `setPersistentAttributes(attributes)`.
As the last step, we save them into the database using `savePersistentAttributes()`.

As soon as a user can use different intents to run the skill (for example): "Alexa ask Toy Robot Simulator to report".
We need to repeat the similar behavior for every handler where the position is used.

Let's update the `MoveIntentHandler`.
But now, if you find the way of describing `Promises` clunky, we can use `await` operator to wait for `Promise`.

```js
const MoveIntentHandler = {
  canHandle(handlerInput) {
    return handlerInput.requestEnvelope.request.type === 'IntentRequest'
      && handlerInput.requestEnvelope.request.intent.name === 'MoveIntent';
  },
  async handle(handlerInput) {
    const attributes = await handlerInput.attributesManager.getPersistentAttributes();
    let { position } = attributes;
    let speechText = '';

    if (typeof position === 'undefined') {
      speechText = 'The robot is not in the position yet. You need to place it first.';
    } else {
      position = calculateNewPosition(position);
      speechText = 'Beep-Boop.';
    }

    handlerInput.attributesManager.setPersistentAttributes(attributes);
    await handlerInput.attributesManager.savePersistentAttributes();

    return handlerInput.responseBuilder
      .speak(speechText)
      .reprompt(speechText)
      .withSimpleCard(CARD_TITLE, speechText)
      .getResponse();
  },
};
```

At first, we declare the `handle` function [as `async`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function).
Then, we use `await` to wait for an `Promise`:

```js
const attributes = await handlerInput.attributesManager.getPersistentAttributes();
```

After that, we do the rest of the job. Closer to the end of the function's implementation we store attributes again:

```js
await handlerInput.attributesManager.savePersistentAttributes();
```

That's it. I will omit the rest of the implementation.
It looks quite similar to once we have covered.

We almost ready to test.

Before we do that, we need to be sure that our lambda function has permissions to manage DynamoDB.

From the [AWS Lambda](https://aws.amazon.com/lambda/) function page, we can check which roles it uses.

<p align="center">
  <img style="max-width: 70%" src="{{ site.url }}/img/posts/alexa-dynamodb/lambda-role.png" />
</p>

Then, we need to find that role in IAM and attach required permissions.
For the sake of that example, it suffices to have the following permissions for DynamoDB: `GetItem`, `CreateTable` and `PutItem` actions.

<p align="center">
  <img style="max-width: 70%" src="{{ site.url }}/img/posts/alexa-dynamodb/dynamodb-actions.png" />
</p>


I assume you are familiar with how to walk those steps.
Otherwise, you can review [the AWS CLI "Hello World" article](http://whatdidilearn.info/2018/07/22/use-ask-cli-to-create-and-deploy-alexa-skills.html#create-a-policy) to check how to create and assign policies.

Once we ensure the proper permissions attached, we can test our implementation.

That is how the dialog with the robot looks:

<p align="center">
  <img style="max-width: 50%" src="{{ site.url }}/img/posts/alexa-dynamodb/part1.png" />
  <img style="max-width: 50%" src="{{ site.url }}/img/posts/alexa-dynamodb/part2.png" />
</p>

You may ask. Doesn't Alexa skill mix the data of different skill users?
Nope. The Alexa SDK is smart enough and splits records in the DynamoDB by User ID or Echo Device ID.
So all the data is separated between users.

That is how our data stored in DynamoDB.
Now, if we navigate to DynamoDB tables in AWS and find our table, we should see a record inside.
The record should contain our attributes.

<p align="center">
  <img style="max-width: 50%" src="{{ site.url }}/img/posts/alexa-dynamodb/attributes-in-dynamodb.png" />
</p>


## Wrapping up

That's pretty much it about storing attributes.
We have covered how to use persistent attributes and DynamoDB to store them.
We've learned how to update the skill's implementation to work with those attributes.
We can also see, that's not that hard to use that feature. Alexa SDK does all the dirty work for us.

Now it's your turn. Try to use that functionality on your skills.

See you next time.

You can find the source code of the complete example on [the GitHub page](https://github.com/ck3g/alexa-skill-how-tos/tree/master/how-to-keep-state-between-sessions).

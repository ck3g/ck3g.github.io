---
layout: post
title: "Local data persistence in React Native using AsyncStorage"
desc: "Learn how to use local storage using AsyncStorage for mobile applications built with React Native."
keywords: "React Native, AsyncStorage, local storage, persistance, persisting layer, Mobile development, iOS, Android, JavaScript"
tags: [ReactNative, BlitzReading]
feature-img: "img/posts/rn-asyncstorage/cover.jpeg"
img-light: false
excerpt_separator: <!--more-->
---

When we build a mobile application, sooner or later we'll face a need to store some user's data to retrieve later.
It can be an authentication token or user's settings.

When we need to solve that issue, we may consider storing the data on a remote server or locally, on the user's phone.

Depending on the situation we may not have a remote server or the data we need to store, doesn't need to be saved remotely.

Today we are going to find out how to store data on a user's phones, and access it every time we need.

We are going to proceed to build an application we've started in [the previous articles](http://whatdidilearn.info/2018/11/18/introduction-to-react-navigation.html) and implement a High Scores feature.
Every time the user finishes the practice session, we will store the results of that session and display the top recent scores.

Let's get started.

<!--more-->

If you are new into these series of blog post where we are building a mobile application from scratch, you can review [previous articles](http://whatdidilearn.info/tags#BlitzReading) to understand the context.
You can also find a source code of that application on [the GitHub page](https://github.com/ck3g/BlitzReading/tree/01e80b0f5641f16f3287d99d72d42268593c3ec6).

## AsyncStorage

React Native has a built-in persistent key-value storage called [`AsyncStorage`](https://facebook.github.io/react-native/docs/asyncstorage).
It allows storing simple sets of data as text or serialized JSON objects.

It picks the right approach of saving data depending on the platform used by an application.

As you can guess from the name, it works asynchronously.
Every function of the library returns a [Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise) object, which we would need to resolve.

Now, let's dive into implementation.

## Implementation

We have a results screen. It renders a number of practiced words.
The practice screen passes the value via navigation params.

We would need to use that value in several places, thus let's extract it into a state.

Open the `src/screens/ResultsScreen.js` file and change the following line

```jsx
Words count: {navigation.getParam('totalWords', 0)}
```

to

```jsx
Words count: {this.state.totalWords}
```

We are indicating here, that we want to display the number from the component's state.

Next, let's define a constructor and set the initial value for `totalWords` as well as an initial value for the high scores.

```jsx
constructor(props) {
  super(props);

  this.state = { totalWords: 0, highScores: [] };
}
```

Now, when the component is mounted, we need to fetch the value and update the state.

```jsx
componentDidMount() {
  const totalWords = this.props.navigation.getParam('totalWords', 0);
  this.setState({ totalWords });
}
```

Now, when we've prepared, let's implement save and load functionality for high scores.

First, we can describe the steps we need to take.

When the results screen is rendered, we need to retrieve already saved results from the store.
Then we need to add a current result to the list of records.
Then we need to persist the updated results back to the store.

Now, when the list of actions is clear, let's proceed with the implementation.

Add the following call to the bottom of the `componentDidMount()` method.

```jsx
this.updateHighScores();
```

The function will handle all the required steps we need in order to fetch, update and save high scores.

Now, we need to implement it.


```jsx
async updateHighScores(totalWords) {
  try {
    let highScores = await fetchHighScores();
    highScores = mergeHighScores(highScores, totalWords);
    saveHighScores(highScores);

    this.setState({ highScores });

    console.log('High Scores', this.state.highScores);
  } catch (error) {
    console.log('Error fetching High Scores', error);
  }
}
```

We are declaring the function with [`async`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function) keyword.
That turns our function into asynchronous one.
We need to do that because we are calling another asynchronous function `fetchHighScores()`.

We are wrapping the whole body into a `try/catch` statement in case our storing and retrieving mechanism raise an error.

First, we fetch stored high scores.
Then we are merging the current result using the `mergeHighScores()` function with the values we've retrieved from the store.
Then we save updated high scores to the store using the `saveHighScores()` function.
After that, we are storing updated results to the state and output them to a console.

The current file doesn't know anything about these 3 functions.
We need to import them at the top of the file:

```jsx
import {
  fetchHighScores,
  mergeHighScores,
  saveHighScores
} from '../storage/highScoreStorage';
```

We've imported functions, now we need to implement them.

Create a new file `src/storage/highScoreStorage.js`.
It will contain all the code required to save and load high scores from the local storage.


First, let's implement the `fetchHighScores()` function;

```jsx
export const fetchHighScores = async () => {
  try {
    let highScores = await AsyncStorage.getItem(STORAGE_KEY);

    if (highScores === null) { return []; }

    return parseHighScores(highScores);
  } catch (error) {
    console.log('Error fetching High Scores', error);
  }
}
```

That function declared with the `async` keywords as well, because we are using an asynchronous [`getItem`](https://facebook.github.io/react-native/docs/asyncstorage#getitem) from the `AsyncStorage` library.

The function retrieves all the data stored under a certain key.
In our case, it's a "HIGH_SCORES" string, which we need to define at the top of the file.

```jsx
const STORAGE_KEY = 'HIGH_SCORES';
```

Now, when we are using `AsyncStorage` we need to import it as well.

```jsx
import { AsyncStorage } from 'react-native';
```

--------------
Async Storage has been extracted from react-native core.
If you are using ReactNative version 0.59, you'll need to install `@react-native-community/async-storage` as a dependency.

Run the following command in your terminal window:

```
npm install --save @react-native-community/async-storage
```
or

```
yarn add @react-native-community/async-storage
```

then instead of

```js
import { AsyncStorage } from 'react-native';
```

import the library as

```js
import AsyncStorage from '@react-native-community/async-storage';
```

----------------



So, we're reading data from the store.
Then, if we didn't get any data, which is happening when we are using the app for the first time, we return an empty array from the function.

If we retrieved some data, we parse it and return the parsed version of it.

Let's look how do we parse that data.

```jsx
const parseHighScores = (highScores) =>
  JSON.parse(highScores).map((highScore) => {
    highScore.createdAt = new Date(highScore.createdAt)
    return highScore;
  });
```

As I already mentioned above, `AsyncStorage` stores the objects as serialized strings.
Thus, the first thing we do, we parse that string into an object using `JSON.parse()`.

In a bit, you'll see that we are storing objects of type `Date`, which is serialized into strings as well.
So, here we are iterating through every high score and convert its date from the string format back to a date object.

By doing all these steps we are providing a valid object right after we retrieved it from the storage.


Let's move to the implementation of the next function. `mergeHighScores()`:

```jsx
export const mergeHighScores = (highScores, totalWords) => {
  const score = {
    score: totalWords,
    createdAt: new Date()
  };

  return [...highScores, score];
}
```

The function receives the list of high scores and the number of total words from the previous session.
It builds an object of the following format: `{ score: 'Number', createdAt: 'Date' }`.
Then returns a new array, which contains all the values from the previous one and the new value.



Finally we get to the `saveHighScores()` function:

```jsx
export const saveHighScores = (highScores) => {
  AsyncStorage.setItem(STORAGE_KEY, JSON.stringify(highScores));
}
```

All that the function does, is receiving the list of high scores and storing it.

To store an item into a local store we are using a [`setItem`](https://facebook.github.io/react-native/docs/asyncstorage#setitem) function.
It receives a key as a first argument and a value as a second.

As soon as we are going to store an object instead of a string, we need to serialize it before saving.

We are using the same "HIGH_SCORES" key as we used for retrieving the data because we want to save data to and load it from the same namespace.

Once we finish that implementation we can run the application, go through the practice session several times and see the data we've persisted:

<p align="center">
  <img src="{{ site.url }}/img/posts/rn-asyncstorage/01-high-scores-console.png" />
</p>

Now, we have the date in the storage and we can retrieve it.
The feature wouldn't be complete without displaying that data on the screen.

For now, let's display top 10 high scores on the results screen, once the user finished the practice session.

In the `ResultsScreen.js`, next to our "Words count" text, add the following line:

```jsx
<HighScores data={this.state.highScores} />
```

We are going to render high scores in a separate component.

Next, import it on the top of the file:

```jsx
import HighScores from '../components/HighScores';
```

Now, create a `src/components/HighScores.js` file where we are about to implement the component.

```jsx
export default HighScores = ({ data }) => {
  const highScores = getTopScores(data);

  return (
    <View>
      <Text>High Scores</Text>
      <TableHeader />
      {
        highScores.map((highScore, index) =>
          <Row highScore={highScore} index={index} key={index} />)
      }
    </View>
  );
};
```

The component receives all the high scores as a `data` property.

Then, we are using `getTopScores` function to get only high scores we need.

```jsx
const getTopScores = (highScores) =>
  highScores
    .sort((first, second) => second.score - first.score)
    .slice(0, 10);
```

First, we are sorting all the high scores by the number of words in the descendant order.
We don't want to show random high scores, we want to display the scores with the highest amount of words on top.

After the sort, we are getting only top 10 records and return them from the function.

Back to our component, we are displaying a "High Scores" as it's title, followed by a `TableHeader` component.
Here how it looks:

```jsx
const TableHeader = () => {
  return (
    <View>
      <Text>#</Text>
      <Text>Words</Text>
      <Text>Date</Text>
    </View>
  );
}
```

It plays the role of table header and has 3 "columns".

Right after the header, we are going through every high score and render a `Row` component for every row.

```jsx
const Row = ({ highScore, index }) => {
  return (
    <View style={styles.row}>
      <Text>{index + 1}.</Text>
      <Text>{highScore.score}</Text>
      <Text>{highScore.createdAt.toDateString()}</Text>
    </View>
  );
}
```

As soon as index started from 0, we are displaying an index value increased by 1 as a high scores rank.
Then we display a score itself followed by a date.

If we run the app, here is what we'll see.

<p align="center">
  <img style="max-width: 50%" src="{{ site.url }}/img/posts/rn-asyncstorage/02-high-scores-unstyled.png" />
</p>

It looks like a mess without styles.

Let's describe some styles and attach them to hour elements.

Fast forwarding, here is how the whole file looks like:

```jsx
import React from 'react';
import { View, Text, StyleSheet } from 'react-native';

const Row = ({ highScore, index }) => {
  return (
    <View style={styles.row}>
      <Text style={styles.index}>
        {index + 1}.
      </Text>
      <Text style={[styles.score, styles.bold]}>
        {highScore.score}
      </Text>
      <Text style={styles.date}>
        {highScore.createdAt.toDateString()}
      </Text>
    </View>
  );
}

const TableHeader = () => {
  return (
    <View style={[styles.row, styles.headerRow]}>
      <Text style={[styles.index, styles.thItem]}>#</Text>
      <Text style={[styles.score, styles.thItem]}>Words</Text>
      <Text style={[styles.date, styles.thItem]}>Date</Text>
    </View>
  );
}

const getTopScores = (highScores) =>
  highScores
    .sort((first, second) => second.score - first.score)
    .slice(0, 10);

export default HighScores = ({ data }) => {
  const highScores = getTopScores(data);

  return (
    <View style={styles.container}>
      <Text style={styles.header}>High Scores</Text>
      <TableHeader />
      {
        highScores.map((highScore, index) =>
          <Row highScore={highScore} index={index} key={index} />)
      }
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    width: 300,
    marginBottom: 15
  },
  header: {
    textAlign: 'center',
    fontSize: 20,
    marginBottom: 15
  },
  headerRow: {
    marginBottom: 5
  },
  row: {
    justifyContent: 'center',
    flexWrap: 'nowrap',
    flexDirection: 'row',
  },
  thItem: {
    fontWeight: 'bold',
    textAlign: 'center',
  },
  index: {
    width: 40,
    fontSize: 12,
    textAlign: 'center'
  },
  score: {
    width: 60,
    textAlign: 'center',
    fontWeight: 'bold'
  },
  date: {
    width: 120,
    textAlign: 'center'
  }
});
```

Now it looks way better.

<p align="center">
  <img style="max-width: 50%" src="{{ site.url }}/img/posts/rn-asyncstorage/03-high-scores-screen.png" />
</p>

That concludes the implementation of High Score feature.

## Wrapping up

We've just learned how we can easily store the data on a user's phone.
We have a fully functional high scores functionality by using only two functions from the `AsyncStorage` library.

Having a local storage can raise a user's experience on a new level.
When the app stores the data locally, it can display it right after the user runs the app.
There is no need to display a pesky loader and an app can work in offline mode.

Of course, when one needs to store large amounts of data and have it to sync with a remote server, then one needs to consider other options.
Luckily, there are plenty of them.

The complete example you can find on [GitHub page](https://github.com/ck3g/BlitzReading/pull/3).

See you next time.

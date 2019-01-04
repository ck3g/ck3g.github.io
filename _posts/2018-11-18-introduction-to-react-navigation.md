---
layout: post
title: "Introduction to React Navigation"
desc: "Split the functionality into separate screens using React Navigation"
keywords: "React, React Native, React Navigation, Mobile development, mobile dev, iOS, Android, JavaScript"
tags: [ReactNative, BlitzReading]
feature-img: "img/posts/rn-navigation/cover.jpeg"
img-light: false
excerpt_separator: <!--more-->
---

In [the previous article](http://whatdidilearn.info/2018/11/11/react-native-basics.html), we start building a new mobile application.
We have managed to finish a working prototype of the application.

Although, the prototype is working, almost the whole implementation held in a single file.
That will make the implementation of the following features slower and trickier.
It makes the code less readable.

Today I would like to split the existing functionality into separate screens.
To achieve that, we are going to install and use "React Navigation" library.

If you haven't read [the previous article](http://whatdidilearn.info/2018/11/11/react-native-basics.html), you may want to read it first to get more context about the app.

<!--more-->

Why do we need React Navigation in the first place?

When we are building applications of the Web, we are able to use links (as `<a>` tags) to navigate between pages.
A web browser also takes care about history and navigating back to already visited pages.

There is no such built-in functionality in React Native, thus React Navigation covers that gap.

React Navigation consists of three basic parts:

  * **Navigator** - Implements navigation pattern and has one or more routes.
  * **Route** - Has a name and links to a screen component.
  * **Screen Component** - Is a React component which will be rendered under that route. Can be another Navigator.

We will see all of them in action.

## Installation

We are going to use version 3 of React Navigation.

Version 3 was just released and pretty new.
At the time I write those line it was [yesterday (November 17, 2018)](https://reactnavigation.org/blog/2018/11/17/react-navigation-3.0.html).

Anyway, I think it makes sense to use it in that article.

To start using the library, we need to install it first.

```
→ npm install --save react-navigation
→ npm install --save react-native-gesture-handler
→ react-native link
```

If you are using Expo SDK, there is no need to apply the last two commands, because that's already included.

Once we apply those commands we are ready to use React Navigation.

## Extracting styles

Now, we have our whole implementation in a single `App` component.
As soon as it is in the same file, we keep the code for styles together with the component.

Before we dive into React Navigation, I would like to extract those styles into a separate file.
That would help us to keep the file smaller and reuse the styles in other files.

Create a new `src/styles.js` file and move the styles from `App.js` into it.

```jsx
import { StyleSheet } from 'react-native';

export default StyleSheet.create({
  // paste the styles from App.js here
});
```

then in `App.js` import them:

```jsx
import styles from './src/styles';
```

and then remove the `const styles = ...` from the `App.js`.

That's it. The simple change will help us later.

## Switch Navigator

Switch Navigator is one of the simplest navigator patterns.
It shows a single screen at a time and does not handle back actions.

In our app we have three relatively independent screens, thus Switch Navigator can be a good choice for us.

First, we can start by defining our navigator:

```jsx
const AppNavigator = createSwitchNavigator({
  App: App
});
```

We have defined the `AppNavigator` which uses the Switch Navigator pattern.
We are passing an object into `createSwitchNavigator`.
The object describes the routes (as keys) and screen components (as values).
In our case, we've named route "App" and link it to our `App` component.

Version 3 of React Navigation requires to define an app container to the root navigator.
So, let's do that and export it from the file:

```jsx
export default createAppContainer(AppNavigator);
```

then we need to import `createSwitchNavigator` and `createAppContainer`:


```jsx
import {
  createSwitchNavigator,
  createAppContainer
} from 'react-navigation';
```

and remove `export default` from the following line

```jsx
export default class App extends Component
```

If we run the app, we won't see any difference, but our application now using React Navigation with a single screen under the hood.
As the next steps, we need to break our component into separate screens.


## Splitting screens

First, we need to describe screens we are going to use within the Switch Navigator.

```jsx
const AppNavigator = createSwitchNavigator({
  App: App,
  Practice: PracticeScreen,
  Results: ResultsScreen
});
```

As you can see, we've described two new routes `Practice` and `Results` and link them to the corresponding components.
As a convention, we're adding 'Screen' suffixes to those components, to indicate that they are screens.

The screens `PracticeScreen` and `ResultsScreen` do not exist yet.
So let's create them one by one.

At the top of the `App.js` file, import the screens we are about to create.

```jsx
import PracticeScreen from './src/screens/PracticeScreen';
import ResultsScreen from './src/screens/ResultsScreen';
```

Then, create an `src/screens` directory with the `ResultsScreen.js` file inside.
We need to create a `ReactScreen` component there and move the content of our `renderResultsScreen()` function, as well as import,  required components, and styles.


```jsx
import React from 'react';
import { Button, View, Text } from 'react-native';
import styles from '../styles';

export default class ResultsScreen extends React.Component {
  render() {
    return (
      <View style={styles.container}>
        <View>
          <Text style={styles.welcome}>Results</Text>
          <Text style={styles.results}>
            Words count: {this.state.totalWords}
          </Text>
          <Button
            onPress={this.onPressPractice}
            title="Practice Again"
          />
        </View>
      </View>
    );
  }
}
```

After we extracted the screen from the `App` component, some functionality which we used to have won't work in that screen.

First missing part is that we don't have `this.state.totalWords` here.
To fix that, we are going to pass that value when we are about to navigate to that screen.
To read params in a screen component React Navigation provides the following functionality:

```
this.props.navigation.getParam(paramName, defaultValue)
```

It tries to fetch a value from `paramName` and fallbacks to `defaultValue` if there isn't any.

Let's replace the `this.state.totalWords` to `this.props.navigation.getParam('totalWords', 0)`.
We will pass that value from the `PracticeScreen` once we get to it.

The next missing part is that the `this.onPressPractice` function does not exist in `ResultsScreen` component.

On the one hand, we can try to copy and adjust that functions' content from the `App` component, but that forces us to duplicate the logic of that method.
We have the similar button on the welcome screen already and we need to keep it.

On the other hand, all we need that button to do is to navigate to the `PracticeScreen` and handle all the practice related logic there.

To navigate between screens we are using `this.props.navigation.navigate(routeName)` function from React Navigation.

Replace

```jsx
this.onPressPractice
```

to

```jsx
() => this.props.navigation.navigate('Practice')
```

That's all we need to change to make `ResultsScreen` work.


Now, let's move on to `PracticeScreen`.
In the same way, as before, create a `scr/screens/PracticeScreen.js` file.
Define a `PracticeScreen` component inside and set `render()` body to be the body of `renderPracticeScreen()`:


```jsx
import React from 'react';
import { Button, View, Text } from 'react-native';
import styles from '../styles';

export default class PracticeScreen extends React.Component {
  render() {
    return (
      <View style={styles.container}>
        <View>
          <Text style={styles.word}>{this.state.currentWord}</Text>
          <Button
            onPress={this.onPressNextWord}
            title="Next Word"
          />
        </View>
      </View>
    );
  }
}
```

In that chunk of code, we are missing some working parts as well.
This time, we are going to tackle them in a different way.

The `PracticeScreen` is a core screen of the application and will keep all the practice logic.

The button's event `onPress` stays the same.
In order to make it work, we need to move `onPressNextWord()` function from the `App` controller here.

```jsx
onPressNextWord() {
  const { words, totalWords } = this.state;
  const nextWord = words.shift()

  this.setState({
    currentWord: nextWord,
    words,
    totalWords: totalWords + 1
  })
}
```

We don't even need to change the function. It stays the same.
But we need to apply a couple of changes to make it work.

Next, we need to define a constructor for the component.

```jsx
constructor(props) {
  super(props)

  this.state = { totalWords: 0 }

  this.onPressNextWord = this.onPressNextWord.bind(this);
}
```

There, we bind the current context and define an initial state.

Now, all the logic related to practice was triggered inside `onPressPractice()` function by pressing a button.
After we split the `App` into screen components, that approach can't work anymore.
We need to trigger practice in a different way when the user navigates to the practice screen.
How do we do that?

React provides us with a bunch of lifecycle callbacks.
One of those callbacks is [`componentDidMount`](https://reactjs.org/docs/react-component.html#componentdidmount).

> `componentDidMount()` is invoked immediately after a component is mounted (inserted into the tree)

Thus, let's define that callback and move the body of `onPressPractice()` function from the `App.js` file.


```jsx
componentDidMount() {
  setTimeout(() => (
    this.setState({ currentScreen: 'results' })
  ), PRACTICE_TIME);

  const words = shuffle(allWords);
  const currentWord = words.shift();

  this.setState({
    currentScreen: 'practice',
    currentWord,
    words,
    totalWords: 0
  });
}
```

Now, let's adjust the body to make it work.

We can drop `currentScreen: 'practice'` line, we don't need it anymore.

Then, we need to define `PRACTICE_TIME` constant. Let's move it from the `App.js` file.

Now, as soon as we are not using the state to navigate between screens anymore, we need to replace the following line:

```jsx
this.setState({ currentScreen: 'results' })
```

with

```jsx
this.props.navigation.navigate('Results', {
  totalWords: this.state.totalWords
})
```

What we're doing here, is navigating to results screen once the timer runs out.
Also, we are passing the `totalWords` parameter, with the value from the practice component's state.
That allows us to render the total amount of words during the last practice session.

You can read more about passing and reading parameters on a [documentation page](https://reactnavigation.org/docs/en/params.html).

As the last step to make that screen work, we need to import `allWords` and `shuffle` functions.
Paste those lines at the top of the file:

```jsx
import allWords from '../words.en.json';
import shuffle from '../shuffle';
```

The last screen we need to work on is a welcome screen.

After we extracted all the other screens from the `App` component, it can play the role of the welcome screen.
All that it does now (or should do) is display a welcome message and a start practice button.

Thus, we need to move the body of `renderWelcomeScreen` to `render` method of the `App` component.
Then, we change the `onPress` event of the button to navigate to the practice screen.
Then, we clean up all leftovers we didn't remove yet.

That's how the `App` components looks now:

```jsx
import React, { Component } from 'react';
import { Button, Text, View } from 'react-native';
import {
  createSwitchNavigator,
  createAppContainer
} from 'react-navigation';
import styles from './src/styles';
import PracticeScreen from './src/screens/PracticeScreen';
import ResultsScreen from './src/screens/ResultsScreen';

class App extends Component {
  render() {
    return (
      <View style={styles.container}>
        <View>
          <Text style={styles.welcome}>Welcome to Blitz Reading!</Text>
          <Button
            onPress={() => this.props.navigation.navigate('Practice')}
            title="Practice"
          />
        </View>
      </View>
    );
  }
}

const AppNavigator = createSwitchNavigator({
  App: App,
  Practice: PracticeScreen,
  Results: ResultsScreen
});

export default createAppContainer(AppNavigator);
```

You can see an app example on the image below.

<p align="center">
  <img style="max-width: 50%" src="{{ site.url }}/img/posts/rn-navigation/01-app-example.gif" />
</p>

Although there are no visual changes, now the app is powered by React Navigation instead of keeping all the "screens" in a single `App` component.

## Wrapping up

React Navigation is a powerful tool.
It provides way more functionality we can cover in a single article.
Today we have covered only a single piece of that: Switch Navigator.
We've scratched the surface.
There are different types of navigation. For example Stack Navigator and Tab Navigator.

I'm going to describe them in one of the following articles.
When we'll be building a suitable functionality for them.

The source code of these changes you can find on [GitHub page](https://github.com/ck3g/BlitzReading/pull/2).

See you next time.

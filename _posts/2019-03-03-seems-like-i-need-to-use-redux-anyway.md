---
layout: post
title: "Seems like I need to use Redux anyway"
desc: "Learn what is Redux, what problems does it solve and how to use it in the React Native applications."
keywords: "React Native, Redux, React, Mobile development, iOS, Android, JavaScript"
tags: [ReactNative, BlitzReading]
excerpt_separator: <!--more-->
---

Over the course of [previous articles](http://whatdidilearn.info/tags#BlitzReading), we were building a BlitzReading app.
The initial idea was to build a simple application without using a lot of additional libraries, for example, Redux.

I thought as soon as the application is relatively small I can manage to implement it without using Redux.
But I've faced some issues.


<!--more-->

## The problem

The application has a High Scores screen where a user can see the list of high scores based on the previous practice sessions.
The can see a similar list on the results screen right after finishing practice.

<p align="center">
  <img style="max-width: 50%" src="{{ site.url }}/img/posts/rn-redux/01-problem1-no-highscores.gif" />
</p>

As you can see, the result screen displays the data, but the high scores screen still missing it.
In order to see the data, we need to re-render the component for example by updating its state.

One of the solutions might be is to trigger state update from the results screen.
Which is tricky and has its disadvantages.
The results screen, in that case, will have to know about high screens component and notify it.
Even though that is not its responsibility of results component.
That makes the codebase more tied together.

As one of the other solutions, we can store the high scores data in the Redux storage once the user reaches the results screen.
Then connect high scores screen with that store.
The high scores screen will read the data from the store (instead of its state) and update itself when needed.

## What is Redux

If terms Redux and storage are unfamiliar to you, now it's time to figure out what is it and how does it work.

[Redux](https://redux.js.org) allows us to keep an application state in a centralized place and access that state from any part of our application.
That means if we need to share some state between several components we don't need to manually pass that data between them.
We can read the state from the storage itself.

Three main parts of Redux are storage, actions, and reducers.

Storage is the place where we keep the application's state.

A state is read-only, that means we cannot update storage directly.
In order to update the state, we call actions.
Actions are plain objects which consist of type and payload.

Reducers are plain functions which receive a state and an action to transform the state into a new one.

Don't worry if that still seems puzzling. We are going to see all of these things in examples below.


## Using Redux in React Native

Using Redux with React Native does not differ from the usage to React.

We are going to fix the issue with high scores screen in the BlitzReading application.
The source code of which you can find on [the GitHub page](https://github.com/ck3g/BlitzReading).

First, we need to install `redux` and `react-redux` libraries to our project.

```
→ npm install --save redux react-redux
```

or

```
→ yarn add redux react-redux
```

Now, let's start by building small pieces one by one.
It should make more sense once we see the whole picture.


Let's start by adding an action.

Create a `src/actions/HighScoresActions.js` file with the following content.

```js
export const updateHighScores = (highScores) => {
  return {
    type: 'HIGHSCORES_UPDATED',
    payload: highScores
  }
};
```

and `src/actions/index.js` file as well

```js
export * from './HighScoresActions';
```

As I mentioned above, actions are plain objects. They contain a type and a payload.
The type is a string that plays the role of some sort of identifier, thus we can separate one action from another later in reducers.

When the user is about to reach the results screen we are going to call that action and pass new data as a payload.

Next, let's create a reducer to change a state.

Add the following code the `src/reducers/HighScoresReducer.js` file.

```js
const INITIAL_STATE = [];

export default (state = INITIAL_STATE, action) => {
  switch (action.type) {
    case 'HIGHSCORES_UPDATED':
      return [...state, ...action.payload];
    default:
      return state;
  }
};
```

We've described a reducer. It receives a state and an action.
Then we check for specific action type.
If the type the one we care about, we update the state and return it.
In our case, we are going to store high scores in an array.

A reducer function can handle several actions. We usually group them by common behavior.
The one we've created can handle actions related to high scores.
You can have a single reducer for the whole application if you want.
We split it here to organize the code in a better way.

Once a reducer is called we check the type of action and update the state according to it.

When any action is called, Redux calls every known reducer and pass that actions as an argument.
Then it is up to reducer to decide if the state should be updated or not.
Every reducer has to return some value even when an action type doesn't satisfy the condition.
That's why we should always have a default condition to return unchanged state.

We've created a reducer, but it just lies in a corner.
We need to combine our reducers together and connect to the application.
To do that, we need to call a `combineReducers` function from the `redux` library.

Create a `src/reducers/index.js` file with the following content.

```js
import { combineReducers } from 'redux';
import HighScoresReducer from './HighScoresReducer';

export default combineReducers({
  highScores: HighScoresReducer
});
```

In that file, we import the reducer we've just created.
Then we define our reducer to be responsible for the `highScores` part of the store.

Now we can connect combined reducers to our application.

The root of our application is `App.js` file.
Open the file and import a couple of helpers.

```js
import { Provider } from 'react-redux';
import { createStore } from 'redux';
import reducers from './src/reducers';
```

Next, replace

```jsx
export default createAppContainer(InitialNavigator);
```

with the following code

```jsx
const AppContainer = createAppContainer(InitialNavigator);

class App extends React.Component {
  render() {
    return (
      <Provider store={createStore(reducers)}>
        <AppContainer />
      </Provider>
    );
  }
}

export default App;
```

We need to have access to Redux store in all the components from our application.
To achieve that we need to wrap our root component with the `Provider` component.

As soon as our root component is an app container from React Navigation, we have to define a small App component which will have a top-level Provider component with the nested AppContainer.

We call `createStore` helper passing all our reducers as an argument.
The result of that function will a Redux store, which we pass to the Provider's store prop.

The app now has its own Redux store. It also has action and reducer.
Let's change our existing codebase to start using those pieces.

To use the store in our components we need to connect them to it.

Let's start from Results Screen.
Open `src/screens/ResultsScreen.js` and import the `connect` from `react-redux`.

```js
import { connect } from 'react-redux';
```

Next, we need to import the required actions.

```js
import { updateHighScores } from '../actions';
```

As the next step, we need to connect the current component to the store.

That's how we do that.

Replace

```jsx
export default ResultsScreen;
```

with

```jsx
const mapStateToProps = (state) => ({
  highScores: state.highScores
});

const mapDispachToProps = ({ updateHighScores });

export default connect(mapStateToProps, mapDispachToProps)(ResultsScreen);
```

Here we call the `connect` function which accepts two arguments.
In the first argument, we are mapping the values from the store's state to props to pass to the component.

In our case, that's the `highScores` prop, which contains all the high scores from the store.

The second argument contains all the actions we want to pass into the component.
In our case, it's `updateHighScores` action.

The `connect` function returns us another function, which we immediately call by passing our component as an argument.
Pay attention there are two pairs of parenthesis.

When the practice session is finished it renders the results component.
The component has a `componentDidMount` callback. It stores high scores to local storage.

```js
componentDidMount() {
  const totalWords = this.props.navigation.getParam('totalWords', 0);
  this.setState({ totalWords });

  this.updateHighScores(totalWords);
}
```

In addition to that, we want to call our `updateHighScores` action and pass a new record as a payload.

```js
this.props.updateHighScores([{ score: totalWords, createdAt: new Date() }]);
```
That call of the action behind the scenes will trigger calls of all connected reducers.
That will lead to updating data in the store.

Now, when the store is connected to the component and we the state of the high scores, we need to use that data to render on the screen.

Replace the following line

```jsx
<HighScores data={this.state.highScores} />
```

with

```jsx
<HighScores data={this.props.highScores} />
```

The results screen component is ready.

Next step, we need to that data in the high scores screen component.
Let's repeat some steps to make it happen.

Import the `connect` function in a `src/screens/HighScoresScreen.js` file.

```js
import { connect } from 'react-redux';
```

This time we don't need to import any actions, because we are not going to update the store from that component.
We need only to read it.

That is why the second argument of `connect` function will be an empty object.

```jsx
const mapStateToProps = ({ highScores }) => ({
  highScores
});

export default connect(mapStateToProps, {})(HighScoresScreen);
```

Replace `this.state.highScores`

```jsx
<HighScores data={this.state.highScores} totalNumber={25} />
```

with `this.props.highScores`

```jsx
<HighScores data={this.props.highScores} totalNumber={25} />
```

We can also remove `constructor`

```js
constructor(props) {
  super(props);

  this.state = { highScores: [] };
}
```

and remove `componentDidMount` callback.

```js
async componentDidMount() {
  try {
    let highScores = await fetchHighScores();

    this.setState({ highScores });
  } catch (error) {
    console.log('Error fetching High Scores', error);
  }
}
```

We don't need them anymore.


We've written a lot of code to connect all the pieces together.
Finally, we are able to check how does it work.

<p align="center">
  <img style="max-width: 50%" src="{{ site.url }}/img/posts/rn-redux/02-fixed-highscores.gif" />
</p>

As you can see, now our results and high score screens are in sync.

We can see that Redux is working with our application, but we still have a problem.
Once we reload the simulator all the data will be gone because the storage resets on each reloads.

Let's fix that.

## Populate data from AsyncStorage

Besides storing the data in Redux storage we also persist the data in local storage.
So all we need to do now is to update the store with the data from local storage.

We already render a [Splash Screen](http://whatdidilearn.info/2019/01/13/how-to-implement-a-splash-screen-in-react-native.html) when the app starts.
It seems like the right place to populate the store.

Open a `src/screens/SplashScreen.js` file and import the required functionality.

```js
import { connect } from 'react-redux';
import { fetchHighScores } from '../storage/highScoreStorage';
import { updateHighScores } from '../actions';
```

Next, update `componentDidMount` callback, fetch the high scores and call the action.

```js
async componentDidMount() {
  // ...
  const highScores = await fetchHighScores();

  if (highScores !== null) {
    this.props.updateHighScores(highScores);
  }
}
```

As usual, we need to connect our component to the Redux storage.

Replace

```js
export default SplashScreen;
```

with

```js
const mapDispatchToProps = ({ updateHighScores });

export default connect(null, mapDispatchToProps)(SplashScreen);
```

In this case, we don't read high scores from Redux, we only update them.
That's why pass `null` as a first argument. We don't need to map anything.

That's is pretty much it.
Now once we reload the app we will see the list of high scores preloaded from the Async Storage.


## Wrapping up

Let's see what did we learn today.

First, we took a look at one of the issues the application has.
The high scores screen wasn't properly populated with the fresh data.

Then we've fixed the problem by integrating Redux into the application.

We've learned what is Redux and its fundamentals.
We've learned about the store, actions, and reducers.
How to define them and how to connect them to our application and components.

That's for sure not a complete tutorial about Redux.
That topic is too big for a single article.

I wanted to show an example of the problem you might face, and how to fix it using Redux.

All the code you can find on [the GitHub page](https://github.com/ck3g/BlitzReading/pull/10) of the project.

See you next time.

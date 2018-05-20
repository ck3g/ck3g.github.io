---
layout: post
title: "How to use Webpack and React with Phoenix 1.3"
desc: "Learn how to configure Webpack and start using React for your Phoenix projects."
keywords: "Elixir, Phoenix, Webpack, React"
tags: [Elixir, Phoenix, Webpack, React]
excerpt_separator: <!--more-->
---

If you are building an app with the heavy focus on front-end.
You may want to use [Webpack](https://webpack.js.org/concepts/) to get some benefits out of it.
Especially if you also would like to add [React](https://reactjs.org/) to your project.

Webpack is quite popular nowadays.
Starting from version 1.4, Phoenix will support it out of the box for assets management.

What if we want to start using it now before Phoenix 1.4 is released.
That is not a problem. We can manually replace [Brunch](http://brunch.io/) with Webpack now.

Let's take a look how we can do that.

<!--more-->


## Configure Webpack

First, we need to remove all the Brunch configuration we already have in our project.

To demonstrate that, I will use the standard Phoenix application which uses Brunch.
So we can see that should we remove.

Let's get started.

As a first step, we should remove following libraries from the development dependencies of the `assets/package.json` file:

```
"babel-brunch"
"brunch"
"clean-css-brunch"
"uglify-js-brunch"
```

Now we are ready to install required dependencies:

→ cd assets
→ npm install --save-dev babel-core babel-loader babel-preset-env
→ npm install --save-dev webpack webpack-cli
→ npm install --save-dev copy-webpack-plugin css-loader mini-css-extract-plugin optimize-css-assets-webpack-plugin uglifyjs-webpack-plugin
→ cd ..

By using `--save-dev` would add those dependencies to `assets/package.json` file for us.

Let's open the file again and change.

```json
"deploy": "brunch build --production",
"watch": "brunch watch --stdin"
```

to

```json
"deploy": "webpack --mode production",
"watch": "webpack --mode development --watch"
```

We would need that soon.

Now, remove the `assets/brunch-config.js` file.

Instead we need to create `assets/webpack.config.js` with the following content:

```js
const path = require('path');
const MiniCssExtractPlugin = require('mini-css-extract-plugin');
const UglifyJsPlugin = require('uglifyjs-webpack-plugin');
const OptimizeCSSAssetsPlugin = require('optimize-css-assets-webpack-plugin');
const CopyWebpackPlugin = require('copy-webpack-plugin');

module.exports = (env, options) => ({
  optimization: {
    minimizer: [
      new UglifyJsPlugin({ cache: true, parallel: true, sourceMap: false }),
      new OptimizeCSSAssetsPlugin({})
    ]
  },
  entry: './js/app.js',
  output: {
    filename: 'app.js',
    path: path.resolve(__dirname, '../priv/static/js')
  },
  module: {
    rules: [
      {
        test: /\.js$/,
        exclude: /node_modules/,
        use: {
          loader: 'babel-loader'
        }
      },
      {
        test: /\.css$/,
        use: [MiniCssExtractPlugin.loader, 'css-loader']
      }
    ]
  },
  plugins: [
    new MiniCssExtractPlugin({ filename: '../css/app.css' }),
    new CopyWebpackPlugin([{ from: 'static/', to: '../' }])
  ]
});
```

Add `import css from "../css/app.css"` on top of the `assets/js/app.js` file

In the `config/dev.exs` change

```elixir
watchers: [node: ["node_modules/brunch/bin/brunch", "watch", "--stdin",
```

to

```
watchers: [node: ["node_modules/webpack/bin/webpack.js", "--mode", "development", "--watch-stdin",
```


Create a `assets/.babelrc` file with the following content:

```
{"presets": ["env"]}
```

Or any other options suitable for you. Which you can find on [the library's webpage](https://github.com/babel/babel/tree/master/packages/babel-preset-env#install)

#### A side note

I've intentionally didn't include `assets/css/phoenix.css` into the `assets/css/app.css` file.

The reason is that the styles which Phoenix 1.3 provides us by default use "glyphicons" fonts.
But those fonts are not provided by Phoenix. That was not a problem before, because can't see any errors related to missing fonts until we try to use them.
That changes with Webpack. When Webpack processes that file and tries to build a dependency graph, it cannot find those fonts and raises an error.

That is not that easy to get rid of CSS rules which use those fonts because a big chunk of the file is minified.

I guess, in most of the cases. If you have Phoenix application you already replaced built-in styles with your own or something else like [Bootstrap](http://whatdidilearn.info/2018/02/11/how-to-use-bootstrap-4-with-phoenix.html) or [Material Design](http://whatdidilearn.info/2018/05/13/how-to-use-materialize-css-with-phoenix.html).

Anyway, for the sake of example, we don't need those styles.

That is pretty much it. Now if you start the web server the project all the styles and JavaScript should work as before.

Now we can proceed and configure React for our project.


## Configure React

First, let's install additional dependencies.

```
→ cd assets
→ npm install --save react react-dom
→ npm install --save-dev babel-preset-react
→ cd ..
```

We need "react" and "react-dom", well, for React. And we need "babel-preset-react" to be able to use JSX in our JavaScript files.

Extend presets `assets/.babelrc` to have `"react"` as well.

```
{"presets": ["env", "react"]}
```

As soon as we are not using any styles yet. Let's add single CSS rule just for the sake of example.

`assets/css/app.css`

```css
.jumbotron {
  background: #eee;
  text-align: center;
  padding: 50px 0 50px;
}
```

Create `assets/js/components/Jumbotron.js`

```js
import React from "react";

const Jumbotron = () => {
  return (
    <div className="jumbotron">
      <h2>Welcome to Phoenix with Webpack and React</h2>
      <p className="lead">
        A productive web framework that<br />
        does not compromise speed and maintainability.
      </p>
    </div>
  )
};

export default Jumbotron;
```

in the `assets/js/app.js`

add

```js
import React from "react";
import ReactDOM from "react-dom";

import Jumbotron from './components/Jumbotron';

ReactDOM.render(<Jumbotron />, document.getElementById("react-app"));
```

The last thing we need to do is to add that "react-app" element on the page.

In the `lib/<project_name>_web/templates/page/index.html.eex` replace everything with

```html
<div id="react-app"></div>
```

We did it. Start the Phoenix server again and you should see "Welcome to Phoenix with Webpack and React" message on the page.
That means our React component works.

# Wrapping up

That concludes the configuration of Webpack and React for Phoenix projects.
Now if you want, you can start your React applications before the release of Phoenix 1.4.

You can check the live example [here](https://phoenix-webpack-example.herokuapp.com/) and all the changes on [GitHub](https://github.com/ck3g/phoenix-webpack-example).

See you next time.

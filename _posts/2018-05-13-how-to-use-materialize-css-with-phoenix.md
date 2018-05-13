---
layout: post
title: "How to use Materialize CSS with Phoenix"
desc: "Learn how to implement Material Design in Phoenix applications."
keywords: "Elixir, Phoenix, Material Design, Materialize"
tags: [Elixir, Phoenix]
excerpt_separator: <!--more-->
---

Some people like using Bootstrap in their projects, some do not.

*If you prefer Bootstrap, you can read ["How to use Bootstrap 4 with Phoenix"](http://whatdidilearn.info/2018/02/11/how-to-use-bootstrap-4-with-phoenix.html))*

Is there another way to build your web applications using powerful template library? Of course, it is.
One of the solutions would be to adopt [Material Design](https://material.io/) by Google.
Let's take a look how we can do that.

<!--more-->

Let's start with greenfield Phoenix application and integrate Material Design using front-end framework called ["Materialize"](https://materializecss.com/).
Once we have our new Phoenix application, we can remove `assets/css/phoenix.css`.
We won't need it.
In addition, I've put some examples from the Materialize page just see more element on that page later.
After those preparations, we are ready to add the library to our project.

## Use CDN

The first way we will consider is to use Content Delivery Network (CDN) to get minified CSS and JS files.

The advantage of using CDN is that the files are distributed all over the world and the users of your application will access the closest CDN network to them.
That helps them to get those files much faster.

You can face the disadvantage of that approach during development. If you have no internet on your development computer, you won't be able to load those files and the styles and some functionality of your application will be broken.

You would need to weigh those pros and cons and decide if that approach suitable for you.

To start using CDN we need to update our "Layout" file `lib/phoenix_materialize_example_web/templates/layout/app.html.eex` and include CSS file into `<head>` right before existing `css/app.css` file.

```erb
<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/materialize/1.0.0-beta/css/materialize.min.css">
<link rel="stylesheet" href="<%= static_path(@conn, "/css/app.css") %>">
```

And the similar for JavaScript. We need to include it before the closing </body> tag. Also before `js/app.js` script.

```erb
<script src="https://cdnjs.cloudflare.com/ajax/libs/materialize/1.0.0-beta/js/materialize.min.js"></script>
<script src="<%= static_path(@conn, "/js/app.js") %>"></script>
```

That's it. Now if you start the web server that should work.

Here is how does it look for my application.

<p align="center">
  <a href="https://phoenix-materialize-example.herokuapp.com/">
    <img src="{{ site.url }}/img/posts/materialize/layout-example.png" />
  </a>
</p>

[Here](https://github.com/ck3g/phoenix_materialize_example/pull/1) you can find all the required changes.

## Use NPM

If by any chance you've decided not to use CDN. That's no problem. You can use NPM instead.

NPM is the package manager for JavaScript. It can be used to install many libraries.

Let's try to setup Materialize CSS.

First, we need to navigate to our assets directory:

```
→ cd assets
```

And install Node.js package. Using `--save` will also add the package into `assets/package.json` file into `dependencies` section.

```
→ npm install materialize-css@next --save
```

The package provides us SASS source files. In order to use them in our project we also need to add `sass-brunch` as a development dependency, by running the following command:

```
→ npm install sass-brunch@2.10 --save-dev
```

The next step is to update the `assets/brunch-config.js` file. We need to extend the `files.stylesheets` section with:

```js
order: {
  after: ["priv/static/css/app.scss"]
}
```

add following lines into `plugins` section

```js
sass: {
  options: {
    includePaths: ["node_modules/materialize-css/sass"],
    precision: 8
  }
}
```

and extend `npm` section with the following configuration:


```js
globals: {
  materialize: 'materialize-css'
}
```

The last thing we need to rename app.css to app.scss

```
→ cd ..
→ mv assets/css/app.css assets/css/app.scss
```

and import Materialize CSS in that file

```sass
@import "materialize";
```

After all those steps, we can run a web-server. We should see the Materialize CSS working.

All the required changes for NPM you can find in [the following Pull Request](https://github.com/ck3g/phoenix_materialize_example/pull/2).


## Download minified files

Almost always there is yet another option. And Materialize CSS is not an exception.

If you like neither CDN nor NPM approach, you can download minified files and use them as regular CSS files in your project.


## Wrapping up

Today we took a look at another template library.
If your bored by Bootstrap and think your project should have a new look then you may consider using Material Design via Materialize CSS.

Checkout the live [Demo](https://phoenix-materialize-example.herokuapp.com/).

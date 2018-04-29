---
layout: post
title: "Deploying Phoenix apps to Heroku"
desc: "Learn how to deploy your Elixir and Phoenix application to Heroku"
keywords: "Elixir deploy, Phoenix deploy, Elixir Heroku, Phoenix Heroku"
tags: [Elixir, Phoenix, Deploy]
excerpt_separator: <!--more-->
---

In the previous articles, we have been building the Prater chat app.
It is a time to deploy it now.

There is a bunch of different ways to deploy Phoenix applications.

I would like to start from the simplest approach and deploy the app to [Heroku](https://www.heroku.com).

For those who are not familiar with Heroku. It is a cloud-based Platform as a Service.
It allows deploying web applications for many different programming languages.
Once you configure the deployment for your app, every new deploy would be triggered by simply run `git push heroku master`.

That is why it's so easy to use Heroku, especially for demo and prototype apps.

Let's get started.

<!--more-->

In order to configure the deployment process to Heroku, we would need to have several tools in our pocket.

### Requirements

* Git - I suppose you already have it
* Heroku account - You can create it [here](https://signup.heroku.com/) if you don't have it.
* [Heroku CLI](https://devcenter.heroku.com/articles/heroku-cli)
* The Phoenix app - The app we want to deploy. I will use the [Prater](https://github.com/ck3g/prater) app for that.

Once you have everything ready, make sure you are signed in, by running

```
→ heroku login
```

in your terminal window.

Also, navigate to the directory with your project you want to deploy.

### Create Heroku app

Now we have everything we need to start.

At first, we need to create a Heroku application. We do it by running the following command.

```
→ heroku apps:create prater-app --buildpack "https://github.com/HashNuke/heroku-buildpack-elixir.git"
```

We can optionally pass the name of the application. I am using "prater-app" for that.
If you won't provide the name, the Heroku will assign a random name to your application.

We should specify the buildpack for the application.

Buildpacks are basically a set of scripts which prepares your application and helps to deploy it to Heroku.
There is no official Elixir buildpack at the moment. So we should use a custom one.

As soon as we have static assets in our application, we need to use an additional buildpack in order to compile them.

```
→ heroku buildpacks:add https://github.com/gjaldon/heroku-buildpack-phoenix-static.git
```

Now, almost every application uses the external dependencies in order to work. One of those dependencies is most likely a database.
We have it as well.

In order to work with dependencies, Heroku provides the [Add-ons tooling](https://elements.heroku.com/addons). Where we can install those dependencies as Add-ons to our Heroku applications.

We need [an add-on for PostgreSQL database](https://elements.heroku.com/addons/heroku-postgresql) and we are going to use free "Hobby Dev" plan.

```
→ heroku addons:create heroku-postgresql:hobby-dev
```

The "Hobby Dev" plan has a limit for the number of DB connections set to 20. We need to stay below that limit.
So let's define `POOL_SIZE` environment variable for later use.

```
→ heroku config:set POOL_SIZE=18
```

The Phoenix application also requires us to provide the "secret key base" token in order to securely verify the integrity of signed cookies.

We don't need to think of ways to create that string, because the `phx.gen.secret` helps us to generate that.

Let's define an environment variable for that:

```
→ mix phx.gen.secret
→ heroku config:set SECRET_KEY_BASE="`mix phx.gen.secret`"
```

### Configure config files

Our Heroku application is almost ready. Now we need to update our config files to make our Phoenix app work after deploy.

First, we need to update `config/prod.exs` file.

For our `Endpoint` configuration we need to update the `url` to contain: `scheme` as "https", `host` to be the host of your Heroku app (mine is `prater-app.herokuapp.com`) and `port` to be `443`.
Then we need to create `force_ssl` with `[rewrite_on: [:x_forwarded_proto]]` and fetch `secret_key_base` from the environment variable.

Here is how that section should look:


```elixir
config :prater, PraterWeb.Endpoint,
  load_from_system_env: true,
  url: [scheme: "https", host: "prater-app.herokuapp.com", port: 443],
  cache_static_manifest: "priv/static/cache_manifest.json",
  force_ssl: [rewrite_on: [:x_forwarded_proto]],
  secret_key_base: Map.fetch!(System.get_env(), "SECRET_KEY_BASE")
```

Then we need to create a new section for our `Repo` configuration:

```elixir
config :prater, Prater.Repo,
  adapter: Ecto.Adapters.Postgres,
  url: System.get_env("DATABASE_URL"),
  pool_size: String.to_integer(System.get_env("POOL_SIZE") || "10"),
  ssl: true
```

Heroku defines `DATABASE_URL` environment variable for us which holds the connection to the database. So here we just use it.
And we fetch the `POOL_SIZE` we have defined earlier.

The last piece in that file, we need to remove `import_config "prod.secret.exs"` at the bottom. We don't need it anymore.

Now we are moving to `channels/user_socket.ex` file.
Here we need to set the timeout for idle Phoenix connections. We need to do that to prevent Heroku killing them after 55 seconds.

```elixir
transport :websocket, Phoenix.Transports.WebSocket,
  timeout: 45_000
```

Next, we need to create a `Procfile` in the project root directory with the following content:

```
web: MIX_ENV=prod mix phx.server
```

As the last step, we can configure Erlang and Elixir versions. That can be done in the `elixir_buildpack.config` file.
Let's create one with the following content:

```
erlang_version=20.0
elixir_version=1.6.0
```

The [Buildpack](https://github.com/HashNuke/heroku-buildpack-elixir) we use, provides a bunch of other options as well.

Now everything is ready for deploy. After the update of the configs, we need to create a new commit with those changes.

### Deploy the app

To trigger out first deploy we can run:

```
→ git push heroku master
```

or, if you're experimenting in a separate git branch, as I do, you need to run

```
→ git push heroku <your-branch-name>:master
```

you can go back to `git push heroku master` once you finish your work and merge the stuff into the "master" branch.

If everything went fine, at the end of the output you should see something similar to that:

```
remote:        https://prater-app.herokuapp.com/ deployed to Heroku
remote:
remote: Verifying deploy... done.
```

No errors, now we need to migrate the database:

```
→ heroku run "POOL_SIZE=2 mix ecto.migrate"
```

<hr />

If you have the following error, when trying to run the migrations:


```
[error] GenServer #PID<0.227.0> terminating
** (RuntimeError) Connect raised a ArgumentError error. The exception details are hidden, as
  they may contain sensitive data such as database credentials.

(postgrex) lib/postgrex/utils.ex:67: anonymous fn/1 in Postgrex.Utils.parse_version/1
```

You may need to update "Postgrex" dependency to fix that issue

```
→ mix deps.update postgrex
```

Then commit your changes, push them to the Heroku and try to run Ecto migration again.

<hr />

If everything went fine, try to open the app in the browser and check if everything works.

## Wrapping up

We have learned how to deploy Phoenix applications to Heroku.
The reason why I choose the Heroku as the first example is to try something simple before move on.

Heroku might be not a best chose for hosting Elixir applications.
It comes with [the bunch of limitations](https://hexdocs.pm/phoenix/heroku.html#limitations).
So you need to take them into account if you decide to use Heroku for your apps in production.

But Heroku is a good example just to try it out.

See you next time.

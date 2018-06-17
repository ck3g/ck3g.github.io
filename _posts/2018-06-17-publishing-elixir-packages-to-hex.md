---
layout: post
title: "Publishing Elixir packages to Hex.pm"
desc: "Learn how to develop and publish your Elixir packages and publish them to Hex.pm"
keywords: "Elixir, Hex, Packages, Publish, Libraries"
tags: [Elixir, Hex.pm]
excerpt_separator: <!--more-->
---

When you are developer and work on different projects,
sooner or later you would face a need to extract some part of functionality into separate libraries to reuse it.

Different languages and frameworks provide their own functionality to distribute libraries among other developers or within a single company.

You probably already know some of them. "npm" is a package manager for JavaScript. For Ruby, it would be "Ruby Gems".
Elixir is not an exception here. For Elixir and Erlang, we have ["Hex"](https://hex.pm/) as a package manager.

So, let's see how we can publish our own Elixir package.

<!--more-->

To publish a package we need to have one.
In my example, I would use a package called ["Stash Exchange"](https://github.com/ck3g/stash_exchange) which I've just created.

It has a single function `exchange/1`. You pass a value into that function to store it.
The function returns the previously stored value and updates the storage with the new value.

Here you can see an implementation of that function:

```elixir
@initial_state 1
@stash_name __MODULE__

def exchange(value) do
  if is_nil(Process.whereis(@stash_name)) do
    Agent.start(fn -> @initial_state end, name: @stash_name)
  end

  Agent.get_and_update(@stash_name, fn state -> {state, value} end)
end
```

Now let's try to publish that package.

## Register a new Hex user

In order to publish packages to Hex, we need to have an account there.
If you already have one, then you can run `mix hex.user auth` from your terminal window to authenticate.

If you are new to Hex then you can register either by using web form (https://hex.pm/signup) or using the command
line by calling `mix hex.user register` and follow the instructions.

## Configure the project metadata

Before we publish a package to Hex we need to provide some required attributes such as name, description, list of licenses etc.

Before you pick the name for your package, make sure the name is available.
You can either review [list of packages](https://hex.pm/packages) or search by the name you have already in mind.

There are some attributes I've added for Stash Exchange package.
First, in the `project` attributes we need to have a description and optional source code URL.
We also need to describe `package` attribute with extended information. In my case, that's `name`, `licenses`, `maintainers` and `links`.

```elixir
def project do
  [
    # ...
    description: "The single value storage",
    source_url: github_link(),
    package: package()
  ]
end

defp package() do
  [
    name: "stash_exchange", # Optional if we want to keep OTP app name
    maintainers: ["Vitali Tatarintev"],
    licenses: ["MIT"],
    links: %{"GitHub" => github_link()}
  ]
end

defp github_link() do
  "https://github.com/ck3g/stash_exchange"
end
```

You can always refer to `mix help hex.publish` to get the list of available attributes.


## Publishing and documentation

Once our metadata is ready we are ready to publish the package.

Packages are being published by using `mix hex.publish`.

The Hex tool will also generate documentation for the package by calling `mix docs`.
[ExDoc](https://github.com/elixir-lang/ex_doc) package provides that command, so it would be nice to have it as a dependency for the publishable package.
You are free to use any other library to generate docs. The documentation will be published on [hexdocs](https://hexdocs.pm/).

Check out [Writing documentation in Elixir](http://whatdidilearn.info/2017/10/23/writing-documentation-in-elixir.html) to learn more about writing and generating docs.

You can also publish or update the package without documentation or even update only documentation by calling
`mix hex.publish package` or `mix hex.publish docs` accordingly.


The `mix help hex.publish` command helps to remember those options as well as others.

Now, after we have cleared up the things around documentation we are ready to publish the package.

```
→ mix hex.publish
Publishing stash_exchange 0.1.0
  App: stash_exchange
  Name: stash_exchange
  Files:
    lib/stash_exchange.ex
    mix.exs
    README.md
    LICENSE.md
  Description: The single value storage
  Version: 0.1.0
  Build tools: mix
  Licenses: MIT
  Maintainers: Vitali Tatarintev
  Links:
    GitHub: https://github.com/ck3g/stash_exchange
  Elixir: ~> 1.6
Before publishing, please read the Code of Conduct: https://hex.pm/policies/codeofconduct
Proceed? [Yn] Y
Building docs...
Docs successfully generated.
View them at "doc/index.html".
Passphrase:
Publishing package...
[#########################] 100%
Package published to https://hex.pm/packages/stash_exchange/0.1.0 (2472ac58ec48ce1a8d720a508a9d4d14c9a6a46e22988c5b92437dcae4d0b63b)
Publishing docs...
[#########################] 100%
Docs published to https://hexdocs.pm/stash_exchange/0.1.0
```

The passphrase is actually your password. It was not obvious for me for the first time.
Also, the publish command didn't work for me in a separate terminal window.
You might need to run `mix hex.user auth` to re-authenticate yourself.


That is it. The package has been published and we can navigate to the package page to check it out.
In my case, that's a [https://hex.pm/packages/stash_exchange/0.1.0](https://hex.pm/packages/stash_exchange/0.1.0) page.

## Use the package as a dependency

The package was published and we can start using it.

Let's create a simple Elixir app, install that package as a dependency and try it out.

```
→ mix new stash_exchange_proxy
```

First, we need to add it as a dependency in the `mix.ex` file:

```elixir
defp deps do
  [{:stash_exchange, "~> 0.1.0"}]
end
```

Then we define a simple proxy function to call the `StashExchange.exchange/1` function:

```elixir
defmodule StashExchangeProxy do
  def exchange(value) do
    StashExchange.exchange(value)
  end
end
```

Now we need to install dependencies

```
→ mix deps.get
```

That is it. We are ready to play with that function in the Interactive Elixir session.

```elixir
iex> StashExchangeProxy.exchange("Elixir")
1

iex> StashExchangeProxy.exchange(%{awesome: true})
"Elixir"

iex> StashExchangeProxy.exchange("?")
%{awesome: true}
```

It seems to be working.


## Wrapping up

We've just published our first Elixir package. As we can see that is extremely easy to do.
Elixir and Hex provide us with all required tools to do that.

Now it is your turn.
Do you have the functionality you would like to share between your projects?
You can extract it and publish as a separate package.

Even if you don't want to make your packages public.
Hex allows publishing private packages as well. Check out ["Private packages"](https://hex.pm/docs/private) page to learn more.

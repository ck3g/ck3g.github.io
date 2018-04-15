---
layout: post
title: "Phoenix: UI tests using Hound"
desc: "Configure Hound to write UI and Integration tests for Phoenix applications."
keywords: "Automated tests, ui tests, integration tests, channel tests, Elixir, ExUnit"
tags: [Elixir, Phoenix, Testing]
excerpt_separator: <!--more-->
---


Let's talk about User Interface (UI) testing.

In the ["Introduction to testing"](http://whatdidilearn.info/2018/03/25/introduction-to-testing.html) article we have covered briefly what UI tests are standing on top of test pyramid. They cover several parts of functionality at once and check how do they work together.

Today we will see them in action.

<!--more-->

<hr />
This article is the part of the testing in Elixir series:

* [Part 1: Introduction to testing](http://whatdidilearn.info/2018/03/25/introduction-to-testing.html)
* [Part 2: Testing Models and Controllers](http://whatdidilearn.info/2018/04/01/testing-phoenix-models-and-controllers.html)
* [Part 3: Testing Channels](http://whatdidilearn.info/2018/04/08/testing-phoenix-channels.html)
* Part 4: User Interface testing
<hr />


In the previous articles, we have managed to write all the tests using built-in functionality of Elixir and Phoenix.
Now we are going to get outside of that circle because neither Elixir nor Phoenix does not provide us a functionality to write UI tests.

So what do we do? We are going to use the external library called ["Hound"](https://github.com/HashNuke/hound).
Hound is the library which is targeted specifically to writing that kind of tests.

So let's get started.

## Configure Hound

Before we write our first test we need to install and configure Hound to work on our project.

I will be following [Hound's setup](https://github.com/HashNuke/hound#setup) guide.

As a first step, we need to install hound as a dependency for our project in the `mix.esx` file.

```elixir
defp deps do
  [
    # ...
    {:hound, "~> 1.0", only: :dev}
  ]
end
```

Then we can install dependencies.

```
mix deps.get
```

now, in the `test/test_helper.exs` we need to add the following line before we start `ExUnit`.

```elixir
Application.ensure_all_started(:hound)

ExUnit.start()
```

also in the `config/test.exs` we need to set `server` value to `true`, to ensure that the server is started when we run the tests.

```elixir
config :prater, PraterWeb.Endpoint,
  http: [port: 4001],
  server: true
```

### PhantomJS

To run the UI tests we need a browser. There is a way to run the browser in the background is to use of available webdrivers. One of them called PhantomJS.

Now add

```
config :hound, driver: "phantomjs"
```

to the `config/config.exs` file.

What is PhantomJS?

> PhantomJS is a headless WebKit scriptable with a JavaScript API. It has fast and native support for various web standards: DOM handling, CSS selector, JSON, Canvas, and SVG.

From [PhantomJS.org](http://phantomjs.org)

Now install it via "homebrew" or using any other available way described on the official page http://phantomjs.org/download.html


## Back to tests

We have prepared everything we need. Now, let's start to write our first UI test.

Create the `test/prater_web/features/welcome_page_test.exs` file with the following content.

```elixir
defmodule PraterWeb.WelcomePageTest do
  use PraterWeb.ConnCase
  use Hound.Helpers

  hound_session()

  test "page has a correct title" do
    navigate_to("/")

    assert page_title() == "Hello Prater!"
    assert String.contains?(page_source(), "Welcome to Prater")
  end
end
```

Let's take a look what do we have here.

First, we define a test suite module. In the same way, we did it for every other test.
Then we are using `ConnCase` module provided us by Phoenix and `Hound.Helpers` module. Last one we get from the Hound library of course.
Then we are calling `hound_session` function to start the Hound session before we run our tests and destroy it afterward.

Inside our test case. We are navigating to the root page.
Once we are on the page we want to check that page title is the one we expected it to be. Also, we are checking that the page contains "Welcome to Prater" text.

The simplest test. Open the page, check the text.

Now let's try to run that. But before we do, there is a small caveat.

In order to keep our test working, we have to keep PhantomJS running (in the separate terminal window) all the time.


We run it in the following way:

```
→ phantomjs --wd
```

Only now we can run `mix test`. Otherwise, you will get the following error.

```
** (RuntimeError) could not create a new session: econnrefused, check webdriver is running
   (hound) lib/hound/session_server.ex:101: Hound.SessionServer.create_session/2
   (hound) lib/hound/session_server.ex:78: Hound.SessionServer.handle_call/3
   (stdlib) gen_server.erl:636: :gen_server.try_handle_call/4
   (stdlib) gen_server.erl:665: :gen_server.handle_msg/6
   (stdlib) proc_lib.erl:247: :proc_lib.init_p_do_apply/3
```

Once we have done all the steps, the test should pass.

OK, now we know the basics and how to run the Hound tests.
Let's write another more useful test. We are going to fill in form fields and submit the form.
As an example let's take the sign in form.

Create a new file `test/prater_web/features/sign_in_test.exs` with the following content

```elixir
defmodule PraterWeb.SignInTest do
  use PraterWeb.ConnCase
  use Hound.Helpers

  hound_session()

  test "user can sign in with valid credentials" do
    navigate_to("/")
    find_element(:link_text, "Sign In") |> click()

    assert current_path() == "/sessions/new"
  end
end
```

We already know that those first few lines do.
In our test, we are navigating to welcome page again, because we know it should contain the "Sign In" link.
As a next step, we are looking for that link and click it.
As a result, we should be navigated to the sign in page.

Let's check it now. The test should pass.

Now, let's proceed with the implementation. After the `assert` line, let's add following:

```elixir
fill_field({:id, "session_email"}, "user@example.com")
fill_field({:id, "session_password"}, "password")

find_element(:css, "button.btn-primary[type='submit']")
|> click()

assert String.contains?(page_source(),
  "You have successfully signed in!")
```

Using `fill_field` function we are (correct) filling in values into those fields.
Then we are using a `CSS` selector for find the submit button and click it.

As a result, we are expecting to see the flash message with "You have successfully signed in!" on the page.

Once we run the test, it will not pass.

```
→ mix test test/prater_web/features/sign_in_test.exs


  1) test user can sign in with valid credentials (PraterWeb.SignInTest)
     test/prater_web/features/sign_in_test.exs:7
     Expected truthy, got false
     code: assert String.contains?(page_source(), "You have successfully signed in!")
     stacktrace:
       test/prater_web/features/sign_in_test.exs:18: (test)
```

There are several ways to understand why the test is failing.
One of them would be to see what is happened on the page.
As soon as our "browser" runs in the background we can take advantage of one of the features provided by Hound.
Take a screenshot and see what do we have there.

To achieve that we can use `take_screenshot/1` function. Let's call it before the failing assert and run the test again.
It still failing but we have a screenshot in the project folder.

That how it looks like

<p align="center">
  <img src="{{ site.url }}/img/posts/ui_tests/invalid_email_or_password.png" />
</p>

We have got "Invalid Email or Password" error.

Of course. We have no users in our database. That is what we forgot to do. Let's create one.

At first let's import our [helpers](https://github.com/ck3g/prater/blob/829e56d67153bac73013c10eaa5b13b9a6fca049/test/support/auth_helpers.ex):

```elixir
import Prater.AuthHelpers
```

and then call `create_user()` the function from the imported module as the first line of our test.

That is how our test should look like:

```elixir
test "user can sign in with valid credentials" do
  create_user()
  navigate_to("/")
  find_element(:link_text, "Sign In") |> click()

  assert current_path() == "/sessions/new"

  fill_field({:id, "session_email"}, "user@example.com")
  fill_field({:id, "session_password"}, "password")

  find_element(:css, "button.btn-primary[type='submit']")
  |> click()

  assert String.contains?(page_source(),
    "You have successfully signed in!")
end
```

Now, if we run the test it should pass.

## Wrapping Up

That's it for today about UI testing.

Even though we have implemented just a few small examples, we have covered around 80% of important functionality we need to write integration tests using Hound. (Or any other library)
The UI tests are fairly simple by their nature. In most cases you need to know how to navigate from page to page, clicking links, fill in form fields and submit those forms.
Yes, there is more other stuff out there. Like working with javascript etc. But knowing those simple concepts will help you figure out how to write any of your tests.

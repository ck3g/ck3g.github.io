---
layout: post
title: "Using Credo to improve your Elixir code"
desc: "We are going to discover Credo. The code analysis tool to improve the codebase of your Elixir applications as well as learning more about Elixir itself."
keywords: "Elixir, Credo, Code analysis, analysis tool, refactoring"
tags: [Elixir]
excerpt_separator: <!--more-->
---

At the beginning of my Ruby on Rails career, I was using the ["Rails Best Practices"](https://rails-bestpractices.com/) code metric tool.
The intent of the tool is pretty simple.
It provides you suggestions how to improve your code. The best part of it, that on the https://rails-bestpractices.com website I was able to find detailed instructions why and how to fix those issues.

I love that tool. I was using it every day to check my code.
The cool part about those suggestions that I can go and read an explanation of that issue on [their website](https://rails-bestpractices.com/).
I was using the tool mostly for learning stuff.

Now it's time to learn some Elixir. And today I would like to discover a similar tool ["Credo"](https://github.com/rrrene/credo).
Credo focuses on teaching and code consistency.

<!--more-->

First, we can install Credo as a dependency to our project.
Then `mix credo` will be ready for our use.

Once we run that analysis tool against our codebase. Credo would provide us bunch of issues and suggestions on how to improve the codebase.

Credo splits those issues into the following categories:

* Code Readability
* Software Design
* Refactoring opportunities
* Warnings
* Consistency

To see the complete list with the description of every category we can run `→ mix credo categories`:

<p align="center">
  <img src="{{ site.url }}/img/posts/credo/categories.png" />
</p>

Now let's try to run it for our project.
For the examples here I would use the ["Prater"](https://github.com/ck3g/prater) application.
You might know about it from my previous posts.

First, let's run the default task and see the results.

```
→ mix credo
```

In my case, it provides me a list of 14 issues. I can see the summary of it at the very bottom of the output.

```
101 mods/funs, found 3 refactoring opportunities, 11 code readability issues.
```

We can also run it using strict mode.

```
→ mix credo --strict
101 mods/funs, found 3 refactoring opportunities, 25 code readability issues, 23 software design suggestions.
```

This time I've got the list of 51 issues. Looks like it's way more strict.

it does not make sense to go through every single issue in that article.
Thus, let's pick a single file and try to work with it.
A single file would be enough to walk through it and figure out what does every issue mean.

Here we can see what do we have after running default credo task for the `lib/prater/auth/auth.ex` file.

<p align="center">
  <img src="{{ site.url }}/img/posts/credo/auth_module.png" />
</p>

And now the results for the same file but with the `strict` mode.

<p align="center">
  <img src="{{ site.url }}/img/posts/credo/auth_module_strict.png" />
</p>

We can see, now we have "Software Design" category in addition to that we had before.

If we try to read through the issues, we can understand that do they mean.

For example "Modules should have a @moduledoc tag." is a pretty descriptive by itself.
On the other hand, "Pipe chain should start with a raw value." may require additional explanation.

To get more information about an issue we can run the credo and provide a filename and a line number of the issue.
You can see the line number right next to the issue description.
In my case it is the line number 21: `lib/prater/auth/auth.ex:21`

<p align="center">
  <img src="{{ site.url }}/img/posts/credo/auth_21_explain.png" />
</p>

We can see that we have way more information. Credo shows us the exact line of our code and bunch of suggestions.

The original piece of the code is

```elixir
User.registration_changeset(%User{}, params) |> Repo.insert()
```

Once we change it to

```elixir
%User{}
|> User.registration_changeset(params)
|> Repo.insert()
```

that issue will be gone.

For the sake of example, let's try to refactor one more issue. Here it is:

```
[F] → Cond statements should contain at least two conditions besides `true`.
      lib/prater/auth/auth.ex:8:5 #(Prater.Auth.sign_in)
```

The extended description says:

```
__ WHY IT MATTERS

   Each cond statement should have 3 or more statements including the
   "always true" statement. Otherwise an `if` and `else` construct might be more appropriate.

   Example:

     cond do
       x == y -> 0
       true -> 1
     end

     # should be written as

     if x == y do
       0
     else
       1
     end
```

Now look at the code:

```elixir
cond do
  user && Comeonin.Bcrypt.checkpw(password, user.encrypted_password) ->
    {:ok, user}
  true ->
    {:error, :unauthorized}
end
```

I think we can agree with Credo about this one. Let's update it to use `if/else` instead.


```elixir
if user && Comeonin.Bcrypt.checkpw(password, user.encrypted_password) do
  {:ok, user}
else
  {:error, :unauthorized}
end
```

And yet another issue fixed.


## Wrapping up

In the end, can we use Credo to learn something new about Elixir and how to format the code?
I think we can.

You can go even deeper and run the credo check from your CI tools.
If there are any issues Credo would fail with the non-zero exit status.
That can make your CI build to fail and force you and developers in your team to fix the issues.

In any case, I encourage you to try the tool against your code and check how can you improve it.
You can find more information about Credo on [the GitHub page](https://github.com/rrrene/credo).

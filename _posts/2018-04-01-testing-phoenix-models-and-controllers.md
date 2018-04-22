---
layout: post
title: "Testing Phoenix Models and Controllers"
desc: "Learn how to cover Phoenix Models and Controllers with tests."
keywords: "Automated tests, unit tests, integration tests, Elixir, ExUnit, models tests, controllers tests"
tags: [Elixir, Phoenix, Testing]
---


[Last time](http://whatdidilearn.info/2018/03/25/introduction-to-testing.html) we have learned about general testing concepts and have talked about unit testing in Elixir.
Now let's go further and learn how to add tests to Phoenix project.
We will learn how to write tests for Models and Controllers.

<hr />
This article is the part of the testing in Elixir series:

* [Part 1: Introduction to testing]({{ site.url }}/2018/03/25/introduction-to-testing.html)
* Part 2: Testing Models and Controllers
* [Part 3: Testing Channels]({{ site.url }}/2018/04/08/testing-phoenix-channels.html)
* [Part 4: User Interface testing]({{ site.url }}/2018/04/15/phoenix-ui-tests-using-hound.html)
* [Part 5: Property-based testing]({{ site.url }}/2018/04/22/property-based-testing.html)
<hr />

During the last months, we were building the ["Prater"](https://github.com/ck3g/prater) chat application.
Thus we have an existing functionality we can cover with tests now.


## Testing Models

Let's start with the Models (aka Schemas). Models are close to the unit tests.
We can even refer to them as unit tests.
Even though they are covering interaction with a DataBase, the scope of coverage is pretty narrow.

Let's get [`Prater.Auth.User` model](https://github.com/ck3g/prater/blob/c3f06e739b696856504b8419155e74bb3f6d5592/lib/prater/auth/user.ex) as an example and cover it with tests.

We have no test file for that model yet. So the first thing would be to create a `test/prater/auth/user_test.exs` file with the empty test suite.


```elixir
defmodule Prater.Auth.UserTest do
  use Prater.DataCase, async: true
end
```

If you are familiar with the ExUnit structure for test suite we [have covered last time](http://whatdidilearn.info/2018/03/25/introduction-to-testing.html#unit-testing).
You can notice that we are not using `use ExUnit.Case`. Instead, we are using `use Prater.DataCase` here defined in [`test/support/data_case.ex`](https://github.com/ck3g/prater/blob/c3f06e739b696856504b8419155e74bb3f6d5592/test/support/data_case.ex).
That module extends testing functionality in order to support test which requires access to data layer (DataBase).

We have also talked about `async: true` option in [the last article](http://whatdidilearn.info/2018/03/25/introduction-to-testing.html#testing-async) and find out that is a cool option which can increase the speed of our test suites.

### Note about an async option

But there is a pitfall when we are testing databases which we need to know.

By default, Phoenix uses the [`Ecto.Adapters.SQL.Sandbox`](https://hexdocs.pm/ecto/Ecto.Adapters.SQL.Sandbox.html) module.
Which basically wraps every test within a transaction in order to rollback it after a test is finished.
That helps to keep the test database clean.

[Phoenix guide](https://hexdocs.pm/phoenix/testing_schemas.html) does not recommend us to use `async: true` option if we are going to interact with the database:

> Note: We should not tag any schema case that interacts with a database as :async. This may cause erratic test results and possibly even deadlocks.

Although, [`Ecto.Adapters.SQL.Sandbox`](https://hexdocs.pm/ecto/Ecto.Adapters.SQL.Sandbox.html#module-database-support) also contains a note about that:

> While both PostgreSQL and MySQL support SQL Sandbox, only PostgreSQL supports concurrent tests while running the SQL Sandbox. Therefore, do not run concurrent tests with MySQL as you may run into deadlocks due to its transaction implementation.

We are using PostgreSQL for that project. So I am going to enable that option at my own peril.

OK. End of note. Let's move on and write some tests.

The `Prater.Auth.User` module contains the `registration_changeset/2` function.


```elixir
def registration_changeset(%User{} = user, attrs) do
  user
  |> changeset(attrs)
  |> validate_confirmation(:password)
  |> cast(attrs, [:password], [])
  |> validate_length(:password, min: 6, max: 128)
  |> encrypt_password()
end
```

which depends on the `Prater.Auth.User.changeset/2` function:

```elixir
def changeset(%User{} = user, attrs) do
  user
  |> cast(attrs, [:email, :username])
  |> validate_required([:email, :username])
  |> validate_length(:username, min: 3, max: 30)
  |> unique_constraint(:email)
  |> unique_constraint(:username)
end
```

It's time to write our first test.

```elixir
alias Prater.Auth.User

describe "User.registration_changeset/2" do
  @invalid_attrs %{}

  test "changeset with invalid attributes" do
    changeset = User.registration_changeset(%User{}, @invalid_attrs)

    refute changeset.valid?
  end
end
```

Here we are checking that the registration changeset should be invalid if we are using an empty map as attributes.
Now let's check the opposite. We want to check when it is also valid. So we are using all required and valid attributes.


```elixir
@valid_attrs %{
  email: "user@example.com",
  username: "user",
  password: "password",
  password_confirmation: "password"
}

test "changeset with valid attributes" do
  changeset = User.registration_changeset(%User{}, @valid_attrs)

  assert changeset.valid?
end
```

Now we know when our changeset would be valid and then it would not be valid. But that does not provide us a lot of information.
What if we want to check the validation error of the changeset.
In that case, we can make a single attribute invalid and check the error message of that field.

```elixir
test "changest with username less than 3 characters" do
  changeset = User.registration_changeset(%User{}, %{@valid_attrs | username: "uu"})

  assert %{username: ["should be at least 3 character(s)"]} = errors_on(changeset)
end
```

The same for the upper border of the `username` field.

```elixir
test "changest with username more than 30 characters" do
  attrs = %{@valid_attrs | username: String.duplicate("u", 31)}
  changeset = User.registration_changeset(%User{}, attrs)

  assert %{username: ["should be at most 30 character(s)"]} = errors_on(changeset)
end
```

And we can cover the rest of the fields in the same way.

Now let's move upper one level and test `Prater.Auth` module. The module contains the following function:

```elixir
def register(params) do
  User.registration_changeset(%User{}, params) |> Repo.insert()
end
```

The function uses the changeset we just have tested and also creates a database record.
By testing that function we are kinda going to test these two pieces together.

Let's create a new file `test/prater/auth/auth_test.exs` with the following content:

```elixir
defmodule Prater.AuthTest do
  use Prater.DataCase, async: true

  alias Prater.Repo
  alias Prater.Auth
  alias Prater.Auth.User

  describe "Auth.register/1" do
    @valid_attrs %{
      email: "user@example.com",
      username: "user",
      password: "password",
      password_confirmation: "password"
    }

    test "changeset with non-unique email" do
      Auth.register(@valid_attrs)

      {:error, changeset} = Auth.register(@valid_attrs)

      assert %{email: ["has already been taken"]} = errors_on(changeset)
    end
  end
end
```

In that test, we are registering a new user. Then we are trying to register another user with the same attributes.
That should not be possible because the user with the same email already exists.
And yes, we can see that the changeset contains a validation error `email: ["has already been taken"]`.

Next, let's check if that function actually creates a record in the database.

```elixir
test "all attributes are saved properly" do
  {:ok, user} = Auth.register(@valid_attrs)
  user = Repo.get!(User, user.id)

  assert is_nil(user.password)
  assert Comeonin.Bcrypt.checkpw(@valid_attrs[:password], user.encrypted_password)
  assert user.email == @valid_attrs[:email]
  assert user.username == @valid_attrs[:username]
end
```

We are registering a user. Then fetch him from the database and check if the record contains all required attributes.
We even check if the password properly encrypted.

As I've already mentioned above, Model Testing is pretty similar to Unit Testing. By using the same technique you can cover most of the Models' functionality.

Let's move on and write some tests for controllers.


# Testing controllers

Controller test we can be attributed to integration layer. Because they are covering several pieces of functionality at once.

At first, let's test how our registration functionality works.

We will start from the `test/prater_web/controllers/registration_controller_test.exs` file.

```elixir
defmodule PraterWeb.RegistrationControllerTest do
  use PraterWeb.ConnCase
end
```

In the same way, as we were using `Prater.DataCase` for testing Models.
The [`PraterWeb.ConnCase`](https://github.com/ck3g/prater/blob/c3f06e739b696856504b8419155e74bb3f6d5592/test/support/conn_case.ex) extends our test suites with the functionality to use connection struct.

Let's start with the simplest test. We have a "Sign Up" page. We would like to request that page and check if it contains the "Sign Up" label.


```elixir
test "GET /registrations/new", %{conn: conn} do
  conn = get conn, "/registrations/new"
  assert html_response(conn, 200) =~ "Sign Up"
end
```

as a second argument of the `test` macro, we are grabbing the connection struct in order to use it.
Then we are sending a GET request to the `/registrations/new` path. Finally, we are checking if a successful response contains the text we want to see. And it does.

Now let's move on to another example. Let's check if our `create` action works.

```elixir
@params %{
  email: "user@example.com",
  username: "username",
  password: "password",
  password_confirmation: "password"
}
test "POST /registrations", %{conn: conn} do
  conn = post conn, "/registrations", [registration: @params]

  assert redirected_to(conn) == "/"
  assert get_flash(conn, :info) == "You have successfully signed up!"
  assert get_last_user().email == "user@example.com"
end

defp get_last_user do
  alias Prater.Auth.User
  import Ecto.Query

  Prater.Repo.one(from u in User, order_by: [desc: u.id], limit: 1)
end
```

At the beginning we define params we will use to create a user.
Then we perform a POST request to the `/registrations` path and passing those params.

As a result, we are expecting the page to be redirected to root path,
and have a flash message properly set up.
In the last step, we are fetching a last user from the database and check if that the user we were creating.
The `get_last_user` function is just to make our test shorter.

We have covered "Happy path" here, you can test the endpoint with invalid attributes in a similar way.

Now let's move on to `RoomController` where we can cover other examples.

Create the `test/prater_web/controllers/room_controller_test.exs` file:

```elixir
defmodule PraterWeb.RoomControllerTest do
  use PraterWeb.ConnCase

  test "GET /", %{conn: conn} do
    conn = get conn, "/"
    assert html_response(conn, 200) =~ "Welcome to Prater"
  end
end
```

The test for `index` action is pretty similar to one we already have.

Let's write a test for `RoomController.show` action.
Unlike `index` action we need to have a room in the database and our user have to be authenticated.
Let's look at the first version of the test and then walk through step by step.

```elixir
test "GET /room/:id", %{conn: conn} do
  user = create_user()
  {:ok, room} = Prater.Conversation.create_room(
    user,
    %{name: "Lobby", description: "The general chat room"}
  )
  conn =
    conn
    |> Plug.Test.init_test_session(current_user_id: user.id)
    |> get("/rooms/#{room.id}")

  assert html_response(conn, 200) =~ "<h2>Lobby</h2>"
  assert html_response(conn, 200) =~ "<div>The general chat room</div>"
end

@default_email "user@example.com"
@default_password "password"

defp create_user(email \\ @default_email) do
  [username, _] = String.split(email, "@")
  params = %{
    email: email,
    username: username,
    password: @default_password,
    password_confirmation: @default_password
  }
  {:ok, user} = Prater.Auth.register(params)

  user
end
```

First, as I've already mentioned we need a user, so we create it. Using `create_user` function as a helper.
Then we are creating a room. The room should belong to a user, and we assign the user to the room.

Next, we need to authenticate the user somehow.

On one hand, we can perform a POST request to sign in our user and then perform another request to access the room's page.
That is a viable solution.
But we probably would need to repeat that approach for other tests, which would make +1 request for every test.
That would slow down every test by the time required to perform that sign in request.
At the end that would pile up and make all the tests slower.

On another hand, we can make a shortcut and pretend our user already authenticated.
If we look at [the implementation of our `SetCurrentUser` plug](https://github.com/ck3g/prater/blob/c3f06e739b696856504b8419155e74bb3f6d5592/lib/prater_web/plugs/set_current_user.ex#L11), we can see that in order to be authenticated all we need is to have a `current_user_id` in the session struct:

```elixir
user_id = Plug.Conn.get_session(conn, :current_user_id)
```

Let's update the session struct then.

The naive solution could be just to use `put_session/2` function. Then struggle with the

```
(ArgumentError) session not fetched, call fetch_session/2
```

error.
The thing is, that we have no session struct that that point because we didn't perform any requests yet.
So one of the solutions would be to use the [Plug.Test.init_test_session/2](https://hexdocs.pm/plug/Plug.Test.html#init_test_session/2) function with the attributes we need.

And finally to perform a GET request to `/room/:id` URL.

As a result, we are expecting to see the room's title and description on the page.

At this moment our test looks quite cluttered. Let's see how we can refactor it to look nicer.

At first, we can extract authentication logic into `setup` block.

```elixir
setup do
  user = create_user()
  conn = Plug.Test.init_test_session(build_conn(), current_user_id: user.id)

  {:ok, conn: conn, user: user}
end
```

Here we are creating the user, and init session with his ID. We are using `build_conn/0` function from the `Prater.ConnCase` module.
In the end, we return the extended connection and the user out of `setup`.

We still need to know the user in order to create the room, so we are fetching it from the `test` arguments.

```elixir
test "GET /room/:id", %{conn: conn, user: user} do
  {:ok, room} = Prater.Conversation.create_room(
    user,
    %{name: "Lobby", description: "The general chat room"}
  )

  conn = get(conn, "/rooms/#{room.id}")

  assert html_response(conn, 200) =~ "<h2>Lobby</h2>"
  assert html_response(conn, 200) =~ "<div>The general chat room</div>"
end
```

Now the test looks cleaner.

Now, by using `setup` in the way we use it, we authenticate the user for every request even if we don't need it as for `index` action.

Can we improve that? Yes, sure. We can use tags.

First, let's add a tag in front of our test and set the value of user's email we want to use:


```elixir
@tag sign_in_as: "user@example.com"
test "GET /room/:id", %{conn: conn, user: user} do
```

Then in the `setup` block, we check for that tag, and if it exists, we would create and authenticate the user and do nothing otherwise.

```elixir
setup %{conn: conn} = config do
  if email = config[:sign_in_as] do
    user = create_user(email)
    conn = Plug.Test.init_test_session(build_conn(), current_user_id: user.id)

    {:ok, conn: conn, user: user}
  else
    :ok
  end
end
```

One more thing we can notice is that we would probably need to create a user in some other test suites.
It would be nice to avoid duplication.

We can extract `create_user/1` function into helper module.

Create the `test/support/auth_helpers.ex` file and move the function there.


```elixir
defmodule Prater.AuthHelpers do
  @default_email "user@example.com"
  @default_password "password"

  def create_user(email \\ @default_email) do
    [username, _] = String.split(email, "@")
    params = %{
      email: email,
      username: username,
      password: @default_password,
      password_confirmation: @default_password
    }
    {:ok, user} = Prater.Auth.register(params)

    user
  end
end
```

Now to make it work we need to import that module in the `ConnCase`. Open `test/support/conn_case.ex` and update the `quote` block:

```elixir
using do
  quote do
    # ...
    import Prater.AuthHelpers
    # ...
  end
end
```

Now check your tests again. They should be green.

## Wrapping up

Today we have covered how to write tests for two main pillars of MVC: Models and Controllers.
Using those two types of tests we can cover a lot of functionality and be calm during refactoring sessions.
Because we don't need to check every piece of functionality manually anymore.

That's all for now. But there is still some type of tests we are going to cover.

All the changes we did today, you can find in [the GitHub repository](https://github.com/ck3g/prater/pull/12).

See you next time.

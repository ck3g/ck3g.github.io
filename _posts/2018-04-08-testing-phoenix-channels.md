---
layout: post
title: "Testing Phoenix channels"
desc: "Learn how to cover Phoenix Channels with tests."
keywords: "Automated tests, unit tests, channel tests, Elixir, ExUnit"
tags: [Elixir, Phoenix, Testing]
excerpt_separator: <!--more-->
---

In the previous articles we have covered [the introduction to testing](http://whatdidilearn.info/2018/03/25/introduction-to-testing.html) and how to [test Phoenix Models and Controllers](http://whatdidilearn.info/2018/04/01/testing-phoenix-models-and-controllers.html).

This time are going to cover the testing approach for Phoenix channels.

<!--more-->

<hr />
This article is the part of the testing in Elixir series:

* [Part 1: Introduction to testing](http://whatdidilearn.info/2018/03/25/introduction-to-testing.html)
* [Part 2: Testing Models and Controllers](http://whatdidilearn.info/2018/04/01/testing-phoenix-models-and-controllers.html)
* Part 3: Testing Channels
* [Part 4: User Interface testing](http://whatdidilearn.info/2018/04/15/phoenix-ui-tests-using-hound.html)
<hr />

As usual, we will use ["Prater"](https://github.com/ck3g/prater) project as our playground. We already have a functionality to join a room and send messages to other users. So let's cover that functionality with tests.

## Testing socket authentication

Every time a user visits a page [we are authenticating the socket](http://whatdidilearn.info/2018/03/04/using-channels-in-phoenix.html#socket-authentication) in order to distinguish one user from another. We signing the token with the user's ID inside. Once user connected we are verifying the token and assign the user's ID to the socket.

Here is the implementation of the `connect` function.

```elixir
defmodule PraterWeb.UserSocket do
  # ...

  @max_age 24 * 60 * 60
  def connect(%{"token" => token}, socket) do
    case Phoenix.Token.verify(socket, "user token", token, max_age: @max_age) do
      {:ok, user_id} ->
        {:ok, assign(socket, :current_user_id, user_id)}
      {:error, _reason} ->
        :error
    end
  end

  # ...
end
```

Let's start with the tests of that functionality.

At first, we need to create a test file `test/prater_web/channels/user_socker_test.exs` with the following content.

```elixir
defmodule PraterWeb.UserSocketTest do
  use PraterWeb.ChannelCase, async: true
  alias PraterWeb.UserSocket

  test "authenticate with valid token" do
    token = Phoenix.Token.sign(@endpoint, "user token", 503)

    assert {:ok, socket} = connect(UserSocket, %{"token" => token})
    assert socket.assigns.current_user_id == "503"
  end
end
```

In a similar way to Models and Controllers, Phoenix provides us a helper module to test channels. It is called `ChannelCase` and it's located in the [`test/support/channel_case.ex`](https://github.com/ck3g/prater/blob/76a0a4b/test/support/channel_case.ex) file.

The module provides us all required functionality to test channels.

First, we sign the token with the user's ID. For now, it does not matter if that user exists or not. Then, we are connecting to the socket and check if the connection was successful. As the last step, we check the assigned value of `current_user_id` to be sure it is the correct one.

And on a contrary, we can test unsuccessful connects.

```elixir
test "authenticate with invalid token" do
  assert :error = connect(UserSocket, %{"token" => "invalid-token"})
  assert :error = connect(UserSocket, %{})
end
```

The first assert checks the connection with an invalid token. It is invalid because we didn't sign it. The second asset checks the connection with the missing token parameter.

That is pretty much it for testing socket authentication.

## Testing channel subscription

When a user visits the room's page the second thing we do after the socket authentication is subscribing him to a room's channel.

That is the implementation of the `join/3` function.
First, we are sending "after_join" event to track the presence (later about that).
Then we are responding back with the list of messages.
We also assign the room's ID to the socket.

```elixir
defmodule PraterWeb.RoomChannel do
  # ...

  def join("room:" <> room_id, _params, socket) do
    send(self(), :after_join)

    {
      :ok,
      %{messages: Conversation.list_messages(room_id)},
      assign(socket, :room_id, room_id)
    }
  end

  # ...
end
```

Let's start with the test for checking the correct assignment of a room's ID to the socket.
We need to create a test file `test/prater_web/channels/room_channel_test.exs` and add a first test.

```elixir
defmodule PraterWeb.RoomChannelTest do
  use PraterWeb.ChannelCase

  alias Prater.Conversation
  alias PraterWeb.UserSocket

  import Prater.AuthHelpers

  test "assigns room_id to the socket after join" do
    user = create_user("user@example.com")
    {:ok, room} = Conversation.create_room(user, %{name: "Lobby"})

    token = Phoenix.Token.sign(@endpoint, "user token", user.id)
    {:ok, socket} = connect(UserSocket, %{"token" => token})

    {:ok, _, socket} = subscribe_and_join(socket, "room:#{room.id}", %{})

    assert socket.assigns.room_id == "#{room.id}"
  end
end
```

First, let's look to our assertment and then we will figure out why do we need so much code for that.

We are trying to test what the socket has a correct room ID assigned to it.

That means we need to have a room in the database.
In order to create a room, we also need to have a user. The first two lines are responsible for that.
Then we need to authenticate a user. We do that in the same way as we did in the socket authentication section.
And finally, we need to subscribe to the channel. That is when the `join/3` function would be triggered.

In a similar way, we can test if `join/3` function responds with the list of messages.

```elixir
test "returns the list of messages after join" do
  user = create_user("user@example.com")
  {:ok, room} = Conversation.create_room(user, %{name: "Lobby"})

  token = Phoenix.Token.sign(@endpoint, "user token", user.id)
  {:ok, socket} = connect(UserSocket, %{"token" => token})

  Conversation.create_message(user, room, %{"content" => "Hello from Test"})

  {:ok, messages, _} = subscribe_and_join(socket, "room:#{room.id}", %{})

  messages_template = %{
    messages: [
      %{content: "Hello from Test", user: %{username: user.username}}
    ]
  }

  assert messages == messages_template
end
```

The as in the previous example we are creating a user, a room and authenticate the socket.
But this time, before we join we also need to create a message because we want to get something back.
After that, we are subscribing to a room's channels and expecting to get a list of messages back. And we do.

Both tests are working, but we have a duplication logic. That means we need to stop and refactor.

We can extract the repetitive behavior into the `setup` block.

```elixir
setup do
  user = create_user("user@example.com")
  {:ok, room} = Conversation.create_room(user, %{name: "Lobby"})

  token = Phoenix.Token.sign(@endpoint, "user token", user.id)
  {:ok, socket} = connect(UserSocket, %{"token" => token})

  {:ok, socket: socket, user: user, room: room}
end
```

Now we need to update the attributes of the `test` macros and fetch `socket`, `user`, and `room`.

```elixir
test "assigns room_id to the socket after join",
  %{socket: socket, room: room} do
# ...
end
```

and

```elixir
test "returns list of messages after join",
  %{socket: socket, user: user, room: room} do
  # ...
end
```

And now that part of the `join/3` function is well tested.


## Testing Presence

That is not everything our `join/3` function does. We are also using the ["Presence"](http://whatdidilearn.info/2018/03/11/using-phoenix-presence.html) functionality.

That is how the function looks like. Once a user subscribed to a channel, we are sending "after_join" event.

```elixir
def join("room:" <> room_id, _params, socket) do
  send(self(), :after_join)

  {
    :ok,
    %{messages: Conversation.list_messages(room_id)},
    assign(socket, :room_id, room_id)
  }
end
```

Which in the end triggers the following callback

```elixir
def handle_info(:after_join, socket) do
  push socket, "presence_state", Presence.list(socket)

  user = find_user(socket)

  {:ok, _} = Presence.track(socket, "user:#{user.id}", %{
    typing: false,
    user_id: user.id,
    username: user.username
  })

  {:noreply, socket}
end
```

where we send "presence_state" event.

Let's test it now.

```elixir
test "broadcasting presence", %{socket: socket, user: user, room: room} do
  {:ok, _, socket} = subscribe_and_join(socket, "room:#{room.id}", %{})

  user_data = %{
    typing: false, user_id: user.id, username: user.username
  }
  assert_push "presence_state", user_data
  assert_broadcast "presence_diff", user_data
end
```

Now we don't need to create a room and authenticate the socket. We already do that in the `setup` block.
All we need to do is to subscribe to a channel and check of "presence_state" has been pushed to a channel and "presence_diff" has been broadcasted.
Both events should contain a user's data we are expecting.

And we are done with the presence.


## Testing broadcasting

When a user writes a message in the chat. We are pushing a new event to a channel.
Then we are trying to create a message in the database. If we succeeded we are broadcasting a "new_message" event to all subscribers.

That is the implementation of that function

```elixir
def handle_in("message:add", %{"message" => content}, socket) do
  room = Conversation.get_room!(socket.assigns[:room_id])
  user = find_user(socket)

  case Conversation.create_message(user, room, %{content: content}) do
    {:ok, message} ->
      message = Repo.preload(message, :user)
      message_template = %{content: message.content, user: %{username: message.user.username}}
      broadcast!(socket, "room:#{message.room_id}:new_message", message_template)
      {:reply, :ok, socket}

    {:error, _reason} ->
      {:reply, :error, socket}
  end
end
```

That is how the test would look like.

```elixir
test "adding a new message", %{socket: socket, user: user, room: room} do
  {:ok, _, socket} = subscribe_and_join(socket, "room:#{room.id}", %{})
  ref = push(socket, "message:add", %{"message" => "I'm a new msg"})

  assert_reply ref, :ok, %{}

  msg = get_last_message()
  msg_template = %{content: msg.content, user: %{username: user.username}}
  broadcast_event = "room:#{msg.room_id}:new_message"

  assert_broadcast "presence_diff", %{}
  assert_broadcast broadcast_event, msg_template
  refute is_nil(msg)
end

defp get_last_message do
  alias Prater.Conversation.Message
  import Ecto.Query

  Prater.Repo.one(from m in Message, order_by: [desc: m.id], limit: 1)
end
```

First, we are subscribing to a channel. After that we are emulating an "message:add" event.
On the first milestone, we are using `assert_reply` to check the successful reply.
Then, we fetch the last message in the database and prepare the expected template.
The function creates a new message in the database, so in that test, we want to be sure we are broadcasting the message we just created.
Then we check if the event has been broadcasted and the message exists in the database.

In that example I'm also using `assert_broadcast "presence_diff", %{}` before the actual broadcast assert.
I've figured it out in an experimental way. I've noticed that without `assert_broadcast "presence_diff", %{}` the `assert_broadcast broadcast_event, msg_template` is always succeeding even if I comment out the actual broadcasting. That behavior still puzzles me. If you know more about that would be happy to learn about that form the comments below.

## Wrapping Up

So, we are one more step further in the Elixir learning. This time we have learned how to test the sockets layer of Phoenix applications.
Now we know how to test the socket authentication, how to test the response of the `join` function, the presence functionality and check that our stuff is properly broadcasted.

Yet still, that is not everything about testing we are still missing the proper integration testing.

See you next time.

All the changes you can find in [the GitHub page](https://github.com/ck3g/prater/pull/13) of the project.

---
layout: post
title: "Introduction to Testing"
desc: "Today we are going to cover the introduction to testing. We are going to cover that are automated tests in general, what kind of test we can have. Next, we will dig deeper into the unit tests using Elixir."
keywords: "Automated tests, unit tests, UI tests, integration tests, Elixir, ExUnit, test pyramid, doctests"
tags: [Elixir, Testing]
excerpt_separator: <!--more-->
---

Testing is an important part of software development.
One might argue that projects can survive with manual testing only and many of them do.
But that raises a couple of questions:

* How hard is it to refactor some parts in those projects?
* Ain't one have a fear to refactor the code which uncovered by tests?
* How much time does it take to check the changes one just did?
* Didn't those changes break some other part of the project?
* How to prove that the code is working as it supposed to work?

Yes, it requires the time to write tests. But does it take less time to test the stuff manually over and over again?

<!--more-->

And yes, some parts are harder to test than the others.
Also, there is no need to aim for 100% test coverage. Testing software is like a writing software. It's all about trade-offs.

At some point in time, one would understand the importance of automatic testing.

Automatic testing of software is a huge topic. Which probably requires a book to cover. We would need several articles to cover most important parts.

<hr />
This article is the part of the testing in Elixir series:

* Part 1: Introduction to testing
* [Part 2: Testing Models and Controllers](http://whatdidilearn.info/2018/04/01/testing-phoenix-models-and-controllers.html)
* [Part 3: Testing Channels](http://whatdidilearn.info/2018/04/08/testing-phoenix-channels.html)
* [Part 4: User Interface testing](http://whatdidilearn.info/2018/04/15/phoenix-ui-tests-using-hound.html)
<hr />

#### What are automated tests?

Automated tests are parts of the code which checks the other parts of the code. The test "passes" (or "green") when the target code works as it is described in the test. The test "fails" (or "red") when the target code works no in the way it is expected to work.

There are several types of test we can have. I would single out 3 most common types.

#### Unit tests

Unit tests are usually the tests with the smallest possible scope. They are testing the smallest and (most of the time) isolated part of the code.
If you want to create a function which adds two numbers then the unit test would be the test which checks if a sum of those numbers is correct.

#### Integration tests

Integration tests are a group of tests which are testing how some parts of the functionality are working together.
By using a single integration test, we can check the functionality of several layers of an application.

#### User Interface (UI) tests

UI tests are the test which simulates user's behavior. Usually, they fire up the browser and start clicking around, filling in fields and submits forms.
Those tests are very slow comparing to unit and integration tests, but they can cover a lot of parts of an application.

You can think about those type of tests as a pyramid.


```
Y
^
|                /\
|               /  \
|              /    \
|             /  UI  \
|            /        \
|           /----------\
|          /            \
|         /  Integration \
|        /                \
|       /------------------\
|      /                    \
|     /      Unit Tests      \
|    /________________________\
|
+------------------------------------> X
```

Where `X` is a number of tests and `Y` can be treated as a speed of execution, cost of maintenance, coverage, etc.

If you want to dig more into the topic, there are a lot of articles around. I think you can start with [Martin Fowler's TestPyramid](https://martinfowler.com/bliki/TestPyramid.html).

The simplest way to get started is to start with Unit Tests. It's easier to write and simpler to understand because they are small.

# Unit Testing

Elixir comes with the built-in unit testing framework - [ExUnit](https://hexdocs.pm/ex_unit/ExUnit.html). So we can start writing tests without additional hassle.

Imagine we are working on a simple project we have created using `mix new testing_example`.

By default `mix` tool creates us `test` directory with two files: `test/test_helper.exs` and `test/testing_example_test.exs`

The `test_helper.exs` file contains single line:

```elixir
ExUnit.start()
```

Mix requires us to create that file, which will be loaded before executing tests.
Mix then run all files in the `test` directory which has `_test` suffix in the filename.

The minimum content of the test file looks like:

```elixir
defmodule TestingExampleTest do
  use ExUnit.Case
end
```

Where `TestingExampleTest` is the name of the module we are going to test followed by word `Test`.
Well, actually that is a convention. You are free to use a different name for the module. That will work.
By following that convention it would be easier to navigate through a project and would help you understand what exactly are we testing here.

And by using `ExUnit.Case` module we are injecting all required functionality into the module.

Let's start with the simplest test.


```elixir
test "true is true" do
  assert true
end
```

Several things are happening here.

First, we define our test by using `test` macro, which accepts description and a block.
The block is a test body.

Then we use one of the assertion functions to check the result. The test passes if the value of `assert` is truthy and fails otherwise.

To invoke our tests we can run `mix test` from the root directory of the project.

```
→ mix test
.

Finished in 0.02 seconds
1 test, 0 failures
```

If we want to check the value to be falsey, we can use `refute` instead of `assert`.

```elixir
test "false is not true" do
  refute false
end
```

#### Testing async

A small note about `use ExUnit.Case` it can accept `async: true` param.
That makes our test cases to run in parallel.
All the tests in the same test case would still be invoked one by one.

Let's look at the example.

```elixir
defmodule TestingExampleTest do
  use ExUnit.Case

  test "true is true" do
    :timer.sleep(1500)
    assert true
  end
end

defmodule ExampleTest do
  use ExUnit.Case

  test "false is not true" do
    :timer.sleep(1000)
    refute false
  end
end
```

If we run `mix test`:

```
→ mix test
..

Finished in 2.5 seconds
2 tests, 0 failures
```

We can see that the execution time is 2.5 seconds.
The first test takes 1.5 seconds to execute and the second one takes only 1 second.

Now, if we update both of our `use ExUnit.Case` calls to use `async`

```elixir
use ExUnit.Case, async: true
```

We would see that our test now requires only 1.5 seconds to run.

```
→ mix test
..

Finished in 1.5 seconds
2 tests, 0 failures
```

Pretty cool huh?

Now let's get back to our tests.

We can easily notice that those tests do not test anything, they do not touch the code we have.
Let's try it on real examples.

Suppose we have the following function, which accepts a list of numbers and returns a sum as a result.

```elixir
defmodule TestingExample do
  def add(numbers), do: Enum.sum(numbers)
end
```

The test (one of them) for that function may be written like:

```elixir
test "returns a sum of the numbers" do
  assert TestingExample.add([1, 2]) == 4
end
```

Inside the assertion, we are calling the function with a `[1, 2]` list and expecting to have `4`.
After we run our tests we can see the following message:

```
→ mix test


  1) test returns a sum of the numbers (TestingExampleTest)
     test/testing_example_test.exs:4
     Assertion with == failed
     code:  assert TestingExample.add([1, 2]) == 4
     left:  3
     right: 4
     stacktrace:
       test/testing_example_test.exs:5: (test)



Finished in 0.02 seconds
1 test, 1 failure
```

Our test fails. We can see what exactly is wrong. Our "left" part, which is a call to a function, equals "3".
Our right part, the value what we are expecting is "4".
Now we can fix the test and run it again.

If we want to test the same function with different arguments, then we can write more tests.

```elixir
test "returns 0 for an empty list" do
  assert TestingExample.add([]) == 0
end
```

The more functions our module has the more tests we can write.
At some point in time when we will have a decent amount of tests, we would notice that the readability of tests decreasing.
All the tests are mixed for the different functions.

Can we improve that somehow? Yes, we can group tests into the same context. We do that by using `describe` macro. Let's fix that for our tests:

```elixir
describe "TestingExample.add/1" do
  test "returns 0 for an empty list" do
    assert TestingExample.add([]) == 0
  end

  test "returns a sum of the numbers" do
    assert TestingExample.add([1, 2]) == 3
  end
end
```

Now it's much easier to understand that those tests are related to the `add/1` function.

Sometimes you may need to prepare something before running your tests.
For example to create some records in the database which your test will be using.
In order to avoid duplication, you can use `setup` or `setup_all` macros for that.

```
setup do
  IO.puts "Runs before every test"
end

setup_all do
  IO.puts "Runs before that test suite"
end
```

## Doctests

Doctests is yet another way to test our code. One can test it by [describing documentation](http://whatdidilearn.info/2017/10/23/writing-documentation-in-elixir.html) examples for a function.

First, we need to add `doctest TestingExample` in our test module. That will start executing documentation tests if there are any.

Now we can describe documentation examples ([Read more about "Writing documentation in Elixir"](http://whatdidilearn.info/2017/10/23/writing-documentation-in-elixir.html)):

```elixir
defmodule TestingExample do
  @doc """
    Returns sum of numbers in the list

  ## Examples

      iex> TestingExample.add([])
      0

      iex> TestingExample.add([1])
      1

      iex> TestingExample.add([1, 2])
      3

  """
  def add(numbers), do: Enum.sum(numbers)
end
```

Now, by running `mix test` we can see 3 doctests are executed:

```
→ mix test
Compiling 1 file (.ex)
.....

Finished in 0.05 seconds
3 doctests, 2 tests, 0 failures
```

Try to change one of the return values in the example to incorrect one. You can see it's working:

```
→ mix test
Compiling 1 file (.ex)


  1) doctest TestingExample.add/1 (1) (TestingExampleTest)
     test/testing_example_test.exs:3
     Doctest failed
     code: TestingExample.add([]) === 10
     left: 0
     stacktrace:
       lib/testing_example.ex:7: TestingExample (module)

....

Finished in 0.05 seconds
3 doctests, 2 tests, 1 failure
```


## Wrapping up

Today we have talked about testing in general and covered unit testing with built-in `ExUnit`.
It may look like `ExUnit` does not have a lot of features. Even though that is enough to write powerful tests.

We've just scratched the surface of the Automated Testing topic, but we have covered the important basics.
We are going to build the knowledge on top of that.

Next time we will move ourselves to Phoenix and will start learning how to write another type of tests in that domain.

See you next time.

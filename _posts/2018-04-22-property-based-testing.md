---
layout: post
title: "Property-based testing"
desc: "Learn about basic concepts of Property-based testing and how to use it in Elixir."
keywords: "Property-based testing, QuickCheck, StreamData, Random Testing, Automated Testing"
tags: [Elixir, Testing]
excerpt_separator: <!--more-->
---

Last week I was visiting the [ElixirConf EU 2018](https://www.elixirconf.eu/).
Speakers were covering a bunch of interesting topics there.
One of the most interesting ideas I've got out of those talks was a "Property-based testing" topic.
That topic was covered by **José Valim** (the creator or Elixir) in the Keynote speech and [then in more details](https://youtu.be/p84DMv8TQuo) by (a core team member) **Andrea Leopardi**.

Let's try to figure out what is a Property-based testing.

<!--more-->

<hr />
This article is the part of the testing in Elixir series:

* [Part 1: Introduction to testing]({{ site.url }}/2018/03/25/introduction-to-testing.html)
* [Part 2: Testing Models and Controllers]({{ site.url }}/2018/04/01/testing-phoenix-models-and-controllers.html)
* [Part 3: Testing Channels]({{ site.url }}/2018/04/08/testing-phoenix-channels.html)
* [Part 4: User Interface testing]({{ site.url }}/2018/04/15/phoenix-ui-tests-using-hound.html)
* Part 5: Property-based testing
<hr />



Property-based testing originates from the [QuickCheck paper](https://www.cs.tufts.edu/~nr/cs257/archive/john-hughes/quick.pdf).

<p align="center">
  <img src="{{ site.url }}/img/posts/pbt/quickcheck-paper.png" />
</p>




The idea behind that instead of providing several expected values for functions, you define properties that should hold true for all input values.
Then you describe your property-based tests in a way that the tool you are using generates and feeds random input data to check the correctness of the functions.

All that functionality can be split into two parts: data generation and property testing.

For Elixir we can use the [StreamData](https://github.com/whatyouhide/stream_data) library. Which provides everything we need.
The StreamData will be the part of Elixir 1.7 starting from July 2018.

Let's look at the following example.

Imagine we have a cash register where a user can pay for some purchases.
The function accepts the total price and the amount of money the user gives to the counter.


```elixir
defmodule CashRegister do
  def pay(price, paid) when paid >= price do
    {:ok, paid - price}
  end

  def pay(_price, _paid) do
    :paid_amount_too_small
  end
end
```

The properties of that function could be:

* The amount of change the user should get back equals to the amount of money paid minus total price.
* In case the provided amount is less than total price, the function responds with an error.

That is how we can describe them using the StreamData library.

```elixir
use ExUnitProperties

@max 1_000_000
property "pay with a paid >= price" do
  check all price <- integer(1..@max),
            paid <- integer(price..@max) do
    assert CashRegister.pay(price, paid) == {:ok, paid - price}
  end
end

property "pay with a paid < price" do
  check all price <- integer(100..@max),
            paid <- integer(100..(price - 1)) do
    assert CashRegister.pay(price, paid) == :paid_amount_too_small
  end
end
```

First, we are using [`property`](https://hexdocs.pm/stream_data/ExUnitProperties.html#property/3) macros instead of `test`. It is very similar to `test` and it just means you are testing a property.

Then we are using [`check`](https://hexdocs.pm/stream_data/ExUnitProperties.html#check/1) macros which accepts generators and run the tests described in the body.

Generators in our case are the random [`integer`](https://hexdocs.pm/stream_data/StreamData.html#integer/1) numbers (the amount is in cents) which we use to fill in `price` and `paid` variables.

In the first property, we ensure that the `paid` is always greater than or equal to `price`. And vice versa in the second example.

Once we run out test suite, the property would run those checks with the random data 100 times (by default). It helps to ensure that we have been testing our code with the decent variety of input data.

### Shrinking

Once the property test is failing, the library tries to shrink the input values to the minimum possible.

For example, you are testing the sort function. And the test fails on some value.
If that value is `[503, 505, 510, 521, 512, 530]` you would need to stop for a while and check what is wrong here.
It would be much easier to spot the error on the following list `[1, 0]`. Here you can easily see that is not sorted.

That is exactly how the shrinking functionality tries to help us.

Let's write the flaky property test to see the output we can get.

```elixir
use ExUnitProperties

property "random comparison" do
  check all a <- list_of(integer(), min_length: 3),
            b <- list_of(integer(), min_length: 3) do
    assert Enum.sort(a) != Enum.sort(b)
  end
end
```

Here we are generating two lists and then check if those lists are not equal.
As soon as we are getting random lists there is a chance that property would fail.

Then we need to run tests several times to get lucky and see the failure

```
→ mix test

  1) property random comparison (PropBaseTest)
     test/prop_base_test.exs:6
     Failed with generated values (after 1 successful run):

         * Clause:    a <- list_of(integer(), min_length: 3)
           Generated: [0, 0, 0]

         * Clause:    b <- list_of(integer(), min_length: 3)
           Generated: [0, 0, 0]

     Assertion with != failed, both sides are exactly equal
     code: assert Enum.sort(a) != Enum.sort(b)
     left: [0, 0, 0]
     stacktrace:
       test/prop_base_test.exs:9: anonymous fn/3 in PropBaseTest."property random comparison"/1
       (stream_data) lib/stream_data.ex:2021: StreamData.check_all/7
       test/prop_base_test.exs:7: (test)


Finished in 0.04 seconds
1 property, 1 failure
```

From that we can see, that our `a` list is `[0, 0, 0]` and the list `b` has the same value. Those are the simplest possible values. That is very easy to see that's the problem here.


## How property-based testing helps

Those are a couple of points (from **Andrea Leopardi** talk) of how Property-based testing helps us.

It helps to:

* Reduce the number of unit tests
* Find obscure bugs
* Reduce to minimal failing input
* Find specification errors
* Cover vast input space


## Conclusion

Property-based testing is not a replacement for Unit Tests. It is an extension which can be handy in some cases.
More than that, you would need to combine it with unit tests to achieve more robust coverage.
For example, if you want to check that two reversed lists are not equal, the property-based testing may not be that useful here.
You may want to add regular unit tests to check that.


**Edsger W. Dijkstra** once said.

> Program testing can be used to show the presence of bugs, but never to show their absence!

Can we improve the way how we test our software? Probably.
Although the Property-based testing is not a silver bullet, it can help us to achieve better results.

As for me, I would like to extend my toolbox with that approach and see how it would help me in my work.

If you are interested to try the property-based testing in other languages, there is a plenty of ported libraries. Most likely you can find it in your language.

If you are interested in the topic I would encourage you to read the canonical ["QuickCheck" paper](https://www.cs.tufts.edu/~nr/cs257/archive/john-hughes/quick.pdf) and [PropEr Testing web site](http://propertesting.com/). There is much more information about the subject.

And of course to check out [Property-based Testing is a Mindset](https://youtu.be/p84DMv8TQuo) by **Andrea Leopardi** at **ElixirConf EU 2018**

+++
title = "Setter Method Return Values In Ruby"
date = 2021-11-17T19:04:22-05:00
tags = ["ruby"]
summary = "Setting the table for a polite receipt of information"
description = "Ruby setter methods may not return the result you expect them to. Find out what they return, and why, in this article."
+++

## Can I Take Your Order?

We're opening a brand new diner, and each customer can have their current order
set, which will add to their list of orders.

```ruby
class Customer
  attr_reader :orders

  def initialize
    @orders = []
  end

  def current_order
    @orders.last
  end

  def current_order=(new_order)
    @orders << new_order

    "thank you for being our valued customer"
  end
end
```

Because customer service is our top priority, we provide a message whenever
they place an order reminding them how lucky we feel to be serving them
their meal.

## Negative Reviews

After our soft opening, we're excited to read everyone's favorable comments
about their food. However, we're getting slammed with bad
reviews about our curt interactions with customers when they place an order. We
went out of our way to modify the setter's return value to give a special
message; what's wrong?

## Mystery Shopper

In order to get to the bottom of this, we put on our fake mustache and glasses
to go undercover at our own restaurant. What we find __shocks us__.

```ruby
irb(main):001:0> definitely_not_the_owner = Customer.new
irb(main):002:0> definitely_not_the_owner.current_order = "pancakes"
=> "pancakes"
```

That's not the welcoming experience we explicitly developed! We shouldn't be
barking back their order in a matter of fact way; we should be thanking them.
Something surprising is happening here; now we need to find what it is.

## Training Manual

Our method is explicit about what the return value should be.

```ruby
def current_order=(new_order)
  @orders << new_order

  "thank you for being our valued customer"
end
```

We expect the last thing evaluated in the method to be what's returned to the
caller. And that's the case...most of the time. After consulting Ruby's
[documentation](https://ruby-doc.org/core-3.0.1/doc/syntax/methods_rdoc.html#label-Return+Values), we find an exception:

> Note that for assignment methods the return value will be ignored when using the assignment syntax. Instead, the argument will be returned

This is our exact case! Our setter method to set the current order is returning
the argument passed to it, and not the result of the last line. It's being
completely ignored in favor of the argument.

## Demanding Exceptional Service

The documentation does provide a way in which customers can see our nice message
though:

> The actual return value will be returned when invoking the method directly

So, while we cannot enforce it, if callers know the right way to ask, we'll
thank them.

```ruby
irb(main):001:0> customer = Customer.new
irb(main):002:0> customer.current_order = "pancakes"
=> "pancakes"
irb(main):003:0> customer.public_send(:current_order=, "french toast")
=> "thank you for being our valued customer"
```

## Setting The Table For Next Time

Ruby setter, or assignment, methods have a special understanding and
expectation for what they'll return. This helps enforce a consistent API, but
can lead to surprising results when writing your own setter method.

Setter methods will return the argument passed to it, regardless of what the
last evaluated statement in the method is. Callers can receive the value of the
last statement if they call the method in a particular way; however, it would be
very unconventional to expect someone to do that.

In the case of our restaurant, that meant a pivoting in our branding. We started
positioning ourselves as an outfit focused on intense efficiency and limited
customer interaction - with a secret message any customer in the know can pass
us to force a little more welcoming of a response with any new order.

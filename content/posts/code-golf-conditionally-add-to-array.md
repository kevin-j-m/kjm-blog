+++
title = "Code Golf: Conditionally Add To An Array"
date = 2021-09-28T19:00:07-04:00
tags = ["ruby"]
summary = "Can you come up with more than four (fore) ways to do this?"
description = "Ruby's expressiveness and vast API provides a myriad of ways to solve the same problem. This post proposes different ways to add elements to an array only if a certain condition applies."
canonicalUrl = "https://www.thegnar.com/blog/code-golf-conditionally-add-to-array"
+++

## The Grass Is Always Greener

We're building a system to track a golfer's statuses during a tournament. This is
a competitive tournament with people who are much better than I will ever be, so
if a golfer is currently scoring under [par](<https://en.wikipedia.org/wiki/Par_(score)>), they're in contention to win. On this 18 hole course, if they've played the first nine holes, they've made the [turn](https://www.golfcompendium.com/2020/07/the-turn-definition-golf-course.html).

Let's explore a number of ways we can build up an array that keeps track of
which, if any, of these statuses a particular golfer qualifies for. Not content
to settle for one that works, we'll dig into a variety of options.

## Teeing Up An Option

We can start with an empty array, and explicitly add in any of the statuses that
the golfer meets the conditions for.

```ruby
def current_statuses
  statuses = []

  if under_par?
    statuses << "in_contention"
  end

  if back_nine?
    statuses << "past_the_turn"
  end

  statuses
end
```

There's nothing tremendously exciting here, and that's not a bad thing! It's
reasonably clear what this is doing.

## Tapping It In

We can make the prior suggestion a little more terse by using [tap](https://docs.ruby-lang.org/en/master/Kernel.html#method-i-tap).

```ruby
def current_statuses
  [].tap do |statuses|
    if under_par?
      statuses << "in_contention"
    end

    if back_nine?
      statuses << "past_the_turn"
    end
  end
end
```

This eliminates the need for the `statuses` temporary array from the prior
section.

## Taking a Compact Swing

We can also build our array to have an entry for each of the conditionals we
have, and removing the ones that aren't relevant.

```ruby
def current_statuses
  [
    under_par? ? "in_contention" : nil,
    back_nine? ? "past_the_turn" : nil,
  ].compact
end
```

By using [compact](https://ruby-doc.org/core-3.0.1/Array.html#method-i-compact),
we'll remove any `nil` values - and we'll take advantage of that functionality
by explicitly adding `nil` if the golfer doesn't meet that condition.

## Working On Your Backswing Takeaway

Speaking of taking things away, we can also do the opposite of the first
approach. We'll start by having each status, and then _removing_ the ones that
do **not** meet the necessary condition.

```ruby
def current_statuses
  statuses = ["in_contention", "past_the_turn"]

  unless under_par?
    statuses -= ["in_contention"]
  end

  unless back_nine?
    statuses -= ["past_the_turn"]
  end

  statuses
end
```

This may be more helpful in situations where there are a lot more statuses, and
only a few of them may need to be removed for certain reasons. It may also help
when the full list of statuses persists on its own elsewhere, but then you also
need a subset of them in a particular case.

## Selecting The Right Club

Lastly, we'll put together a hash keyed on the statuses where the value is the result of the conditions. We can then flex some familiarity with Ruby's [Enumerable module](https://ruby-doc.org/core-3.0.1/Enumerable.html) to pick out the applicable sections of the hash, returning only the statuses.

```ruby
def current_statuses
  {
    "in_contention" => under_par?,
    "past_the_turn" => back_nine?,
  }.select { |_status, condition| condition }.keys
end
```

Similar to the prior suggestion, this may be helpful when you want to have the
full list of statuses and their associated conditions all compiled in one place,
but then want to peel off which are relevant for a particular golfer at this
time.

## Asking Help From The Caddie

Here are some of the ways that we might solve this problem. What other ways
could we build this functionality? [Let me know](https://twitter.com/kevin_j_m)
what other approaches you would take.

> This post originally published on [The Gnar Company blog](https://blog.thegnar.co/code-golf-conditionally-add-to-array).

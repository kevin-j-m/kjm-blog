+++
title = "To Change or Not to Change"
date = 2021-04-18T07:29:25-04:00
tags = ["ruby", "software-design"]
summary = "Here are more questions to try to answer THAT question"
description = "Here is a set of criteria to apply when determining between different options."
+++

## Evaluating Alternatives

- **To Change Or Not To Change**
- [Consistency Conundrum]({{< ref "/consistency-conundrum" >}})

## Optional Feedback

Imagine you've just wrapped up a new feature and submitted it for code review,
which includes the following method:

```ruby
def complex_calculation(base_price, promo_code)
  # calculation
end
```

One of your team members reviews the change, and leaves a suggestion.

> You could consider using keyword arguments for this method.

That would change the method to look like this:

```ruby
def complex_calculation(base_price:, promo_code:)
  # calculation
end
```

Do you make this change? How do you decide?

## Evaluating This Suggestion

At [RailsConf 2019](https://railsconf.com/2019/program/sessions#session-759), I
asked the question, "[I know I can, but should I?](https://www.youtube.com/watch?v=2NiePLJVjNI)"
The talk provides a set of criteria you can use to make a choice between
alternatives. Let's apply that here.

### Impact

From a functionality perspective, this makes no perceived difference. The code
will work the same one way or another and neither have any known or noticeable
implications, such as performance.

However, this alternative arose from a reaction a team member had who also needs
to work on this code, so there must be some value in this suggestion to them.
Additionally, this change, one way or another, will end up serving as prior art
that may be used to base future decisions on.

The change itself has very limited impact on the code itself, but it does look
like there's a possible impact to the team worth exploring.

### Cost

Any change involves risk - and risk has a cost. For example, here we can't only
make the change to the method's signature. We also need to modify any callers
to use the keyword arguments. If we miss one,
and don't have complete test coverage, we may miss a call site - resulting in a
runtime exception when the code is executed in production.

Considering that this is the place this method is introduced, this risk
_should_ be clear to mitigate. Any callers will be in this changeset, so we
should be able to identify them while reviewing this change. Making this change
itself is not particularly risky.

Changing this method to keyword arguments should not take much time - I've
already made the change to the method above! Now we only need to change the
callers. Even if this were an urgent need to get deployed as soon as possible,
it seems reasonable that we could implement this here and now should we choose
to.

### Maintenance

Using keyword arguments here provides two benefits to callers: readability and
flexibility.

I'm going to intentionally obscure variable names to make a point here, but
calling our method before looked like this:

{{< highlight ruby "hl_lines=3" >}}
def print_calculation_results
  calculator = Calculator.new
  result = calculator.complex_calculation(@x, @y)

  puts result
end
{{< /highlight >}}

Compare this to the keyword args approach:

{{< highlight ruby "hl_lines=3-6" >}}
def print_calculation_results
  calculator = Calculator.new
  result = complex_calculation(
    base_price: @x,
    promo_code: @y,
  )

  puts result
end
{{< /highlight >}}

It's more verbose, but subjectively, I think it's also more readable and
understandable.

It also frees us from caring about the position or order of the arguments.
Calling the method this way works as well:

```ruby
  result = complex_calculation(
    promo_code: @y,
    base_price: @x,
  )
```

I would tend to believe that the clarity and flexibility gained from the keyword
arguments would make it easier to maintain code that used this method. It can
also prevent against subtle bugs, where you accidentally meant to pass `@x` as the
base price, but instead you passed `@y` first, because you forgot which argument
should be in what order.

This seems like a maintenance win!

### Consistency

Keyword arguments were added to the ruby langauge in [Ruby 2.0.0](https://www.ruby-lang.org/en/news/2013/02/24/ruby-2-0-0-p0-is-released/), originally
released in 2013. In my experience, its usage is not a surprise to most
rubyists, nor an unwelcome one. Even if someone is unfamiliar with them,
understanding the basic usage of them is well-received.

We'll talk more about consistency at the end.

## Verdict

We have an alternative that has no identified net impact on the function of the
code, but that a team member has a stated preference for. The change would not
require an intense lift to accommodate, nor maintain. It (subjectively) improves
readability and can prevent against subtle errors, and it uses a language
feature that's been supported for many, many years.

In a vacuum, this suggestion seems like a net positive. Given our criteria here,
we should accept this change!

Different changes will be evaluated against the
criteria differently, and the weighted importance of the criteria will change,
but we can still use the rubric as a framework for how to make a decision.

For an in-depth explanation of these criteria, I recommend watching the [full
talk](https://www.youtube.com/watch?v=2NiePLJVjNI&feature=youtu.be).

## Is It That Simple?

I said above that when considering this suggestion in isolation, it seems like a
net benefit. However, this code does not exist in a vacuum. It's part of an
existing application, and needs to fit in with the rest of the ecosystem. I
left out any discussion of consistency with the application itself.

If keyword arguments aren't used anywhere else in the application, should we
reject the change because it's inconsistent? What criteria can we use to
determine how to maintain consistency **and** support change? We'll explore that
in our [next post]({{< ref "/consistency-conundrum" >}}).

> This post originally published on [The Gnar Company blog](https://blog.thegnar.co/to-change-or-not-to-change).

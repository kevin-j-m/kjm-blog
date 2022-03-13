+++
title = "Consistency Conundrum"
date = 2021-04-24T14:14:09-04:00
tags = ["ruby", "software-design"]
summary = "If the only constant is change, how do we keep our code consistent?"
description = "How can we approach being consistent within a codebase while also continuing to evolve?"
canonicalUrl = "https://www.thegnar.com/blog/consistency-conundrum"
+++

## Evaluating Alternatives

- [To Change Or Not To Change]({{< ref "/to-change-or-not-to-change" >}})
- **Consistency Conundrum**

## Consistent Starting Point

In our [previous]({{< ref "/to-change-or-not-to-change" >}}) post, we wrote the following
code:

```ruby
def complex_calculation(base_price, promo_code)
  # calculation
end
```

We received this suggestion in code review:

> You could consider using keyword arguments for this method.

On the whole, we agree that keyword arguments would benefit this method.
However, if keyword arguments are never used in this application, does that mean
we shouldn't do it? We don't want to be inconsistent!

Let's consider some choices we could make specifically in regards to consistency
within our application.

## Options

### Do Nothing

Generally, I don't think this gets the credit it deserves as an
explicitly-defined option. The inertia of the system may be too great to warrant
making the change. The choice to write or structure code in the application this
way may be intentional and necessary. It may introduce too much cognitive
overhead to justify making the change. There are lots of excuses I could make to
justify not doing the work!

I don't think any of these apply to our keyword arguments example. However, if
someone suggested:

> This computation is natively supported in [NumPy](https://numpy.org/). Have
> you considered integrating that?

Then I may want to cross-reference the implementations for correctness, but I
may not want to rewrite my application in Python or explore Ruby/Python bridges
to use that one function. I'm drawing on the consistency of the rest of my
application being written in Ruby to justify that choice.

### Make The Change Everywhere

We could decide to invest the time and energy to use this convention
_everywhere_ immediately. Then, we'd be consistent throughout our application!

In the case of our keyword arguments example, I would advise against that,
unless there are only one or two other places where we'd use them, and they're
easily identified.

Even then, I'd think twice about it. Any change has a risk, and changing the
working code purely in the name of consistency is a risk with limited upside.

However, consider this change:

```ruby
def desired_widgets
  Widget.where("status IN #{user_input}")
end
```

With the following code review suggestion:

> We should modify this method to sanitize user input before sending it as part
> of a database query.

If we created the query like this because it's consistent with how other
queries are written, then it's time to drop everything and change that everywhere.
We've opened ourselves up to [SQL injection](https://owasp.org/www-community/attacks/SQL_Injection)
attacks and we need to immediately invest the time to remediate those issues for
our application's health and safety.

As a side note, consider a static security analysis tool like
[Brakeman](https://brakemanscanner.org/) to [run automatically](https://github.com/TheGnarCo/gnarails/blob/fe72e5fe74455400088d89f7af2a2d9bf1899d26/templates/bin/brakeman)
as part of your [build process](https://github.com/TheGnarCo/gnarails/blob/fe72e5fe74455400088d89f7af2a2d9bf1899d26/templates/.circleci/config.yml#L49-L51)
so that your application is not solely relying on reviewers' eyes to catch
critical security implications.

### Make The Change Here, And Going Forward

For our keyword arguments example, I suggested not to make the change
everywhere. However, that doesn't mean we should ignore it in the name of
consistency. If there's a demonstrated benefit, we should take advantage of
that. Instead, we can embrace the consistency of the standard _going forward_.
As you make future changes, be mindful of current best practices and
conventions, and change existing code to meet those conventions as you have
another reason to change the code.

Lastly, document these conventions so that it's clear to all team members what
the expectations are going forward if you encounter this question. That can
reduce ambiguity and help people understand why this internal inconsistency exists.

This approach of leaving the code better than you found it can apply
to long-reaching goals as well. If you're converting an application from
[JavaScript to TypeScript](https://www.typescriptlang.org/docs/handbook/migrating-from-javascript.html),
you can convert files as you need to change them. If
you're updating an application from [feature specs](https://relishapp.com/rspec/rspec-rails/docs/feature-specs/feature-spec)
to [system specs](https://relishapp.com/rspec/rspec-rails/docs/system-specs/system-spec),
you can do that as you change the area of code the feature spec is testing.

## Being Consistently Inconsistent

Software development is a rapidly evolving ecosystem of best practices, tools,
and approaches. While we don't intend to chase trends, we also need to
incorporate new improvements that will benefit our applications - as long as
we're intentional about them. For a healthy, long-running application, that
can mean applying new paradigms in areas under active development, while leaving
existing work in its current state for the time being.

Living in the two worlds can be uncomfortable, and it is inconsistent. But it's
intentional inconsistency with a plan towards consistency and based on a value
judgement on the cost of the change vs. the benefit it'll provide.

> This post originally published on [The Gnar Company blog](https://blog.thegnar.co/consistency-conundrum).

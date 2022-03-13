+++
title = "Wrapping Up Rails Exceptional Behavior"
date = 2021-03-28T14:15:20-04:00
tags = ["ruby", "rails", "debugging"]
summary = "Putting a bow on our investigation"
description = "In our last post, we encountered some inconsistent behavior between Rails 5 and Rails 6. Now that we've identified the inconsistency, we'll explain why it's happening."
canonicalUrl = "https://www.thegnar.com/blog/wrapping-up-rails-exceptional-behavior"
+++

## Exceptional Behavior in Rails

1. [(W)rapping About Exceptional Behavior In Rails]({{< ref "/wrapping-about-exceptional-behavior-in-rails" >}})
2. **Wrapping Up Rails Exceptional Behavior**

## Reset

In our [last post]({{< ref "/wrapping-about-exceptional-behavior-in-rails" >}}), we
encountered some inconsistent behavior between Rails 5 and Rails 6. In Rails 5,
raising a `RuntimeError` in a controller after rescuing from an
`ActiveRecord::RecordNotFound` exception was still returning a 404 HTTP status
code. In Rails 6, the status code is a 500.

We looked around, and we think we've isolated the area of interest to be in the
`ExceptionWrapper` [class](https://github.com/rails/rails/blob/63d3f3f4d868a5ed9eacf00af2a80278aa005051/actionpack/lib/action_dispatch/middleware/exception_wrapper.rb).

## Revisit The Wrapper

We looked into what was creating our wrapper and discovered that we were always
passing it the `RuntimeError`. After taking a much-needed break, we start
reading the code again, and, almost immediately, we see a [transformation](https://github.com/rails/rails/blob/63d3f3f4d868a5ed9eacf00af2a80278aa005051/actionpack/lib/action_dispatch/middleware/exception_wrapper.rb#L36):

```ruby{3}
def initialize(backtrace_cleaner, exception)
  @backtrace_cleaner = backtrace_cleaner
  @exception = original_exception(exception)
end
```

The exception that is passed in is modified. Let's look at this
`original_exception` [method](https://github.com/rails/rails/blob/63d3f3f4d868a5ed9eacf00af2a80278aa005051/actionpack/lib/action_dispatch/middleware/exception_wrapper.rb#L106-L112).

```ruby
def original_exception(exception)
  if @@rescue_responses.has_key?(exception.cause.class.name)
    exception.cause
  else
    exception
  end
end
```

Recall that our `RuntimeError` is raised as a result of handling an
`ActiveRecord::RecordNotFound` exception. The `RecordNotFound` exception **is** the
[cause](https://ruby-doc.org/core-3.0.0/Exception.html#method-i-cause) of the `RuntimeError`. We previously discovered that `RecordNotFound` is added to `@@rescue_responses` in ActiveRecord's [railtie](https://github.com/rails/rails/blob/d75c2a175215c0f6d011b60f1c9f2b6466184adb/activerecord/lib/active_record/railtie.rb#L22-L27).

The cause of our exception is in the hash, and as such, the **cause** is set as
the `@exception` variable in the initializer. That cause is `RecordNotFound`,
and a `RecordNotFound` exception is supposed to return a 404 status code.

We can now explain why a 404 is returned!

## Regifting (Rails 6 Redux)

We now have a handle on the behavior in Rails 5; however, this investigation
started because we noticed it was different in Rails 5 and Rails 6. Let's check
in on the `ExceptionWrapper` initializer in [Rails 6](https://github.com/rails/rails/blob/0440369d03ae99f9f044b00e39dcd3d9871c65c2/actionpack/lib/action_dispatch/middleware/exception_wrapper.rb#L46-L48).

```ruby
def initialize(backtrace_cleaner, exception)
  @backtrace_cleaner = backtrace_cleaner
  @exception = exception
end
```

No longer are we retrieving the `original_exception`. That doesn't tell the
whole story though. When we ask for the [status code](https://github.com/rails/rails/blob/4c78cc8b04861f02d660aefc37979eb2244db6ba/actionpack/lib/action_dispatch/middleware/exception_wrapper.rb#L62), we're not using `@exception`. Instead, we now have an `unwrapped_exception` to [investigate](https://github.com/rails/rails/blob/4c78cc8b04861f02d660aefc37979eb2244db6ba/actionpack/lib/action_dispatch/middleware/exception_wrapper.rb#L49-L55).

```ruby
def unwrapped_exception
  if wrapper_exceptions.include?(exception.class.to_s)
    exception.cause
  else
    exception
  end
end
```

Rather than looking in `rescue_responses`, we're now looking in
`wrapper_exceptions`, which it appears is a [list](https://github.com/rails/rails/blob/4c78cc8b04861f02d660aefc37979eb2244db6ba/actionpack/lib/action_dispatch/middleware/exception_wrapper.rb#L35-L37) of one exception that should
behave particularly exceptionally.

If the exception is an `ActionView::Template::Error`, then look up the status
code based on the cause of the exception. Otherwise, determine it based on the
exception itself.

`RuntimeError` isn't in this list of `wrapper_exceptions`, so we don't use the
cause (`ActiveRecord::RecordNotFound`) to determine the status code. We use the
`RuntimeError` itself. That has no special handling in `rescue_responses`, so a
500 HTTP status code is returned.

## Thank You Card

The [commit](https://github.com/rails/rails/pull/35049/commits/ef40fb6fd88f2e3c3f989aef65e3ddddfadee814) that makes this change contains a very well-worded description of this scenario, including:

> When the cause is mapped to an HTTP status code the last exception is unexpectedly uwrapped

Thanks to [Yuki Nishijima](https://github.com/yuki24) for fixing this!

> This post originally published on [The Gnar Company blog](https://blog.thegnar.co/wrapping-up-rails-exceptional-behavior).

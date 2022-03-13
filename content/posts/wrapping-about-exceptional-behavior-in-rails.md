+++
title = "(W)rapping About Exceptional Behavior in Rails"
date = 2021-03-21T20:24:01-04:00
tags = ["ruby", "rails", "bundler", "debugging"]
categories = []
summary = "Unwrapping a mystery"
description = "Why is our Rails app returning different HTTP status codes in different versions of Rails? Learn some tips and tricks for navigating a large ruby code base in this post."
canonicalUrl = "https://www.thegnar.com/blog/wrapping-about-exceptional-behavior-in-rails"
+++

## Exceptional Behavior in Rails

1. __(W)rapping About Exceptional Behavior In Rails__
2. [Wrapping Up Rails Exceptional Behavior]({{< ref "/wrapping-up-rails-exceptional-behavior" >}})

## Reset
## Pop Quiz

Consider the following controller:

```ruby
class FoosController < ActionController::API
  rescue_from ActiveRecord::RecordNotFound, with: :raise_not_found

  def create
    raise ActiveRecord::RecordNotFound
    head :ok
  end

  def raise_not_found
    raise "What status code?"
  end
end
```

Here we have an endpoint that that always raises `ActiveRecord::RecordNotFound`.
However, we're [using](https://api.rubyonrails.org/v6.1.1/classes/ActiveSupport/Rescuable/ClassMethods.html)
`rescue_from` to handle this exception, and raise a new `RuntimeError`.

When you hit this controller endpoint, what status code do you expect to
receive?

## It Depends

In Rails 6, we get a 500.

```bash
> curl -I -X POST localhost:3000/foos
HTTP/1.1 500 Internal Server Error
```

However, in Rails 5, we get a 404.

```bash
> curl -I -X POST localhost:3000/foos
HTTP/1.1 404 Not Found
```

Why does this difference exist? Let's dig into how to investigate.

## Research

If you've ever hit the `#show` route for a resource, passing it an ID that doesn't
exist, you may have noticed that Rails responds with a 404 Not Found HTTP status
code. Rails is intercepting the exception that's raised from calling `find`
(`ActiveRecord::RecordNotFound`) and, rather than returning a 500, it knows to
return a 404 instead.

### Start With What You Know

To give us some grounding, let's begin by explicitly stating what we believe to be
true:

- Rails has some special handling of certain exceptions to respond with a
  different HTTP status code.
- If we `rescue_from` one of those exceptions in Rails 5 and re-raise a new
  exception, we'll still see that special handling behavior.
- In Rails 6, with the same implementation, the server now responds with a 500,
  no longer considering it a special case.

### Go Explore

The exception that we know exhibits this behavior is
`ActiveRecord::RecordNotFound`. We also know that Rails returns a 404 Not Found
status code. If we search through the Rails codebase for [RecordNotFound](https://github.com/rails/rails/search?p=1&q=RecordNotFound) or [not_found](https://github.com/rails/rails/search?p=1&q=not_found) (which is the [symbol representation](https://guides.rubyonrails.org/layouts_and_rendering.html#using-render) of the status code), we get a number of results - but there are fewer for `not_found`, so let's look through that.

Towards the bottom of the first page of results (as of this writing), we see the
`ExceptionWrapper` class, which includes [this](https://github.com/rails/rails/blob/291a3d2ef29a3842d1156ada7526f4ee60dd2b59/actionpack/lib/action_dispatch/middleware/exception_wrapper.rb#L8-L24):

```ruby
  cattr_accessor :rescue_responses, default: Hash.new(:internal_server_error).merge!(
      "ActionController::RoutingError"                     => :not_found,
      "AbstractController::ActionNotFound"                 => :not_found,
      "ActionController::MethodNotAllowed"                 => :method_not_allowed,
      "ActionController::UnknownHttpMethod"                => :method_not_allowed,
      "ActionController::NotImplemented"                   => :not_implemented,
      "ActionController::UnknownFormat"                    => :not_acceptable,
      "ActionDispatch::Http::MimeNegotiation::InvalidType" => :not_acceptable,
      "ActionController::MissingExactTemplate"             => :not_acceptable,
      "ActionController::InvalidAuthenticityToken"         => :unprocessable_entity,
      "ActionController::InvalidCrossOriginRequest"        => :unprocessable_entity,
      "ActionDispatch::Http::Parameters::ParseError"       => :bad_request,
      "ActionController::BadRequest"                       => :bad_request,
      "ActionController::ParameterMissing"                 => :bad_request,
      "Rack::QueryParser::ParameterTypeError"              => :bad_request,
      "Rack::QueryParser::InvalidParameterError"           => :bad_request
    )
```

This looks like a big list of exceptions that are mapped to HTTP status codes.
This seems promising - but `ActiveRecord::RecordNotFound` isn't in this hash!

If we search further, we will eventually run into [ActiveRecord's railtie](https://github.com/rails/rails/blob/d75c2a175215c0f6d011b60f1c9f2b6466184adb/activerecord/lib/active_record/railtie.rb#L22-L27),
which includes:

```ruby
config.action_dispatch.rescue_responses.merge!(
  "ActiveRecord::RecordNotFound"   => :not_found,
  "ActiveRecord::StaleObjectError" => :conflict,
  "ActiveRecord::RecordInvalid"    => :unprocessable_entity,
  "ActiveRecord::RecordNotSaved"   => :unprocessable_entity
)
```

This is adding ActiveRecord exceptions to that same hash we saw above in
`ActionDispatch`. We've found where the special handling of mapping exceptions
to status codes occurs!

### Pull Out Some Tools

Returning to the `ExceptionWrapper` class, we see the `status_code_for_exception` [method](https://github.com/rails/rails/blob/291a3d2ef29a3842d1156ada7526f4ee60dd2b59/actionpack/lib/action_dispatch/middleware/exception_wrapper.rb#L117-L119), which looks to take in an exception class and convert it to a status code, based on the `@@rescue_responses` hash.

```ruby
def self.status_code_for_exception(class_name)
  Rack::Utils.status_code(@@rescue_responses[class_name])
end
```

Maybe if we could see what class name is getting passed into that, we could see
if the exception is somehow getting transformed before or after that point. But,
we need a way to get into the rails source code on our running rails app -
that's not our code; how can we do that?

#### `Open`ing A World of Possibilities

Rails 5 appears to be showing confusing results - we're explicitly raising a
`RuntimeError`, but it's returning a 404. As such, let's look in Rails 5.

You can [access the source](https://boringrails.com/tips/bundle-open-debug-gems)
of any dependency using bundler's `open` [command](https://bundler.io/bundle_open.html).
Rails itself is a series of gems, and we can see that this `ExceptionWrapper`
is part of `actionpack`, so let's open that up:

```bash
bundle open actionpack
```

This will open up the source code for the `actionpack` gem in the editor you have
defined. We open the `ExceptionWrapper` class, and we know we want to find out
what value is passed to it, but we're not sure what else we might want to see
while we're there. Using [Ruby's](https://ruby-doc.org/stdlib-3.0.0/libdoc/irb/rdoc/Binding.html) `binding.irb`, we can [start a console](https://jemma.dev/blog/binding-irb) when we hit that method.

```ruby
def self.status_code_for_exception(class_name)
  binding.irb
  Rack::Utils.status_code(@@rescue_responses[class_name])
end
```

After starting a rails server, and issuing a request to our endpoint via curl,
we eventually hit our breakpoint:

```ruby
From: /ruby/2.6.6/lib/ruby/gems/2.6.0/gems/actionpack-5.2.4.5/lib/action_dispatch/middleware/exception_wrapper.rb @ line 86 :

    81:         "Full Trace" => full_trace_with_ids
    82:       }
    83:     end
    84:
    85:     def self.status_code_for_exception(class_name)
 => 86:       binding.irb
    87:       Rack::Utils.status_code(@@rescue_responses[class_name])
    88:     end
    89:
    90:     def source_extracts
    91:       backtrace.map do |trace|

irb(ActionDispatch::ExceptionWrapper):001:0>  class_name
=> "RuntimeError"
```

This isn't surprising that we're getting the `RuntimeError`, but doesn't help
explain how we're getting a 404 returned. Let's exit and regroup on a new
strategy.

```ruby
irb(ActionDispatch::ExceptionWrapper):002:0> exit
Completed 500 Internal Server Error in 2ms (ActiveRecord: 0.0ms)



From: /ruby/2.6.6/lib/ruby/gems/2.6.0/gems/actionpack-5.2.4.5/lib/action_dispatch/middleware/exception_wrapper.rb @ line 86 :

    81:         "Full Trace" => full_trace_with_ids
    82:       }
    83:     end
    84:
    85:     def self.status_code_for_exception(class_name)
 => 86:       binding.irb
    87:       Rack::Utils.status_code(@@rescue_responses[class_name])
    88:     end
    89:
    90:     def source_extracts
    91:       backtrace.map do |trace|

irb(ActionDispatch::ExceptionWrapper):001:0>
```

Wait! This method is called a second time. And that time, the class name is
_still_ `"RuntimeError"`. It gets called a **third** time, and that time, we see:

```ruby
irb(ActionDispatch::ExceptionWrapper):001:0> class_name
=> "ActiveRecord::RecordNotFound"
```

We've found our `NotFound`! But - what do we do now?

### `Call`ing In A Favor

We're still in our console session, and we've executed the `status_code_for_exception` for the third time when processing a single HTTP request. What's calling this method? Ruby will tell us its [caller](https://ruby-doc.org/core-3.0.0/Kernel.html#method-i-caller):

```ruby
irb(ActionDispatch::ExceptionWrapper):002:0> caller
=> ["/ruby/2.6.6/lib/ruby/2.6.0/irb/workspace.rb:85:in `eval'",
"/ruby/2.6.6/lib/ruby/2.6.0/irb/workspace.rb:85:in `evaluate'",
"/ruby/2.6.6/lib/ruby/2.6.0/irb/context.rb:385:in `evaluate'",
"/ruby/2.6.6/lib/ruby/2.6.0/irb.rb:493:in `block (2 levels) in eval_input'",
"/ruby/2.6.6/lib/ruby/2.6.0/irb.rb:647:in `signal_status'",
"/ruby/2.6.6/lib/ruby/2.6.0/irb.rb:490:in `block in eval_input'",
"/ruby/2.6.6/lib/ruby/2.6.0/irb/ruby-lex.rb:246:in `block (2 levels) in each_top_level_statement'",
"/ruby/2.6.6/lib/ruby/2.6.0/irb/ruby-lex.rb:232:in `loop'",
"/ruby/2.6.6/lib/ruby/2.6.0/irb/ruby-lex.rb:232:in `block in each_top_level_statement'",
"/ruby/2.6.6/lib/ruby/2.6.0/irb/ruby-lex.rb:231:in `catch'",
"/ruby/2.6.6/lib/ruby/2.6.0/irb/ruby-lex.rb:231:in `each_top_level_statement'",
"/ruby/2.6.6/lib/ruby/2.6.0/irb.rb:489:in `eval_input'",
"/ruby/2.6.6/lib/ruby/2.6.0/irb.rb:428:in `block in run'",
"/ruby/2.6.6/lib/ruby/2.6.0/irb.rb:427:in `catch'",
"/ruby/2.6.6/lib/ruby/2.6.0/irb.rb:427:in `run'",
"/ruby/2.6.6/lib/ruby/2.6.0/irb.rb:796:in `irb'",
"/ruby/2.6.6/lib/ruby/gems/2.6.0/gems/actionpack-5.2.4.5/lib/action_dispatch/middleware/exception_wrapper.rb:86:in `status_code_for_exception'",
"/ruby/2.6.6/lib/ruby/gems/2.6.0/gems/actionpack-5.2.4.5/lib/action_dispatch/middleware/exception_wrapper.rb:46:in `status_code'",
"/ruby/2.6.6/lib/ruby/gems/2.6.0/gems/actionpack-5.2.4.5/lib/action_dispatch/middleware/debug_exceptions.rb:105:in `render_for_browser_request'",
"/ruby/2.6.6/lib/ruby/gems/2.6.0/gems/actionpack-5.2.4.5/lib/action_dispatch/middleware/debug_exceptions.rb:87:in `render_exception'",
"/ruby/2.6.6/lib/ruby/gems/2.6.0/gems/web-console-3.7.0/lib/web_console/extensions.rb:28:in `render_exception_with_web_console'",
"/ruby/2.6.6/lib/ruby/gems/2.6.0/gems/actionpack-5.2.4.5/lib/action_dispatch/middleware/debug_exceptions.rb:71:in `rescue in call'",
"/ruby/2.6.6/lib/ruby/gems/2.6.0/gems/actionpack-5.2.4.5/lib/action_dispatch/middleware/debug_exceptions.rb:59:in `call'",
"/ruby/2.6.6/lib/ruby/gems/2.6.0/gems/web-console-3.7.0/lib/web_console/middleware.rb:135:in `call_app'",
"/ruby/2.6.6/lib/ruby/gems/2.6.0/gems/web-console-3.7.0/lib/web_console/middleware.rb:30:in `block in call'",
"/ruby/2.6.6/lib/ruby/gems/2.6.0/gems/web-console-3.7.0/lib/web_console/middleware.rb:20:in `catch'",
"/ruby/2.6.6/lib/ruby/gems/2.6.0/gems/web-console-3.7.0/lib/web_console/middleware.rb:20:in `call'",
"/ruby/2.6.6/lib/ruby/gems/2.6.0/gems/actionpack-5.2.4.5/lib/action_dispatch/middleware/show_exceptions.rb:33:in `call'",
"/ruby/2.6.6/lib/ruby/gems/2.6.0/gems/railties-5.2.4.5/lib/rails/rack/logger.rb:38:in `call_app'",
"/ruby/2.6.6/lib/ruby/gems/2.6.0/gems/railties-5.2.4.5/lib/rails/rack/logger.rb:26:in `block in call'",
"/ruby/2.6.6/lib/ruby/gems/2.6.0/gems/activesupport-5.2.4.5/lib/active_support/tagged_logging.rb:71:in `block in tagged'",
"/ruby/2.6.6/lib/ruby/gems/2.6.0/gems/activesupport-5.2.4.5/lib/active_support/tagged_logging.rb:28:in `tagged'",
"/ruby/2.6.6/lib/ruby/gems/2.6.0/gems/activesupport-5.2.4.5/lib/active_support/tagged_logging.rb:71:in `tagged'",
"/ruby/2.6.6/lib/ruby/gems/2.6.0/gems/railties-5.2.4.5/lib/rails/rack/logger.rb:26:in `call'",
"/ruby/2.6.6/lib/ruby/gems/2.6.0/gems/sprockets-rails-3.2.2/lib/sprockets/rails/quiet_assets.rb:13:in `call'",
"/ruby/2.6.6/lib/ruby/gems/2.6.0/gems/actionpack-5.2.4.5/lib/action_dispatch/middleware/remote_ip.rb:81:in `call'",
"/ruby/2.6.6/lib/ruby/gems/2.6.0/gems/actionpack-5.2.4.5/lib/action_dispatch/middleware/request_id.rb:27:in `call'",
"/ruby/2.6.6/lib/ruby/gems/2.6.0/gems/rack-2.2.3/lib/rack/method_override.rb:24:in `call'",
"/ruby/2.6.6/lib/ruby/gems/2.6.0/gems/rack-2.2.3/lib/rack/runtime.rb:22:in `call'",
"/ruby/2.6.6/lib/ruby/gems/2.6.0/gems/activesupport-5.2.4.5/lib/active_support/cache/strategy/local_cache_middleware.rb:29:in `call'",
"/ruby/2.6.6/lib/ruby/gems/2.6.0/gems/actionpack-5.2.4.5/lib/action_dispatch/middleware/executor.rb:14:in `call'",
"/ruby/2.6.6/lib/ruby/gems/2.6.0/gems/actionpack-5.2.4.5/lib/action_dispatch/middleware/static.rb:127:in `call'",
"/ruby/2.6.6/lib/ruby/gems/2.6.0/gems/rack-2.2.3/lib/rack/sendfile.rb:110:in `call'",
"/ruby/2.6.6/lib/ruby/gems/2.6.0/gems/railties-5.2.4.5/lib/rails/engine.rb:524:in `call'",
"/ruby/2.6.6/lib/ruby/gems/2.6.0/gems/puma-3.12.6/lib/puma/configuration.rb:227:in `call'",
"/ruby/2.6.6/lib/ruby/gems/2.6.0/gems/puma-3.12.6/lib/puma/server.rb:706:in `handle_request'",
"/ruby/2.6.6/lib/ruby/gems/2.6.0/gems/puma-3.12.6/lib/puma/server.rb:476:in `process_client'",
"/ruby/2.6.6/lib/ruby/gems/2.6.0/gems/puma-3.12.6/lib/puma/server.rb:334:in `block in run'",
"/ruby/2.6.6/lib/ruby/gems/2.6.0/gems/puma-3.12.6/lib/puma/thread_pool.rb:135:in `block in spawn_thread'"
]
```

Starting from the bottom, we can see the full path that leads us to executing
`status_code_for_exception`. Tracing our way backwards from
`status_code_for_exception`, we eventually find where the `ExceptionWrapper` is
[created](https://github.com/rails/rails/blob/63d3f3f4d868a5ed9eacf00af2a80278aa005051/actionpack/lib/action_dispatch/middleware/debug_exceptions.rb#L78):

```ruby
def render_exception(request, exception)
  backtrace_cleaner = request.get_header("action_dispatch.backtrace_cleaner")
  wrapper = ExceptionWrapper.new(backtrace_cleaner, exception)
  ...
```

We drop a `binding.irb` in here, and it looks like the exception is always our
`RuntimeError`. It looks like something inside of `ExceptionWrapper` is doing
_something_ to change this to an `ActiveRecord::RecordNotFound` exception.

### Walk Away

We've been at this for a while, and we've got a lead to track down, but at this
point, let's take a break. We should clean up our mess by [restoring](https://bundler.io/man/bundle-pristine.1.html)
`actionpack` back to how it was before.

```bash
bundle pristine
```

Let's take stock in what we've done:

- Identified that Rails has a list of exceptions that it maps to specific HTTP
  status codes.
- Used `bundle open` to manipulate the source code of our dependency.
- Inserted a `binding.irb` to play around in a method that we think is
  interesting, and asked where it's being invoked with `caller`.
- Confirmed that what's calling our class in question is always passing it a `RuntimeError`.

We still don't know where the `ActiveRecord::RecordNotFound` exception is coming
from.

We'll take another look at this with fresh eyes in our [next post]({{< ref "/wrapping-up-rails-exceptional-behavior" >}}).

> This post originally published on [The Gnar Company blog](https://blog.thegnar.co/wrapping-about-exceptional-behavior-in-rails).

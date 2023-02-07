+++
title = "Access Request Headers in a Rails Controller"
date = 2024-07-11T18:04:22-05:00
tags = ["ruby", "rails"]
summary = "Heading into Rails source code to find an answer"
description = "This post explores where to access request headers by looking at the Rails source code after unexpectedly accessing the response headers."
+++

## Heads Up

A coworker presented a failing request spec. They asked if they were passing headers incorrectly in the test.

```ruby
it "reports to be a teapot when asked to brew coffee" do
  headers = { "X-COMMAND" => "brew coffee" }
  get drinks_url, headers: headers

  expect(response.status).to eq 418
end
```

They wrote the test exactly like I'd [expect](https://rspec.info/features/6-0/rspec-rails/request-specs/request-spec/). But, rather than providing the 418, a 200 OK was the status code. I then looked at the controller this request spec was accessing.

```ruby
def index
  if headers["X-COMMAND"] == "brew coffee"
    head 418
  end

  @drinks = Drink.all
end
```

Nothing *obvious* caught my attention. But now that I'd been effectively [nerd sniped](https://xkcd.com/356/), I had to figure out what was going on.

## Heading In For a Closer Look

I added a breakpoint inside the controller to inspect the headers when the test was running.

```ruby
irb:001:0> headers
=>
{"X-Frame-Options"=>"SAMEORIGIN",
 "X-XSS-Protection"=>"0",
 "X-Content-Type-Options"=>"nosniff",
 "X-Download-Options"=>"noopen",
 "X-Permitted-Cross-Domain-Policies"=>"none",
 "Referrer-Policy"=>"strict-origin-when-cross-origin"}
```

As expected, given the failing test, the `X-COMMAND` header was nowhere to be found. But luckily, they did seem familiar to me. They looked to be Rails' [default headers](https://github.com/rails/rails/blob/f1aa436d738af1852b610189aeb93a5609bfe3b0/actionpack/lib/action_dispatch/railtie.rb#L30). But those [default headers](https://edgeguides.rubyonrails.org/configuring.html#config-action-dispatch-default-headers) are for the __response__, not the *request*.

I still had my console session with my breakpoint, so I asked what kind of headers we were interacting with.

```ruby
irb:002:0> headers.class
=> ActionDispatch::Response::Header
```

This confirmed we were dealing with the response, not request, headers.

## Heads Down

I needed to trace my way backwards from what I have or know. I asked what defines the headers method by asking for the [source location](https://ruby-doc.org/3.2.1/Method.html#method-i-source_location). That'll tell me the file and line number.

```ruby
irb:003:0> method("headers").source_location
=> [".../gems/actionpack-7.0.3.1/lib/action_controller/metal.rb", 147]
```

[That line](https://github.com/rails/rails/blob/7-0-stable/actionpack/lib/action_controller/metal.rb#L147) shows `headers` delegated to an [internal attribute](https://guides.rubyonrails.org/active_support_core_extensions.html#internal-attributes) `@_response`.

```ruby
delegate :headers, :status=, :location=, :content_type=,
         :status, :location, :content_type, :media_type, to: "@_response"
```

That internal attribute is accessible in the controller by calling `response`. We can see that from the `attr_internal` definition on [line 145](https://github.com/rails/rails/blob/7-0-stable/actionpack/lib/action_controller/metal.rb#L145).

```ruby
attr_internal :response, :request
```

`response` isn't the ONLY internal attribute on that line though. There's ALSO a `request`. In our console, let's see what that request is.

```ruby
irb:004:0> request.class
=> ActionDispatch::Request
```

That class also [responds to](https://api.rubyonrails.org/classes/ActionDispatch/Request.html#method-i-headers) `headers`, providing the [request headers](https://api.rubyonrails.org/classes/ActionDispatch/Http/Headers.html).

## Heading In For The Close

The change to get our test to pass is small. We don't want the response headers, which is what the `headers` variable is. We need the request headers, which are accessible at `request.headers`.

```ruby
def index
  if request.headers["X-COMMAND"] == "brew coffee"
    head 418
  end

  @drinks = Drink.all
end
```

Now that we're accessing the headers of the __request__ our test passes.

Naming is hard. Asking for a controller's headers could be either the request or the response headers. Turns out, Rails will give you the response headers. To access the request headers, explicitly ask for them from the request object.

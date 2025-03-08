+++
title = "Preserving Flash Messages in Rails"
date = 2025-03-14T08:00:22-04:00
tags = ["ruby", "rails"]
summary = "Check out this flash sale!"
description = "This post explains part of the conditions by which Rails determines to preserve flash messages."
+++

## Flash Sale!

We're offering our best deals on select products for a limited time. We're going to link to this flash sale from many different pages on our site. The call-to-action (CTA) we display at the top of the flash sale page will change based on which page you access the flash sale from.

We're going to store that text in a flash message. For example, let's say you're looking for the contact information for everyone at the shop. If you click a link on that page to visit the flash sale, we reference that you were just on the contact page.

```ruby
class ContactsController < ApplicationController
  def index
    if FlashSale.on?
      flash[:sale] = "Thanks to you for looking to contact us"
    end
  end
end
```

Keep in mind this sale may be over in...a flash. In between loading the contacts page and clicking the link to view the flash sale, it may be over. If that's the case, we still want to show the CTA. However, we'll send you to the general products page, rather than the flash sale page.

We sample some of our requests to log information about them. If we're sampling this request, and they're getting redirected because the flash sale is over, we want to log what CTA brought them there. That way we can figure out what page they were on.

```ruby
class FlashSalesController < ApplicationController
  def index
    if FlashSale.off?
      if AttemptLogger.log?
        Rails.logger.info "CTA '#{flash[:sale]}' used to access flash sale after it's over"
      end

      redirect_to products_path and return
    end

    @products = [
      donner_red_hss_starter_kit,
      martin_junior_acoustic,
      squier_affinity_strat_junior_hss,
    ]
  end
end
```

The view for the products page displays the flash message. We still have the experience of referencing the page they came from. Even if it isn't our best deals.

```sh
<h1>All Products</h1>

<% if flash[:sale].present? %>
  <p> <%= flash[:sale] %> </p>
<% end %>
```

## Gone In A Flash

Let's confirm this behavior by writing some [system tests](https://guides.rubyonrails.org/testing.html#system-testing). We'll start by verifying the redirect when the flash sale ends before they can view the deals.

```ruby
it "redirects to the products page with the flash message when the flash sale has ended" do
  allow(FlashSale).to receive(:on?).and_return(true)
  visit contacts_path

  allow(FlashSale).to receive(:off?).and_return(true)
  click_link "View Deals"

  expect(page).to have_selector "h1", text: "All Products"
  expect(page).to have_content "Thanks to you for looking to contact us"
end
```

Our test passes!

```sh
⇒ rspec spec/system/flash_sales_spec.rb:8
Run options: include {:locations=>{"./spec/system/flash_sales_spec.rb"=>[8]}}
.

Finished in 0.12213 seconds (files took 1.89 seconds to load)
1 example, 0 failures
```

Next we'll iterate on this by verifying the same user behavior when we log the CTA.

```ruby
it "logs the flash message when the flash sale has ended and the log sampler wants the message" do
  allow(FlashSale).to receive(:on?).and_return(true)
  visit contacts_path

  allow(FlashSale).to receive(:off?).and_return(true)
  allow(AttemptLogger).to receive(:log?).and_return(true)
  click_link "View Deals"

  expect(page).to have_selector "h1", text: "All Products"
  expect(page).to have_content "Thanks to you for looking to contact us"
end
```

Unfortunately, we have a different result here.

```sh
⇒ rspec spec/system/flash_sales_spec.rb:17
Run options: include {:locations=>{"./spec/system/flash_sales_spec.rb"=>[17]}}
F

Failures:

  1) Flash Sale logs the flash message when the flash sale has ended and the log sampler wants the message
     Failure/Error: expect(page).to have_content "As thanks to you for looking to contact us"
       expected to find text "As thanks to you for looking to contact us" in "All Products"
     # ./spec/system/flash_sales_spec.rb:26:in `block (2 levels) in <top (required)>'

Finished in 0.21189 seconds (files took 1.98 seconds to load)
1 example, 1 failure

Failed examples:

rspec ./spec/system/flash_sales_spec.rb:17 # Flash Sale logs the flash message when the flash sale has ended and the log sampler wants the message
```

Our flash message is nowhere to be found. For an almost identical test. After [searching]( {{< ref "searching-for-a-reason" >}} ) through the documentation, we find [this](https://api.rubyonrails.org/classes/ActionDispatch/Flash.html):

> Anything you place in the flash will be exposed to the very next action and then cleared out.

That leaves us even *more* confused. Our first test passed, even though we went from the flash sale endpoint to the products endpoint. Is that two actions? And if not, why did it clear it before exposing it to the user when we log the message?

## Viewing Source In A Flash

That documentation tells us where to find the implementation for the flash: `actionpack/lib/action_dispatch/middleware/flash.rb`. We can use [bundle open](https://bundler.io/man/bundle-open.1.html) to explore the source code.

```sh
$ bundle open actionpack
```

We navigate to the flash file and look around. In there we see a [method](https://github.com/rails/rails/blob/v8.0.1/actionpack/lib/action_dispatch/middleware/flash.rb#L71) called `commit_flash`. Based only on the name, we guess that might have something to do with keeping the values of the flash. In one of our tests, we aren't keeping the values. Did we find a bug in Rails?

We drop a breakpoint in the method, changing Rails' source code (temporarily) for our application. Then we can run our tests again to investigate.

```ruby
 def commit_flash
+  binding.irb # Added by us!
   return unless session.enabled?

   if flash_hash && (flash_hash.present? || session.key?("flash"))
     session["flash"] = flash_hash.to_session_value
     self.flash = flash_hash.dup
   end

   if session.loaded? && session.key?("flash") && session["flash"].nil?
     session.delete("flash")
   end
 end
```

## A Flash In The Pan

We observe the following behavior when running our tests:

1. When we don't log the message in the `FlashSalesController#index` method, the value of `flash_hash` in `commit_flash` is `nil` at the end of processing the flash sale index action.
2. When we *do* log the message, `flash_hash` has a value.

That still seems backwards from the behavior we're seeing. When we try to show the flash message on the products page, it's not there when we log the message. But that's the scenario where `flash_hash` has a value. Now we need to take a detour to understand where this `flash_hash` comes from. We find it's a [method](https://github.com/rails/rails/blob/v8.0.1/actionpack/lib/action_dispatch/middleware/flash.rb#L67).

```ruby
def flash_hash
  get_header Flash::KEY
end
```

That leads us to wonder what can *set* that header. And we see [just above](https://github.com/rails/rails/blob/v8.0.1/actionpack/lib/action_dispatch/middleware/flash.rb#L63):

```ruby
def flash=(flash)
  set_header Flash::KEY, flash
end
```

As our eyes continue to look up, we find a [method](https://github.com/rails/rails/blob/v8.0.1/actionpack/lib/action_dispatch/middleware/flash.rb#L57) that calls `flash=`.

```ruby
def flash
  flash = flash_hash
  return flash if flash
  self.flash = Flash::FlashHash.from_session_value(session["flash"])
end
```

This feels a bit circular in the beginning. We look to see if `flash_hash` is set and return that if so. But we're looking here to see how `flash_hash` could get a value. It is the *last* line that's setting the header that `flash_hash` uses. So, that's called when `flash_hash` doesn't have a value. The flash object is grabbed from the session and converted to a `FlashHash`.

That will store that `FlashHash` instance in the header. We can then access that by calling the `flash_hash` method.

We also see a [comment](https://github.com/rails/rails/blob/v8.0.1/actionpack/lib/action_dispatch/middleware/flash.rb#L54C9-L54C86) above the `flash` method.

```ruby
# Access the contents of the flash. Returns a ActionDispatch::Flash::FlashHash.
```

*This* is what we're calling when we call `flash[:sale]` in our logging message. `FlashHash` defines the `[]` [method](https://github.com/rails/rails/blob/v8.0.1/actionpack/lib/action_dispatch/middleware/flash.rb#L169).

## Flash Back

Let's revisit each conditional in `commit_flash` and explore how they relate to our test scenarios.

```ruby
if flash_hash && (flash_hash.present? || session.key?("flash"))
  session["flash"] = flash_hash.to_session_value
  self.flash = flash_hash.dup
end
```

When this is called after the `FlashSalesController#index` action and we *don't* log a message, `flash_hash` is `nil`. That's because we never call `flash`, which would set the header that `flash_hash` reads from. `flash` never gets called, so the header never gets set. We never enter the conditional block in that scenario.

However, when we do log the message, the header is there, and `flash_hash` is present. That causes the contents of `flash_hash` to be converted to a session value and stored back in the session.

Converting a `FlashHash` instance to its [session value](https://github.com/rails/rails/blob/v8.0.1/actionpack/lib/action_dispatch/middleware/flash.rb#L143) does the following:

```ruby
def to_session_value
  flashes_to_keep = @flashes.except(*@discard)
  return nil if flashes_to_keep.empty?
  { "discard" => [], "flashes" => flashes_to_keep }
end
```

When we make a flash *from* the session, we send [all the keys](https://github.com/rails/rails/blob/v8.0.1/actionpack/lib/action_dispatch/middleware/flash.rb#L135) as [discard values](https://github.com/rails/rails/blob/v8.0.1/actionpack/lib/action_dispatch/middleware/flash.rb#L150). As a result, when we're converting back *to* the session, `flashes_to_keep` is empty. This returns `nil`. And that's what we set `session["flash"]` to.

The second conditional in `commit_flash` is:

```ruby
if session.loaded? && session.key?("flash") && session["flash"].nil?
  session.delete("flash")
end
```

When we log the message, we load the session by calling `flash[:sale]`. That's because we're reading from the session to pass a value to `FlashHash.from_session_value`.

```ruby
def flash
  flash = flash_hash
  return flash if flash
  self.flash = Flash::FlashHash.from_session_value(session["flash"])
end
```

Accessing the session [loads the session for reading](https://github.com/rails/rails/blob/v8.0.1/actionpack/lib/action_dispatch/request/session.rb#L115), which through the `load!` [method](https://github.com/rails/rails/blob/v8.0.1/actionpack/lib/action_dispatch/request/session.rb#L256) sets the `@loaded` [instance variable](https://github.com/rails/rails/blob/v8.0.1/actionpack/lib/action_dispatch/request/session.rb#L280) that is used to [determine if a session is loaded](https://github.com/rails/rails/blob/v8.0.1/actionpack/lib/action_dispatch/request/session.rb#L236).

Because the session is loaded, it has a flash key, __and__ the value is `nil` (because converting it to a session value discarded all the keys), then we delete flash from the session entirely.

If we never invoke `flash` before redirecting, we never load it from the session, never convert it back to a session value, and never delete it from the session. It remains available when processing the `ProductsController#index` action.

When we do call `flash` to write the log message, we *do* delete it from the session, so `ProductsController#index` doesn't have it.

## Flash Forward

This is great that we understand *why* this is happening. But none of this actually fixes our problem. We want our tests to pass. We *don't* want the flash removed from the session. We want the `:sale` key to still exist.

Recall the conditions by which the flash is deleted from the session:

```ruby
if session.loaded? && session.key?("flash") && session["flash"].nil?
  session.delete("flash")
end
```

When we log the message, we need to load the session to access the flash, so we can't get around that. However, if the value of `session["flash"]` wasn't `nil`, that'd be enough to preserve it.

The reason `session["flash"]` is `nil` is because we didn't have any flashes to keep when converting the object to a session value.

```ruby
def to_session_value
  flashes_to_keep = @flashes.except(*@discard)
  return nil if flashes_to_keep.empty?
  { "discard" => [], "flashes" => flashes_to_keep }
end
```

We do have an option to explicitly keep the entire flash (or a particular key). It's the aptly-named [keep](https://github.com/rails/rails/blob/v8.0.1/actionpack/lib/action_dispatch/middleware/flash.rb#L250) method. That removes keys from the `@discard` set, so they'll be retained.

Let's update our flash sale controller to use this.

```ruby
 class FlashSalesController < ApplicationController
   def index
     if FlashSale.off?
       if AttemptLogger.log?
         Rails.logger.info "CTA '#{flash[:sale]}' used to access flash sale after it's over"
+        flash.keep(:sale)
       end

       redirect_to products_path and return
     end

     @products = [
       donner_red_hss_starter_kit,
       martin_junior_acoustic,
       squier_affinity_strat_junior_hss,
     ]
   end
 end
```

With that one call to `keep`, both of our tests pass. Whether we log the flash CTA or not, it's available for the `ProductsController#index` action to display.

## An Illuminating Flash

Our goal was to preserve a flash message. Instead, it seems we learned more about the conditions by which we destroy the flash message. That allowed us to back up and explore options to fail that conditional.

The Rails source code *is* vast and dense, but it's also readily available. We discovered the reason behind the behavior we saw __and__ a solution by being willing to explore it. Consider `bundle open`ing the next dependency you're confused or interested by. See what you can learn. It may take a while, or you may have your answer in a flash.

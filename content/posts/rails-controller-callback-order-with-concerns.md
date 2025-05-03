+++
title = "Rails Controller Callback Order With Concerns"
date = 2025-05-16T20:53:21-04:00
tags = ["ruby", "rails"]
summary = "Stay on the line for the next available representative"
description = "This post explores how concerns with controller callbacks are ordered against other callbacks"
+++

## How can I help you today?

We're building an app to display the prompts from an automated phone system. The system will read these out when people call our technical support phone number. We use a [callback](https://guides.rubyonrails.org/action_controller_overview.html#controller-callbacks) to set the list of prompts to read. Whether you're calling for details about consuming our GraphQL endpoints or looking for tips on charging your phone battery, you're going to hear these messages first.

Our controller action renders that list.

```ruby
class TechSupportPromptsController < ApplicationController
  before_action :set_prompts

  def show
    render inline: "<%= @prompts.join(' - ') %>"
  end

  private

  def set_prompts
    @prompts = []
    @prompts << "Thank you for calling technical support."
    @prompts << "Call volume is higher than expected."
    @prompts << "Press 9 to receive a call back when an agent is available."
  end
end
```

If you're curious about avoiding a view for rendering, you can read more about it in the [Rails Guides](https://guides.rubyonrails.org/layouts_and_rendering.html#using-render-with-inline). Pay attention to the part where it advises against using it in most cases! I'm doing it here for brevity, which I'm now diminishing the value of by spending a paragraph explaining it.

## Making a test call

We want to make sure that this is working as expected, so let's write a test.

```ruby
it "displays the prompts in the proper order" do
  get tech_support_prompt_path

  expect(response.body.split(" - ")).to eq [
    "Thank you for calling technical support.",
    "Call volume is higher than expected.",
    "Press 9 to receive a call back when an agent is available."
  ]
end
```

This test passes, so we ship it.

## Sharing the call (Party Line)

We're preparing for our next feature. We will display the prompts for our general customer support phone number. After reading the requirements, we see some consistency. When calling tech support or customer support, we end by telling the caller how to request a callback.

Before starting on this, we're going to prepare the tech support display for some reuse. We want to share the code that adds these common prompts at the end of the interaction. As a starting point, we separate these later prompts to a separate method. We then invoke it in a separate before action.

```ruby
class TechSupportPromptsController < ApplicationController
  before_action :set_prompts
  before_action :add_callback_notices

  def show
    render inline: "<%= @prompts.join(' - ') %>"
  end

  private

  def set_prompts
    @prompts = []
    @prompts << "Your call is important to us."
  end

  def add_callback_notices
    @prompts << "Call volume is higher than expected."
    @prompts << "Press 9 to receive a call back when an agent is available."
  end
end
```

Our test passes, because these callbacks are [invoked in the order in which they're written](https://reganchan.ca/blog/ordering-of-filters-in-rails-controllers/).

> When using `before_action`, the filters are called in the order that they are defined

Our next step to reuse these is to move these common prompts into a [concern](https://api.rubyonrails.org/classes/ActiveSupport/Concern.html). This will allow us to access the prompts in different controllers. Even better, we surmise, the concern itself can define the `before_action`. Now all the controller needs to do is include the concern.

```ruby
module Callbackable
  extend ActiveSupport::Concern

  included do
    before_action :add_callback_notices
  end

  private

  def add_callback_notices
    @prompts << "Call volume is higher than expected."
    @prompts << "Press 9 to receive a call back when an agent is available."
  end
end
```

And now we include the concern in the controller.

```ruby
class TechSupportPromptsController < ApplicationController
  include Callbackable

  before_action :set_prompts
end
```

Running our test doesn't pass. Instead it raises an exception!

```shell
NoMethodError:
  undefined method `<<' for nil
  # ./app/controllers/concerns/callbackable.rb:6:in `block (2 levels) in <module:Callbackable>'
```

The concern is attempting to shovel a message onto a variable that is currently `nil`. `nil` does not respond to `<<`. The `@prompts` variable is not initialized at this point. The rule about callback order applies here for the concern as well.

We include the concern before calling the callback which initializes `@prompts`. Therefore `@prompts` has no value.

We can correct for that by including the concern after the callback.

```ruby
class TechSupportPromptsController < ApplicationController
  before_action :set_prompts
  include Callbackable
end
```

Our test passes again.

## Calling our taste into question

That may not sit well with you. Maybe it offends your sensibilities. Perhaps there are other include statements for your controller. You may want to put them all together, right after defining the class for the controller. However, the order of this include is important. It *must* be after the other `before_action`.

Even if you're totally fine with the include being anywhere, it's unclear that it needs to be after the callback. Luckily we have a test that will catch if someone inadvertently moves it. Even with the test, it's not obvious that the ordering is important. We have no idea what the concern does without looking at it. We don't know there's a callback running in it.

Alternatively, we can modify the concern to not specify the callback. It still defines the `add_callback_notices` method.

```ruby
module Callbackable
  extend ActiveSupport::Concern

  private

  def add_callback_notices
    @prompts << "Call volume is higher than expected."
    @prompts << "Press 9 to receive a call back when an agent is available."
  end
end
```

In our controller, we have access to the method by including the concern. However, it's again our responsibility to call it. We do so by adding a second callback, just like we did when this method existed in the controller.

```ruby
class TechSupportPromptsController < ApplicationController
  include Callbackable

  before_action :set_prompts
  before_action :add_callback_notices
end
```

In this scenario, we still need to know the rule about what order Rails applies these callbacks. I personally find it more clear to see the callback defined in the controller anyway. It also gives you the freedom to include the concern anywhere. Perhaps with the rest of the modules you're including! You don't need to be concerned with the order of includes.

Give me [a call]({{< ref "about/#elsewhere-on-the-internet" >}}) to let me know what you think.

+++
title = "ActiveModel Conditional Validations"
date = 2024-10-20T18:00:22-04:00
tags = ["ruby", "rails"]
summary = "Do you validate parking?"
description = "There are options to applying validations conditionally in ActiveModel, but consider other ways to model your rules."
+++

## Validating The Problem Domain

Today we're modeling a system that involves providing validated parking. It's a benefit provided by different businesses. They offer their customers a discount at a nearby parking lot. The lot has the following rules:

1. All parking validation tickets must have an expiration time when they're issued.
2. The parking lot owner has a special ticket that does not expire. Instead, the owner provides a PIN after scanning their ticket for free parking.
3. Please avoid hitting parked cars.

The last rule is more of a guideline than anything and won't factor into our modeling.

We will use the `ActiveModel::Validations` [module](https://api.rubyonrails.org/classes/ActiveModel/Validations.html) to implement these system restrictions.

## Parking Expiration

Parking validation tickets must have an expiration time. We will use a [presence validation](https://guides.rubyonrails.org/active_record_validations.html#presence) to help enforce that.

```ruby

class ValidatedParkingTicket
  include ActiveModel::Validations

  attr_accessor :expires_at
  validates :expires_at, presence: true
end
```

We will exercise this with a test.

```ruby
it "is invalid without an expiration timestamp" do
  ticket = ValidatedParkingTicket.new
  expect(ticket.valid?).to eq false

  ticket.expires_at = Time.current
  expect(ticket.valid?).to eq true

end
```

By including the `ActiveModel::Validations` module, we gain the ability to set validations. You may have seen these in `ActiveRecord` models. We can also determine if the instance is valid, as well as seeing any errors.

## Owner PIN

Our second rule states that there is a special ticket. One that the owner has with a PIN.

```ruby
class ValidatedParkingTicket
  include ActiveModel::Validations

  attr_accessor :owner_pin
end
```

It's not enough to have the ability to get and set a PIN for the owner's ticket. We need to make sure that the owner's ticket has a PIN. But other kinds of tickets explicitly should not have a PIN. They're not an owner!

You may have seen validations that only apply during certain lifecycle events. Perhaps you've seen something like a field only needs to be present when creating an instance. Or another attribute must be greater than one when updating the record.

These conditional validations use the `on` option. And these lifecycle events are contexts within which the validation is applicable.

However, you're not limited to these events. You can define a [custom context](https://guides.rubyonrails.org/active_record_validations.html#on).

Let's do that to make sure that the PIN is required within the context of an owner.

```ruby
class ValidatedParkingTicket
  include ActiveModel::Validations

  attr_accessor :owner_pin
  validates :owner_pin, presence: true, on: :owner
end
```

We must explicitly tell ActiveModel's methods about the context.

```ruby

it "requires a PIN when applying the owner context" do
  ticket = ValidatedParkingTicket.new
  expect(ticket.valid?(:owner)).to eq false

  ticket.owner_pin = "1234"
  expect(ticket.valid?(:owner)).to eq true
end
```

When we don't specify the context, the validation does not apply.

```ruby

it "does not require a PIN without the owner context" do
  ticket = ValidatedParkingTicket.new(expires_at: Time.current)

  expect(ticket.valid?).to eq true
end
```

## Owner Expiration

Recall that we also require a parking validation to have an expiration timestamp. That doesn't apply for the owner. Their ticket is always valid. As long as they can provide the PIN following swiping their ticket, they don't pay to park.

To accomplish this, we will use another form of conditional validations, the `if` [option](https://guides.rubyonrails.org/active_record_validations.html#using-a-proc-with-if-and-unless).

```ruby
class ValidatedParkingTicket
  include ActiveModel::Validations

  attr_accessor :expires_at
  validates :expires_at, presence: true, if: -> { validation_context != :owner }
end
```

Note the use of `validation_context`. We're still using contexts here to know whether the expiration applies. This allows us to find out [what context](https://api.rubyonrails.org/classes/ActiveModel/Validations.html#method-i-validation_context) we're operating in. If it's not an owner, then we require the expiration timestamp.

```ruby
it "is valid without an expiration date when applying the owner context" do
  ticket = ValidatedParkingTicket.new(owner_pin: "1234")

  expect(ticket.valid?(:owner)).to eq true
end
```

## Stay in Your Spot

Put together, we have the following setup to model our rules.

```ruby
class ValidatedParkingTicket
  include ActiveModel::Validations

  attr_accessor :expires_at
  validates :expires_at, presence: true, if: -> { validation_context != :owner

  attr_accessor :owner_pin
  validates :owner_pin, presence: true, on: :owner
end
```

You might notice we have very divergent behavior. Our validations depend on the context we're operating in. We need to remember to apply the context in the appropriate situations. That involves using some `ActiveModel` functionality that may be less familiar to some.

This can lead to situations where our data is unclearly validated. Maybe we forget one time to apply the owner context. This happens when setting up a new ticket for the owner after they lose their original ticket. They'll be coming back soon frustrated when their ticket expired.

We can alternatively avoid these conditional validations entirely. Instead, we can treat these as separate entities. Most of the time, visitors receive these tickets from businesses. They require an expiration date and don't know anything about a PIN.

```ruby
class VisitorParkingTicket
  include ActiveModel::Validations

  attr_accessor :expires_at
  validates :expires_at, presence: true
end
```

The owner has a special ticket. It has a PIN and no concept of an expiration timestamp.

```ruby
class LotOwnerParkingTicket
  include ActiveModel::Validations

  attr_accessor :pin
  validates :pin, presence: true
end
```

These validations are unambiguous. A `LotOwnerParkingTicket` must have a PIN. The naming of the class itself makes it clear what kind of ticket we're dealing with.

This can also help with behavior that's specific to the types of tickets. Let's say we need to determine if a parking ticket has expired. For a owner ticket, that doesn't apply. It's valid as long as they provide the proper PIN. For the visitor, the ticket expires at a certain time.

We can apply those concepts without needing to intertwine the different cases. We do that with separate classes here.

## Exiting the Lot

There are options to conditionally applying validations to classes. We can specify the context(s) within which certain validations apply. We can use `if` (and its counterpart `unless`) to provide limitations on when the validation takes place.

When you're reaching for these types of validations, stop for a moment. Consider if there's a different modeling approach that'll apply the validations. This does not mean to never use this functionality. They're valid and have their place. When introducing them, consider it an opportunity to revisit your approach. Think if there's another way to organize your rules.

And watch out for parked cars.

+++
title = "Finding an Initially Confusing Result in Rails"
date = 2022-03-14T07:30:41-04:00
tags = ["ruby", "rails"]
summary = "Blocking off some time to explain what feels like inconsistent behavior"
description = "Rails' find_or_initialize_by method accepts an optional block, but that block is only executed when an existing record is not found by the provided attributes."
+++

## Initial Impressions

We run into an old friend Bob and a new friend Carol on the street. Bob recently got married and changed their last name. We're meeting Carol for the first time. We update our mental Rolodex of friends during this meeting.

```ruby
Friend.create(first_name: "Bob", last_name: "Smith")
Friend.where(first_name: "Carol").count
=> 0

# Chance encounter on the street

bob = Friend.find_or_initialize_by(first_name: "Bob") do |friend|
  friend.last_name = "Jones"
end

carol = Friend.find_or_initialize_by(first_name: "Carol") do |friend|
  friend.last_name = "Thompson"
end
```

What are the values of Carol's and Bob's last names in our working memory right now during this encounter?

### Carol's Last Name

Carol's last name is Thompson. We have no other friends named Carol, so we create a new object and in the block, set their last name to Thompson.

```ruby
carol.last_name
=> "Thompson"
```

### Bob's Last Name

Bob's last name is *still* "Smith", the value it's persisted in our database as.

```ruby
bob.last_name
=> "Smith"
```

Even though in our block, we set their last name to Jones, it didn't take affect. It seems we forgot our friend's new last name! This could lead to an embarrassing situation later on in the conversation. What happened?

## Finding The Source of the Confusion

Thanks to Rails' [documentation](https://api.rubyonrails.org/v7.0.2/classes/ActiveRecord/Relation.html#method-i-find_or_initialize_by), we discover the `find_or_initialize_by` method is in `ActiveRecord::Relation`. From there, we can look at the [source code](https://github.com/rails/rails/blob/de53ba56cab69fb9707785a397a59ac4aaee9d6f/activerecord/lib/active_record/relation.rb#L226) of the method:

```ruby
def find_or_initialize_by(attributes, &block)
  find_by(attributes) || new(attributes, &block)
end
```

## Attributing The Difference

If we find an existing record by the attributes provided, then we return that record. If not, `find_by` returns `nil` and we visit the right-hand side of the expression. That will create a new record with the attributes and [pass the block]({{< ref "/activerecord-new-block" >}}) to `new`. Notice that the block is not executed at all when `find_by` returns a record.

## Mistakenly Blocking Out Our Friend's New Name

That explains why Bob's last name isn't updated in our memory. We documented the change, but it never took effect. `find_or_initialize_by` didn't use the block. The saved representation for Bob returned from the method without executing the block.

When using `find_or_initialize_by` with a block, pay careful attention. The block will only execute in one of those conditions - when initializing an object.

## Clarifying Our Initial Intent

With this information, let's consider when and how we want to evaluate the code in the block.

### Update Neither Found Nor New Records

Passing a block to `find_or_initialize_by` is *optional*. When the criteria we're using to find a record is all we want our new record to have, there's no need to supply the block.

```ruby
bob = Friend.find_or_initialize_by(first_name: "Bob")
```

Our goal right now is to change Bob's last name. This alone does not get us there.

### Only Update New Records

This is what our initial implementation is doing. As a reminder, we have:

```ruby
bob = Friend.find_or_initialize_by(first_name: "Bob") do |friend|
  friend.last_name = "Jones"
end
```

We’re searching for our friend named “Bob”. When we find one, we get our friend back from the database. When we don't, we instantiate a new friend and also set their last name as “Jones”.

That may be confusing to yourself and others reading this code in the future. It may not be clear when the block executes. As an alternative, we could choose to explicitly call that out. A friend that’s not persisted will respond to `new_record?` with `true`, and we can use that to only update their last name when they’re new.

```ruby
bob = Friend.find_or_initialize_by(first_name: "Bob")

if bob.new_record?
  bob.last_name = "Jones"
end
```

The block allows us to interact with new records beyond setting the attributes to identify the record. Block or not though, this isn’t the end result we want __here__ in this example.

## Update Both Found and New Records

This is the functionality we desire in this particular case. We don't want to find a friend named "Bob Jones". We won't find one. We know Bob's last name changed. But if we do have a friend Bob stored in our database, we want to update their last name.

We can use `find_or_initialize_by` so that we have a friend instance, whether we have one stored or not. From there, we can use what we've learned in the prior sections. We'll avoid passing the method a block - and we'll unconditionally change the returned value to set the last name.

```ruby
bob = Friend.find_or_initialize_by(first_name: "Bob")
bob.last_name = "Jones"
```

With this change, we make sure to commit our friend’s new last name to memory. Whether they existed in our system before or not, their last name is now "Jones".

## Finding The Initial Inspiration

Thanks to Ben Drozdoff for the conversation that led to this post.

Thanks to Matthew Draper for the suggestion to augment this with a solutions-based conclusion, depending on when you expect the code in the block to execute.

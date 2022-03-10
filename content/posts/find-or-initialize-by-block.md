+++
title = "Finding an Initially Confusing Result in Rails"
date = 2022-03-14T07:30:41-05:00
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

When using `find_or_initialize_by` with a block, pay careful attention. The block will only execute in one of those conditions - when initializing an object. That may be confusing to yourself and others reading this code in the future. Avoid the block to prevent misunderstanding the state of your record. Make sure you commit your friend's new last name to memory.

## Finding The Initial Inspiration

Thanks to Ben Drozdoff for the conversation that led to this post.

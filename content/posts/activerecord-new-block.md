+++
title = "ActiveRecord's New Takes a Block, Kid"
date = 2021-05-14T12:20:07-05:00
tags = ["ruby", "rails", "activerecord"]
summary = "Hangin' Tough With All The Ways To Initialize An Object"
+++

## Making New ActiveRecord Models ([Let's Try It Again](https://en.wikipedia.org/wiki/Let%27s_Try_It_Again))

If we want to make a new instance of an ActiveRecord model with particular
attributes, we have a number of options.

We can pass the attributes in as a hash:

```ruby
u = User.new(first_name: "Jordan", last_name: "Knight")
```

We can set the attributes after creating the object:

```ruby
u = User.new
u.first_name = "Jordan"
u.last_name = "Knight"
```

And there's a third option - we can _also_ pass `new` a block:

```ruby
u = User.new do |user|
  user.first_name = "Jordan"
  user.last_name = "Knight"
end
```

## When Could We Use This? ([The Block](https://en.wikipedia.org/wiki/The_Block_(album)))

Let's say we have a system that members of a band use to check their tour
schedule. Band members are users, and when we add a member, we want to make a
user for them.

```ruby
def add_member(first_name:, last_name:)
  @members << User.new(first_name: first_name, last_name: last_name)
end
```

Additionally, users have a username attribute, and we want to keep that unique
within a given band. We also want the system to define the username when we add
a band member.

```ruby
def add_member(first_name:, last_name:)
  username = "#{band_name}_#{last_name}"
  if @members.pluck(:last_name).include?(last_name)
    username << unique_value
  end

  @members << User.new(
    first_name: first_name,
    last_name: last_name,
    username: username,
  )
end

def unique_value
  ...
end
```

However, if we prefer the aesthetic, we can also define those attributes in a
block:

```ruby
def add_member(first_name:, last_name:)
  @members << User.new do |user|
    user.first_name = first_name
    user.last_name = last_name

    user.username = "#{band_name}_#{last_name}"
    if @members.pluck(:last_name).include?(last_name)
      user.username << unique_value
    end
  end
end
```

## Seeing The Result ([The Right Stuff](https://en.wikipedia.org/wiki/You_Got_It_(The_Right_Stuff)))

Let's check our work to see the usernames of our band members.

```ruby
[1] pry(main)> band = Band.new("New Kids on the Block")
[2] pry(main)> band.add_member(first_name: "Jordan", last_name: "Knight")
[3] pry(main)> band.add_member(first_name: "Donnie", last_name: "Wahlberg")
[4] pry(main)> band.add_member(first_name: "Jonathan", last_name: "Knight")
[5] pry(main)> band.members.pluck(:username)
=> ["New_Kids_on_the_Block_Knight",
    "New_Kids_on_the_Block_Wahlberg",
    "New_Kids_on_the_Block_Knight_65"]
```

Jonathan's username has additional characters appended to it, as Jordan already
claimed the username `"New_Kids_on_the_Block_Knight"`.

## Tap Dance ([Step By Step](<https://en.wikipedia.org/wiki/Step_by_Step_(New_Kids_on_the_Block_song)>))

If you're familiar with Ruby's `tap` [method](https://docs.ruby-lang.org/en/master/Kernel.html#method-i-tap),
you might be wondering what all the fuss is about. We can do the same thing
with `tap`:

```ruby
def add_member(first_name:, last_name:)
  @members << User.new.tap do |user|
    user.first_name = first_name
    user.last_name = last_name

    user.username = "#{band_name}_#{last_name}"
    if @members.pluck(:last_name).include?(last_name)
      user.username << unique_value
    end
  end
end
```

This works with any Ruby object, not just those that inherit from
`ActiveRecord::Base`, so why bother with having to know if we can pass a block
to `new` or not, based on what the object inherits from?

That's fair, but `new` is not the only ActiveRecord method that takes a block.
Others include `create`, `build`, and `find_or_initialize_by`. There the
differences with `tap` start to show:

```ruby
new_user = User.create(first_name: "Jordan", last_name: "Knight").tap do |u|
  u.first_name = "Jonathan"
end
```

Our `new_user` has the first name of Jonathan, resulting from the call to `tap`:

```ruby
new_user.first_name
=> "Jonathan"
```

However, that's only persisted in memory - not in the database. What we stored
in the database is what we passed to `create`.

```ruby
new_user.reload.first_name
=> "Jordan"
```

We can also pass a block to `create` directly:

```ruby
new_user = User.create(first_name: "Jordan", last_name: "Knight") do |u|
  u.first_name = "Jonathan"
end
```

And in that case, the first name of the user in memory and in the database is
Jonathan.

```ruby
new_user.first_name
=> "Jonathan"
new_user.reload.first_name
=> "Jonathan"
```

Why would we mix setting attributes with `create` both by passing a hash _and_
a block, either with or without `tap`? Other than to explain quirks and
differences in what method you're passing a block to, I am also interested in
knowing. If you have real-world use cases, [let me know](https://twitter.com/kevin_j_m)!

## Finding Blocks in Rails Source Code ([Face the Music](https://en.wikipedia.org/wiki/Face_the_Music_(New_Kids_on_the_Block_album)))

If you're curious about where in Rails' source code `new` is set up to take a
block, we can start by [looking](https://github.com/rails/rails/blob/70c5496542e5dc82ca28840cb01e710200ce5d14/activerecord/lib/active_record/base.rb#L299)
in `ActiveRecord::Base`. As of the time this article was published, there's not
much implementation in that class. Instead, we have to look in the `Core` [module](https://github.com/rails/rails/blob/70c5496542e5dc82ca28840cb01e710200ce5d14/activerecord/lib/active_record/core.rb#L582)
to find the `initialize` method that takes a block.

Initializing an ActiveRecord model with a block is also defined in the
[documentation](https://github.com/rails/rails/blob/70c5496542e5dc82ca28840cb01e710200ce5d14/activerecord/lib/active_record/base.rb#L35-L40).

Thanks for [hangin' tough](https://en.wikipedia.org/wiki/Hangin%27_Tough) to
the end of this article. I hope you learned a thing or two about passing blocks
to ActiveRecord methods.

> This post originally published on [The Gnar Company blog](https://blog.thegnar.co/activerecord-new-block).

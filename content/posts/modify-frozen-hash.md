+++
title = "Duped into modifying a frozen hash"
date = 2023-02-17T19:00:00-04:00
tags = ["ruby", "rails"]
summary = "I froze in my tracks after seeing this result"
description = "Calling freeze on an object may not provide the immutability you expect. Here we dig into some of freeze's nuances to explore a surprise of my own making I encountered working with ActiveSupport."
+++

## Flash Freeze

You can freeze a hash to prevent modifying its contents. The effect of freezing the hash is only one level deep. The values in the hash aren't frozen. As expected, I can't change the entire value of a key.

```ruby
irb(main):001:0> h = { c: "three" }
=> {:c=>"three"}
irb(main):002:0> frozen = h.freeze
=> {:c=>"three"}
irb(main):003:0> frozen[:c] = "four"
(irb):13:in `<main>': can't modify frozen Hash: {:c=>"three"} (FrozenError)
```

However, I can modify a string value in the frozen hash.

```ruby
irb(main):001:0> h = { c: "three" }
=> {:c=>"three"}
irb(main):002:0> frozen = h.freeze
=> {:c=>"three"}
irb(main):003:0> frozen[:c].upcase!
=> "THREE"
irb(main):004:0> frozen
=> {:c=>"THREE"}
```

I can also modify a hash inside the frozen hash.

```ruby
irb(main):001:0> h = { a: { b: 2 } }
=> {:a=>{:b=>2}}
irb(main):002:0> frozen = h.freeze
=> {:a=>{:b=>2}}
irb(main):003:0> frozen[:a][:b] = 4
=> 4
irb(main):004:0> frozen
=> {:a=>{:b=>4}}
```

And modify an array inside the frozen hash.

```ruby
irb(main):001:0> h = { d: [4, 5, 6] }
=> {:d=>[4, 5, 6]}
irb(main):002:0> frozen = h.freeze
=> {:d=>[4, 5, 6]}
irb(main):003:0> frozen[:d].map!(&:even?)
=> [true, false, true]
irb(main):004:0> frozen
=> {:d=>[true, false, true]}
```

I can modify an instance of an object inside a frozen hash.

```ruby
irb(main):001:0> h = { a: User.new(first_name: "Alice") }
=> {:a=>#<User:0x000000010b50f8c8 @first_name="Alice">}
irb(main):002:0> frozen = h.freeze
=> {:a=>#<User:0x000000010b50f8c8 @first_name="Alice">}
irb(main):003:0> frozen[:a].first_name = "Carol"
=> "Carol"
irb(main):004:0> frozen
=> {:a=>#<User:0x000000010b50f8c8 @first_name="Carol">}
```

The immutability isn't "deep". It's shallow. It doesn't nest down to lower levels. Adding deep freezing to Ruby itself has been [discussed](https://bugs.ruby-lang.org/issues/2509). There are also gems that you can use to recursively freeze objects.

The affect of freeze only being shallow isn't specific to a hash. It applies to other data structures and objects as well. However, I specifically want to talk about hashes because...

## Frozen Rail(s) Shot

Rails includes a `HashWithIndifferentAccess` [class](https://api.rubyonrails.org/classes/ActiveSupport/HashWithIndifferentAccess.html). That class considers the keys `:a` and `"a"` to be the same key. It's part of `ActiveSupport`, and that also includes a [core extension](https://github.com/rails/rails/blob/7-0-stable/activesupport/lib/active_support/core_ext/hash/indifferent_access.rb#L9-L11) to the hash class. That lets you call `.with_indifferent_access` on a hash to transform it into a hash with indifferent access.

```ruby
class Hash
  def with_indifferent_access
    ActiveSupport::HashWithIndifferentAccess.new(self)
  end
end
```

We can use a `HashWithIndifferentAccess` like this.

```ruby
irb(main):001:0> h = { a: 1 }
=> {:a=>1}
irb(main):002:0> h[:a]
=> 1
irb(main):003:0> h["a"]
=> nil
irb(main):004:0> indifferent = h.with_indifferent_access
=> {"a"=>1}
irb(main):005:0> indifferent[:a]
=> 1
irb(main):006:0> indifferent["a"]
=> 1
```

You can also freeze a hash with indifferent access. It's like freezing a hash in the ways described above. It still only applies one level deep.

```ruby
irb(main):001:0> h = { c: "three" }
=> {:c=>"three"}
irb(main):002:0> indifferent = h.with_indifferent_access
=> {"c"=>"three"}
irb(main):003:0> frozen = i.freeze
=> {"c"=>"three"}
irb(main):004:0> frozen[:c] = "four"
can't modify frozen ActiveSupport::HashWithIndifferentAccess: {"c"=>"three"} (FrozenError)
irb(main):005:0> frozen[:c].upcase!
=> "THREE"
irb(main):006:0> frozen
=> {"c"=>"THREE"}
```

## The Cold Never Bothered Me Anyway

`HashWithIndifferentAccess` provides another way you can accidentally allow changes to a frozen hash.

```ruby
irb(main):001:0> frozen = { a: 1 }.freeze
=> {:a=>1}
irb(main):002:0> frozen[:a] = 2
(irb):2:in <main>': can't modify frozen Hash: {:a=>1} (FrozenError)
irb(main):003:0> indifferent = frozen.with_indifferent_access
=> {"a"=>1}
irb(main):004:0> indifferent[:a] = 2
=> 2
irb(main):005:0> indifferent
=> {"a"=>2}
```

Here I created a frozen hash and could not modify it. I then called `with_indifferent_access` on it. The resulting `HashWithIndifferentAccess` *could* change - even with the original hash frozen.

## Brain Freeze

Originally I guessed that maybe `HashWithIndifferentAccess` is using the `dup` [method](https://ruby-doc.org/core-3.1.2/Object.html#method-i-dup). That does not preserve the frozen status of what it's duplicating.

```ruby
irb(main):001:0> frozen = { a: 1 }.freeze
=> {:a=>1}
irb(main):002:0> frozen.frozen?
=> true
irb(main):003:0> duplicated = h.dup
=> {:a=>1}
irb(main):004:0> duplicated.frozen?
=> false
```

As an aside, `clone` *will* preserve the frozen status.

```ruby
irb(main):001:0> frozen = { a: 1 }.freeze
=> {:a=>1}
irb(main):002:0> frozen.frozen?
=> true
irb(main):003:0> cloned = h.clone
=> {:a=>1}
irb(main):004:0> cloned.frozen?
=> true
```

However, my intuition was wrong. `dup` doesn't play a role in the source code. The constructor for `HashWithIndifferentAccess` takes an argument. In our case it's our original frozen hash. It will [pass that argument](https://github.com/rails/rails/blob/7-0-stable/activesupport/lib/active_support/hash_with_indifferent_access.rb#L71) to its `update` method.

That will eventually [iterate](https://github.com/rails/rails/blob/7-0-stable/activesupport/lib/active_support/hash_with_indifferent_access.rb#L412-L417) through each key, value pair in the original hash and write it to the `HashWithIndifferentAccess`:

```ruby
other_hash.to_hash.each_pair do |key, value|
  if block && key?(key)
    value = block.call(convert_key(key), self[key], value)
  end
    regular_writer(convert_key(key), convert_value(value))
  end
end
```

The `regular_writer` method is an [alias](https://github.com/rails/rails/blob/7-0-stable/activesupport/lib/active_support/hash_with_indifferent_access.rb#L87) for `[]=`.

`HashWithIndifferentAccess` is also indifferent about the original hash's frozen status. It constructs a new hash, or hash-like object, setting its keys and values based on the original hash. Whether that original hash is frozen or not doesn't matter. The new hash-like object is not.

```ruby
irb(main):001:0> frozen = { a: 1 }.freeze
=> {:a=>1}
irb(main):002:0> frozen.frozen?
=> true
irb(main):003:0> frozen.with_indifferent_access.frozen?
=> false
```

The frozen status of the *values* do carry over though.

```ruby
irb(main):001:0> frozen = { a: "not changing".freeze }.freeze
=> {:a=>"not changing"}
irb(main):002:0> indifferent = frozen.with_indifferent_access
=> {"a"=>"not changing"}
irb(main):003:0> indifferent[:a].upcase!
(irb):8:in `upcase!': can't modify frozen String: "not changing" (FrozenError)
```

The string in the value of `:a` is still frozen, even after making it a hash with indifferent access.

## Things Got Real Quiet Real Fast (Tenth Avenue Freeze-out)

Calling `freeze` on an object does give you certain (limited) immutability guarantees. Be particularly mindful of how you're interacting with a frozen object. In my case, calling `h.freeze.with_indifferent_access` left me *thinking* I was working with a frozen hash with indifferent access. I was wrong. Flipping the order, and calling `h.with_indifferent_access.freeze` *does* give me what I was expecting.

Stay cool!

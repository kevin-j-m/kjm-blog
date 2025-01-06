+++
title = "What Is It (in Ruby 3.4)?"
date = 2025-01-10T15:40:50-05:00
tags = ["ruby"]
summary = "It's it!"
description = "An explanation of the it block parameter introduced in Ruby 3.4"
+++

This post may appear to be about new functionality in Ruby. However, it may instead be an attempt to write an article with the worst possible SEO.

Anyway, hit it.

{{< rawhtml >}}
<iframe width="560" height="315" src="https://www.youtube.com/embed/mTJ542VTMEE?si=MvYQFv9tqU3n1712" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>
{{< /rawhtml >}}

## What Is It?

As of Ruby 3.4, `it` is another way to refer to the first parameter in a block. This may be more obvious by way of an example.

```ruby
["What is it?"].map { it + " It's it" } * 4

=>
[
  "What is it? It's it",
  "What is it? It's it",
  "What is it? It's it",
  "What is it? It's it",
]
```

The block we pass to `map` does not define a name for the parameter to represent the variable we're manipulating in each iteration. Instead, Ruby knows `it` refers to that variable.

Starting in Ruby 2.7, we had access to numbered parameters in a block with an underscore. We can write that same example using numbered parameters.

```ruby
["What is it?"].map { _1 + " It's it" } * 4

=>
[
  "What is it? It's it",
  "What is it? It's it",
  "What is it? It's it",
  "What is it? It's it",
]
```

Before Ruby 2.7, we would define the name of a variable in a block, which we can still do with more recent versions of Ruby.

```ruby
["What is it?"].map { |i| i + " It's it" } * 4

=>
[
  "What is it? It's it",
  "What is it? It's it",
  "What is it? It's it",
  "What is it? It's it",
]
```

## Handling nested calls

Should you have nested blocks that refer to `it`, Ruby will take `it` to refer to the parameter of the innermost block.

```ruby
[
  "You want it all, but you can't have it",
  "Yeah",
  "It's in your face, but you can't grab it",
  "Yeah",
]
  .map { it == "Yeah" ? [it, it, it].map { it.upcase }.join(", ") : it }

=>
[
  "You want it all, but you can't have it",
  "YEAH, YEAH, YEAH",
  "It's in your face, but you can't grab it",
  "YEAH, YEAH, YEAH",
]
```

The same is not possible with numbered parameters. An exception is raised due to the ambiguity of what `_1` refers to.

```ruby
[
  "You want it all, but you can't have it",
  "Yeah",
  "It's in your face, but you can't grab it",
  "Yeah",
]
  .map { _1 == "Yeah" ? [_1, _1, _1].map { _1.upcase }.join(", ") : _1 }

syntax error found (SyntaxError)
  5 |   "Yeah",
  6 | ]
> 7 | ... _1.upcase }.join(", ") : _1 }
    |     ^~ numbered parameter is already used in outer block
```

## Method names in scope

Ruby also will infer that `it` refers to the block parameter should there be a method called `it` in scope.

```ruby
def it = "This isn't it"

[
  "Can you feel it, see it, hear it today?",
  "If you can't, then it doesn't matter anyway",
]
  .map { it.gsub("it", "IT") }

=>
[
  "Can you feel IT, see IT, hear IT today?",
  "If you can't, then IT doesn't matter anyway",
]
```

## Variable names in scope

However, if there is a __variable__ defined as `it`, that will take precedence over the block parameter in the block.

```ruby
it = "This isn't it"

[
  "You will never understand it, 'cause it happens too fast",
  "And it feels so good, it's like walking on glass",
]
  .map { it.gsub("it", "IT") }

=>
[
  "This isn't IT",
  "This isn't IT",
]
```

## You've got to share it, so you dare it

If you want to learn more about this new feature, check out the [proposal](https://bugs.ruby-lang.org/issues/18980) and the [documentation](https://docs.ruby-lang.org/en/3.4/Proc.html#class-Proc-label-it).

> You can touch it, smell it, taste it, so sweet  
> But it makes no difference 'cause it knocks you off your feet  

That's it.

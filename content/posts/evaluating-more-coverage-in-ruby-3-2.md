+++
title = "Evaluating More Coverage in Ruby 3.2"
date = 2023-01-14T16:04:22-05:00
tags = ["ruby"]
summary = "Covering a new addition to Coverage"
description = "This post demonstrates how Ruby 3.2.0 can now measure coverage of code passed to the eval method."
+++

## Measuring Coverage of Eval

As I mentioned in my [prior post]({{< ref "my-first-code-commit-in-ruby" >}}), Ruby 3.2.0 has some changes to the `Coverage` module. Now the module can measure the coverage of a Ruby expression in a string passed to the [eval](https://ruby-doc.org/3.2.0/Kernel.html#method-i-eval) method.

This is important because of templates. ERB, when we ask for the template through the [result method](https://docs.ruby-lang.org/en/master/ERB.html#method-i-result), calls `eval`. When Rails is rendering a view, that [also calls](https://github.com/rails/rails/blob/2f36f0a2bb9d6244035f37e62c978ac11ef88411/actionview/lib/action_view/template.rb#L368-L372) `eval`. More specifically, Rails calls the `module_eval` [method](https://ruby-doc.org/3.2.0/Module.html#method-i-module_eval).

Have you wondered how much of the logic in your views is exercised in your test suite? Thanks to this change, now you can see that in tools like [SimpleCov](https://github.com/simplecov-ruby/simplecov/pull/1037).

## Feature Introduction

Let's walk through an example demonstrating this functionality.

```ruby
require "coverage"
Coverage.start(eval: true, lines: true)
eval("1 > 2 ? 'not reached' : 'covered'", nil, "filename.rb", 1)
Coverage.result
=> {"filename.rb"=>{:lines=>[1]}, "(irb)"=>{:lines=>[nil, nil, 1, 1]}}
```

We need to require the Coverage module first. After that, we ask coverage to start measuring with the `start` method. Here we explicitly ask it to measure `eval`. We're using lines coverage to answer how many times each line is run.

We call `eval`, passing a string with a ternary statement. We also pass in the optional filename and line number parameters as well. We check our measurement with the `result` method.

The keys of that hash are file names, or places, where Ruby measures coverage. Because we passed in the filename parameter in our `eval` call, the `eval` coverage has a key of the filename passed to `eval`.

Lines coverage provides an array of numbers. Each number tells us how many times each line was executed. The first item in the array, at index 0, is how many times the first line was executed. Here we see our first line of our single-line `eval` statement was executed once, as we'd expect.

## Opting in to eval

In Ruby 3.2.0, measuring coverage of `eval` statements is optional. By default,
coverage will *not* measure `eval` coverage. You must explicitly tell it to by
passing `eval: true` to `Coverage.start`. Notice how we have no coverage results
without passing `eval: true`. That's because otherwise, Coverage is looking to
measure loaded files.

```ruby
require "coverage"
Coverage.start(lines: true)
eval("1 > 2 ? 'not reached' : 'covered'", nil, "filename.rb", 1)
Coverage.result
=> {}
```

Using the `:all` option will also measure the coverage of `eval`.

```ruby
require "coverage"
irb(main):002:0> Coverage.start(:all)
irb(main):003:0> eval("1 > 2 ? 'not reached' : 'covered'", nil, "filename.rb", 1)
irb(main):004:0> Coverage.result
=>
{"(irb)"=>{:lines=>[nil, nil, 1], :branches=>{}, :methods=>{}},
 "filename.rb"=>
  {:lines=>[1], :branches=>{[:if, 0, 1, 0, 1, 33]=>{[:then, 1, 1, 8, 1, 21]=>0, [:else, 2, 1, 24, 1, 33]=>1}}, :methods=>{}}}
```

## Setting the Mode

I have unintentionally demonstrated this when showing the `:all` option, but you
can measure coverage of `eval` statements with the [different modes available]({{< ref "rubys-got-you-covered#coverage-modes" >}}) in Coverage.

Our `eval` statement has two code paths on a single line because it's a ternary.
Let's measure the branches coverage of our statement.

```ruby
require "coverage"
Coverage.start(eval: true, branches: true)
eval("1 > 2 ? 'not reached' : 'covered'", nil, "filename.rb", 1)
Coverage.result
=> {"filename.rb"=>{:branches=>{[:if, 0, 1, 0, 1, 33]=>{[:then, 1, 1, 8, 1, 21]=>0, [:else, 2, 1, 24, 1, 33]=>1}}}, "(irb)"=>{:branches=>{}}}
```

Here we see that we've executed the `else` statement of our if test, but not the other side (the `then`) of our conditional. A more in-depth explanation of this output is available [here]({{< ref "rubys-got-you-covered#branches-coverage" >}}).

## Mode Required

*Generally*, Coverage will start in lines mode when provided no options in `start`. However, I've noticed that when you ask Coverage to start and measure eval coverage, you __must__ also specify the mode(s) you want to measure.

As you can see, starting Coverage with eval on and no mode gives us no coverage
results.

```ruby
require "coverage"
Coverage.start(eval: true)
eval("1 > 2 ? 'not reached' : 'covered'", nil, "filename.rb", 1)
Coverage.result
=> {"filename.rb"=>{}, "(irb)"=>{}}
```

## Credit

Thank you to [Samuel Williams](https://github.com/ioquatix) for introducing this functionality into the Ruby
codebase. Thank you to [Yusuke Endoh](https://github.com/mame) for adding it to SimpleCov, and also for
writing and maintaining *most* of the Coverage functionality available in Ruby's
standard library.

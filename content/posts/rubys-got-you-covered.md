+++
title = "Ruby's Got You Covered"
date = 2021-10-21T20:00:07-04:00
tags = ["ruby"]
summary = "Explaining Ruby's Coverage Module"
description = "Ruby's coverage module includes many options that can answer different questions about your code. This article explores the modes that the module provides."
+++

## Coverage

Perhaps you've heard of [test coverage](https://en.wikipedia.org/wiki/Code_coverage),
which is a measurement of how much of your application code is executed when
your tests run. That number is typically represented as a percentage, and people
may use that metric to assess the relative health of a codebase. The efficacy of
such metrics is [debated]({{< ref "/remote-ruby-143" >}}), but the metrics are
still prevalent.

This article will demonstrate the mechanism ruby provides to measure coverage
and present some examples for how to use it and is a summary of the information
I shared about coverage at [RubyConf 2020](https://youtu.be/EyLO0EEm3BQ).

{{< rawhtml class="float" >}}
<iframe width="560" height="315" src="https://www.youtube.com/embed/EyLO0EEm3BQ" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
{{< /rawhtml >}}
## Running Coverage

Ruby ships with a [Coverage module](https://docs.ruby-lang.org/en/3.0.0/Coverage.html) as part of the language.
To use it, you must first `require` the module.

After doing that, you have access to `Coverage`. Coverage begins running when
you call the `start` method. It then expects the file you want coverage to be
assessed on to be `require`d or `load`ed. Finally, you can see coverage's output
by calling `result`.

For example, let's say we're all in a band and we're practicing a new
[cover song](https://youtu.be/_7Qk9MdyiOM).

```ruby
# rehearsal.rb
our_band = Band.new(name: "Blogger Band")

song = CoverMe.new
song.original_artist = "Bruce Springsteen"
song.band = our_band

song.play
```

In order to run coverage on this file, we can do the following:

```ruby
require "coverage"

Coverage.start
load "rehearsal.rb"
Coverage.result
```

## Coverage Modes

Ruby's coverage module has many modes, or different ways of assessing coverage.
Each mode answers a different question about the code that was run under coverage:

* Lines - how many times was each line executed?
* Oneshot Lines - which lines were executed?
* Methods - how many times was each method executed?
* Branches - how many times was each conditional executed?

You can specify which modes to run by passing an argument to `Coverage.start`.

### Lines Coverage

```ruby
Coverage.start(lines: true)
```

This is the mode that runs if you do not pass any arguments to `Coverage.start`.
Each relevant line has a counter that is incremented each time the line is
visited in code execution while coverage is running. Irrelevant lines, those
that are things like empty lines or `end` statements, are ignored. At the
conclusion, you will see how many times each line is executed.

Our guitarist wants to track how often they break a string during rehearsal. A
string is broken when the `@broken` instance variable is set.

```ruby
class String
  def break_string
    @broken = true
    BrokenStringSound.new
  end
end
```

Coverage's result provides a hash, where the keys are all the files that were
run while coverage was running. Each value is a hash that has a key for the
mode(s) of coverage run.

For lines coverage, the value of that inner hash is an array showing how many
times each line was executed. The integer at index 0 of this array shows how
many times line 1 was run.

```ruby
{
  ...
  "string.rb" => {:lines => [1, 1, 4, 4, nil,...]},
  ...
}
```

The `nil` represents an irrelevant line, in this case, an `end` statement. To
answer our question, we need to see how many times line 3 of the string file was
run, which is index 2 in the array - and we see our guitarist broke 4 strings in
one rehearsal.

### Oneshot Lines Coverage

```ruby
Coverage.start(oneshot_lines: true)
```

Similar to lines coverage, this also documents that a relevant line was executed while coverage was running. However, it’s a binary report of whether it was executed or not. It will not tell you how often. This may be sufficient in many cases, and comes with the benefit of being more performant every subsequent time a particular line of code is executed under coverage.

The drummer has a break in the song where they play a small fill.

```ruby
class Drum
  def small_fill
    bang_tom
    roll_snare(duration: 2)

    if extend_fill?
      hit_crash_cymbal
    end

    strike_ride_cymbal
  end
end
```

The band isn't sure if the drummer is hitting the crash cymbal during the fill.
To find out, they can use oneshot lines coverage, which will tell if the line of
code is executed. They don't care how many times; only if it ever happened.

The result looks similar to lines coverage:

```ruby
{
  ...
  "drum.rb"=>{:oneshot_lines=>[1, 2, 7, 3, ...]},
  ...
}
```

The values in the array are different from lines coverage though. Here, each
integer in the array is a line number that was executed. Remember, oneshot lines
won't tell you how many times a line was run. The order of elements does not
matter, unlike lines coverage.

In the case of our drum fill, 7 is in the array, which is the line number to hit
the crash cymbal, so the drummer is extending the fill.

### Methods Coverage

```ruby
Coverage.start(methods: true)
```

Methods coverage brings the granularity of lines coverage up to a coarser grain. Rather than tracking individual lines, it’s concerned with whether a particular method is executed. It can be a 10 line method where the first line is the only line ever executed. Methods coverage will still consider that as executed the same as a 20 line method where each line is executed.

Now that our guitarist knows they break a lot of strings, they need to thin out
the gear they bring to gigs so they have more room in their bag for strings.
They're wondering which effects pedals they're even using on their pedal board.
They have a lot, and each of them responds to `trigger`, which turns them off
or on when you press them.

```ruby
class ReverbPedal
  def trigger
    ...
  end
end

class OverdrivePedal
  def trigger
    ...
  end
end

class DelayPedal
  def trigger
    ...
  end
end
```

We can use methods coverage to see which of those pedals are being triggered
during rehearsal.

```ruby
{
  ...
  "reverb_pedal.rb"=>
    {:methods=>{[ReverbPedal, :trigger, 2, 2, 4, 5]=>2}},
  "overdrive_pedal.rb"=>
    {:methods=>{[OverdrivePedal, :trigger, 2, 2, 4, 5]=>0}},
  "delay_pedal.rb"=>
    {:methods=>{[DelayPedal, :trigger, 2, 2, 4, 5]=>3}},
  ...
}
```

Unlike the results we've seen thus far, this isn't only returning an array in
the value of the mode hash. Instead, there's another hash where the key
identifies the method, and the value is the number of times the method is
executed. Let's dig into what each of the elements identifying a method are.

```ruby
[OverdrivePedal, :trigger,  2,  2,  4,  5]
#       ^            ^      ^   ^   ^   ^
#       |            |      |   |   |   |
#     Class          |      |   |   |   |
#     Name           |      |   |   |   |
#                  Method   |   |   |   |
#                  Name     |   |   |   |
#                           |   |   |   |
#                         Start |   |   |
#                         Line  |   |   |
#                               |   |   |
#                               |   |   |
#                             Start |   |
#                             Column|   |
#                                   |   |
#                                   |   |
#                                  End  |
#                                  Line |
#                                       |
#                                       |
#                                      End
#                                      Column
```

To help our guitarist clean up their pedal board, we can see that the overdrive
pedal isn't used at all, and can be left at home next time.

### Branches Coverage

```ruby
Coverage.start(branches: true)
```

Branches Coverage tracks execution of different conditional paths and documents how often those different paths are run. The unique benefit that this provides over lines coverage is in conditionals that execute multiple code paths in a single line, such as ternary statements. You may have a part of that conditional that’s never run or tested, but you would not know that if you’re relying on lines coverage alone.

Our singer wants to use an echo effect during the song, and has a friend setting
the intensity as they practice.

```ruby
class CoverMe
  def chorus(number)
    echo_intensity = number.positive? && number.even? ? 10 : 30

    Lyric.new(line: line, effect: :echo, effect_level: echo_intensity)
  end
end
```

During one run-through of the song, they're happy with the effect and want to
check how often they used each intensity. Because this is expressed as a
ternary, we __can't__ use lines coverage. We could use it if the method were
structured like this:

```ruby
def chorus(number)
  echo_intensity = if number.positive? && number.even?
    10
  else
    30
  end
  ...
end
```

However, in either case, we *can* use branches coverage to see which of the
different branches were followed.

The output of branches coverage looks similar to that of methods coverage.

```ruby
{
  "cover_me.rb" => {
    :branches => {
      {
        [:if, 0, 34, 25, 34, 67] => {
          [:then, 1, 34, 60, 34, 62] => 0,
          [:else, 2, 34, 65, 34, 67] => 2,
        }
      }
    }
  }
}
```

The differences from methods coverage are:

1. Branches coverage nests each branch within its conditional, so the data
   structure is nested one level deeper than methods coverage.
2. Branches coverage assigns a unique identifier to each conditional or branch.


Let's look at what each of the elements identifying a branch are.

```ruby

[:then,  1,  34,  60,  34,  62]
#   ^    ^   ^    ^    ^    ^
#   |    |   |    |    |    |
# Branch |   |    |    |    |
#        |   |    |    |    |
#        Id  |    |    |    |
#            |    |    |    |
#            |    |    |    |
#          Start  |    |    |
#          Line   |    |    |
#                 |    |    |
#                 |    |    |
#               Start  |    |
#               Column |    |
#                      |    |
#                      |    |
#                     End   |
#                     Line  |
#                           |
#                           |
#                          End
#                          Column
```

Looking at the results, the satisfactory performance had the echo intensity
cranked up the 30 the entire time. The `else` condition of the ternary was the
only branch executed. Now the band knows how to set the effect for their next
performance.

### All Coverage Modes

```ruby
Coverage.start(:all)
```

Passing the `:all` symbol to `Coverage.start` will ask it to run every coverage
mode; however, if you inspect the output, you'll notice that one is missing.

```ruby
require "coverage"

Coverage.start(:all)
load "rehearsal.rb"
result = Coverage.result

result["guitar.rb"].keys
=> [:lines, :methods, :branches]
```

Oneshot lines is missing!

Oneshot lines and lines modes [cannot be run at the same time](https://github.com/ruby/ruby/blob/d92f09a5eea009fa28cd046e9d0eb698e3d94c5c/ext/coverage/coverage.c#L53),
so lines coverage
is run, as you can still use it to answer if a line was executed at all.

## Coverage In Practice

It may be unlikely that you use the `Coverage` module directly. However, there
are tools you can use to measure code coverage that builds on this abstraction.

There are many tools for measuring test coverage, but one is [SimpleCov](https://github.com/simplecov-ruby/simplecov). It also
supports [branches coverage](https://github.com/simplecov-ruby/simplecov#branch-coverage-ruby--25). To measure coverage of production code, check out
[Coverband](https://github.com/danmayer/coverband), which you can set up to use [oneshot lines](https://github.com/danmayer/coverband/blob/43c5ac94febc7a961346b0e9408d829d4d2ef8ad/test/rails5_dummy/config/coverband.rb#L15) mode.

Ruby's coverage module includes many options that can answer different questions
about your code. What do you think you could use it for in your application?
[Let me know](https://twitter.com/kevin_j_m)!

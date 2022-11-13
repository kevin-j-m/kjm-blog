+++
title = "Flog-Driven Development"
date = 2023-05-13T07:20:24-04:00
tags = ["ruby", "software-design-concert-series", "anyone-can-play-guitar"]
summary = "Refactoring for a better score"
description = "This post explores how flog, a tool that analyzes the complexity of your code, can identify areas to refactor."
+++

## Anyone Can Play Guitar Series

1. [Enumerating Musical Notes]({{< ref "/enumerating-musical-notes" >}})
2. [Revisiting Calling Sonic Pi From Ruby]({{< ref "/revisiting-calling-sonic-pi-from-ruby" >}})
3. [Programming Guitar Greatness]({{< ref "/programming-guitar-greatness" >}})
4. [Composing Our Own Guitar Amps From Inherited Gear]({{< ref "/composing-our-own-guitar-amps-from-inherited-gear" >}})
5. __Flog-Driven Development__

## Remembering Refactoring

In a [prior post]({{< ref "/programming-guitar-greatness" >}}), we extracted the details of how to tune a guitar out of the `Guitar` class. We [moved it]({{< ref "/programming-guitar-greatness#tune-low" >}}) to a separate `Tuner` class. In that telling, we did so because it spoke to our sensibilities. It afforded us more space in the `Guitar` class to focus on other responsibilities. It gave a central location to focus on tuning.

What if those justifications weren't enough? What if we needed metrics to give an explanation for our refactoring? Let's use the guidance from a tool called [flog](https://github.com/seattlerb/flog) to guide our changes.

## Flog

Flog [describes itself](https://github.com/seattlerb/flog#label-DESCRIPTION-3A) as a tool that:

> reports the most tortured code in an easy to read pain report. The higher the score, the more pain the code is in.

You point flog to a file, or directory, and it provides you with a score. The higher the score, the more attention you might want to pay to it. As for how flog calculates the number, I'll let flog [summarize itself](https://ruby.sadi.st/Flog.html) again:

> Flog essentially scores an ABC metric: Assignments, Branches, Calls, with particular attention placed on calls.

Let's look at the flog score of our `Guitar` class with all the tuning details inside the class:

```bash
⇒ flog guitar_with_tuning.rb -a
65.6: flog total
9.4: flog/method average

16.8: Guitar#standard_tuning
16.8: Guitar#down_half_step_tuning
13.0: Guitar#tune
6.6: Guitar#pick
5.2: Guitar#initialize
4.3: Guitar#restring
3.0: Guitar#none
```

Let's investigate the method with the biggest flog score.

```ruby
def standard_tuning
  @strings[5].tune(note: :e, octave: 2)
  @strings[4].tune(note: :a, octave: 2)
  @strings[3].tune(note: :d, octave: 3)
  @strings[2].tune(note: :g, octave: 3)
  @strings[1].tune(note: :b, octave: 3)
  @strings[0].tune(note: :e, octave: 4)
end
```

We know that flog is particularly attuned to looking for calls. Each line in this method has a call to access an element in an array, and then another call to the `tune` method. That's two calls per line, with six lines in the method.

## What do we do with the number?

So, bigger is worse, but how big is bad? At what number should you take action? Thoughtbot's [Ruby Science](https://github.com/thoughtbot/ruby-science) book suggests a method is long or complex with a flog score above 10. It also posits that a class is long or complex with a flog score above 50.

With those heuristics in mind, our `Guitar` class is long, and all the methods related to tuning are long as well. Let's start by looking at the public method in that class, the method to tune the guitar:

```ruby
def tune(tuning = :standard)
  case tuning
  when :standard
    standard_tuning
  when :down_half_step
    down_half_step_tuning
  when :drop_d
    drop_d_tuning
  else
    raise "unknown tuning"
  end

  @tuning = tuning
end
```

The technique I'd start with to simplify a complex method is to extract smaller methods from it. However, there doesn't seem to be much gained from that here. This happens to be a really long case statement, each branch of which calls another method.

In an attempt to achieve a lower flog score, I'm going to rely on the similar naming of each of these tuning methods. I'll reach for a different tool: metaprogramming.

```ruby
def tune(tuning = :standard)
  raise "unknown tuning" unless VALID_TUNINGS.include?(tuning)

  send("#{tuning}_tuning")

  @tuning = tuning
end
```

What does flog think about this change?

```bash
⇒ flog guitar_tuning_metaprogramming.rb
58.1: flog total
8.3: flog/method average

5.6: Guitar#tune
```

The flog score decreased from 13 to 5.6. A clear improvement - in the eyes of the metric. Is it better though? I'd argue that's still a matter of taste. How do you feel about the metaprogramming? How comfortable will you and your team be maintaining this? What pressure does the required naming scheme for any of the tuning methods place on your system? Consider these questions to decide whether to make this change, regardless of the flog score.

## Extract Class

Even with this change, we still have a class that has a flog score over 50. That puts it in the "too long" category, according to Ruby Science. So, let's do what we did in the prior post and move everything related to tuning into a `Tuner` class.

```ruby
class Tuner
  def initialize(guitar)
  def tune(tuning = :standard); end
  def standard_tuning; end
  def down_half_step_tuning; end
  def drop_d_tuning; end
  def open_a_tuning; end
  def modal_c_tuning; end
  def all_fourths_tuning; end
  def all_fifths_tuning; end
end
```

We still need to tune our guitar. We achieve that by delegating this responsibility to our `Tuner` class.

```ruby
class Guitar
  def tune(tuning = :standard)
    Tuner.new(self).tune(tuning)
  end
end
```

What does that mean for our flog score?

```bash
⇒ flog guitar_separate_tuner.rb
18.9: flog total
3.8: flog/method average
```

Unsurprisingly, deleting most of the code in a class reduces the complexity of that class. We didn't get rid of the complexity though. We just moved it somewhere else. Let's check the flog score of our new `Tuner` class.

```bash
⇒ flog tuner.rb
58.6: flog total
11.7: flog/method average

25.2: Tuner#standard_tuning
25.2: Tuner#down_half_step_tuning
5.3: Tuner#tune
```

The complexity of our tuning methods actually got *worse*, increasing from 16.8 to 25.2. That's because we added another call on each of our lines.

Tuning one string used to look like this:

```ruby
@strings[5].tune(note: :e, octave: 2)
```

But now we need to find the string from the guitar that's passed into the tuner.

```ruby
@guitar.strings[5].tune(note: :e, octave: 2)
```

## Extracting new concepts

To handle the complexity, we're going to reshuffle the responsibility of these ideas. Right now the methods for the different tunings tune each string of the guitar. Instead, let's have the tunings only know what notes to tune the strings to.

We'll also extract each of these tunings out to a separate module.

```ruby
module StandardTuning
  def self.pitches
    [
      { note: :e, octave: 4 },
      { note: :b, octave: 3 },
      { note: :g, octave: 3 },
      { note: :d, octave: 3 },
      { note: :a, octave: 2 },
      { note: :e, octave: 2 },
    ]
  end
end
```

Now when we want to add a new tuning, we add a new module, rather than another method on the `Tuner` class.

The act of tuning is now isolated into the `Tuner#tune` method.

```ruby
class Tuner
  def tune(tuning = :standard)
    raise "unknown tuning" unless VALID_TUNINGS.include?(tuning)

    pitches_for(tuning).each_with_index do |pitch, index|
      @guitar.strings[index].tune(**pitch)
    end
  end

  private

  def pitches_for(tuning)
    Object.const_get(tuning_class_name(tuning)).pitches
  end

  def tuning_class_name(tuning)
    "#{tuning}_tuning".split("_").map(&:capitalize).join
  end
end
```

Keeping with our original refactor, this still uses metaprogramming. This time we use it to recall the appropriate module to get the pitches from.

What does flog think about this?

```bash
⇒ flog tuner_without_tunings.rb
21.9: flog total
4.4: flog/method average

8.5: Tuner#tune
6.8: Tuner#tuning_class_name
3.6: Tuner#pitches_for
```

Our `Tuner` class has a flog score under 50, and all the methods are under 10. But, like last time, did we just shift the complexity into our extracted tuning modules?

```bash
⇒ flog standard_tuning.rb -a
1.5: flog total
1.5: flog/method average

1.5: StandardTuning::pitches

⇒ flog down_half_step_tuning.rb -a
1.5: flog total
1.5: flog/method average

1.5: DownHalfStepTuning::pitches
```

Each of these is lightweight - a single method returning an array of hashes. According to Ruby Science's heuristics, we no longer have complex methods or classes!

## Flog as a forcing function

At the time I didn't refer to flog when making these changes. But it was interesting to reconstruct them with a dedicated focus on their flog scores. When you don't have any intuition around the complexity of a piece of code, it can help as an objective source.

Choosing to lean on some of these changes would require more than only the score. Introducing the metaprogramming would depend on my team's attitude towards that language feature. 

Those high flog numbers may have driven me to explore different options. I may not have gotten to the point of building separate modules for the tunings. That freed any given class to know about *every* different tuning. Paying attention to the flog score pushed me in ways I may not have otherwise considered.

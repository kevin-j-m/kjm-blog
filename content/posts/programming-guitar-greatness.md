+++
title = "Programming Guitar Greatness"
date = 2023-03-18T07:00:10-04:00
tags = ["ruby", "software-design", "anyone-can-play-guitar"]
summary = "A Texas Flood of Domain Modeling"
description = "This post explores different domain modeling tools and object-oriented development to teach a computer how to play the guitar like Stevie Ray Vaughan."
+++

## Anyone Can Play Guitar Series

1. [Enumerating Musical Notes]({{< ref "/enumerating-musical-notes" >}})
2. [Revisiting Calling Sonic Pi From Ruby]({{< ref "/revisiting-calling-sonic-pi-from-ruby" >}})
3. __Programming Guitar Greatness__

> I use heavy strings, tune low, play hard, and floor it. Floor it. That's technical talk.
> -- [Stevie Ray Vaughan](https://twitter.com/srvofficial/status/836589348489424896?lang=en)

Stevie Ray Vaughan is one of my favorite guitarists. Unfortunately, I can't play anything like he can. To make up for it, let's teach a computer to play guitar like him and see what we can learn.

Hit play for some background music or inspiration, and let's get started.

{{< rawhtml >}}
<iframe width="560" height="315" src="https://www.youtube.com/embed/KC5H9P4F5Uk" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
{{< /rawhtml >}}

## Use Heavy Strings

Guitars have many strings (typically six) that you manipulate to make different sounds. To build a system to play guitar, it needs to know about strings. To avoid any [potential confusion](https://ruby-doc.org/3.1.2/String.html), we'll build a `GuitarString` class.

Different strings on a guitar are different thicknesses. Thicker strings play notes at a lower frequency than thinner strings. We measure this string thickness in thousands of an inch.

A set of standard strings is a set of nines. The thinnest string in the set is 0.009 inches thick. Stevie Ray Vaughan played a set of 13s that were even thicker on the low end than a stock set of 13s.

These strings are hard to bend, hard to move, and hard to play with. Stevie called them "heavy". We'll know if a string is heavy by comparing it to a standard set.

```ruby
class GuitarString
  def heavy?
    gauge_number > common_gauge_number
  end
end
```

### Use Domain Terms

We taught our computer about guitar strings using the language of a guitarist. I would typically refer to a wire as having a particular thickness. Stevie referred to his strings as being "heavy". To represent his description, we'll use words that resonate in the world our system models. Our guitar strings are "heavy", not "thick".

## Tune Low

We tune each string on a guitar to a note. The most common combination of string tunings is standard tuning. Stevie didn't play in standard tuning. He tuned down a half step. Each string is tuned to a slightly lower pitch than you'd regularly expect.

To support this, we need to be able to to tune the guitar. We need to be able to tune the guitar to standard tuning, and down a half step. We'll accept which tuning to use for our guitar as an argument to a `tune` method. We'll switch on that argument to implement that tuning.

```ruby
class Guitar
  def tune(tuning = :standard)
    case tuning
    when :standard
      standard_tuning
    when :down_half_step
      down_half_step_tuning
    end
  end
end
```

These aren't the only tunings possible for a guitar. There are a __lot__ of them. Supporting more will mean this method gets more complex. Our guitar class gets more complex.

```ruby
class Guitar
  def tune(tuning = :standard); end
  def standard_tuning; end
  def down_half_step_tuning; end
  def drop_d_tuning; end
  def open_a_tuning; end
  def modal_c_tuning; end
  ...
end
```

As we handle more and more tunings, our guitar grows in complexity. It starts to look like an instrument primarily responsible for tuning itself. The number of methods that have to do with tuning takes away from the more exciting things you can do with a guitar.

To focus on a guitar's other responsibilities, let's extract this logic. A `Tuner` class will take a guitar as a dependency and know how to tune it to a variety of tunings. That changes the implementation in our `Guitar` class to look like this:

```ruby
class Guitar
  def tune(tuning = :standard)
    Tuner.new(self).tune(tuning)
  end
end
```

All the complexity of different guitar tunings is still in our system. It's in the `Tuner` class now, not `Guitar`.

### Extract Related Behavior

In a system responsible for playing the guitar, that class attracts a lot of behavior. That can make understanding all the responsibilities that the class has difficult. When we can identify a set of related behaviors, we should explore moving it into a separate class.

An early indicator of this can be when related methods are physically grouped together in a class. Especially when those methods don't have anything to do with other parts of a class. Our guitar had a lot of different methods for each of the different tunings. We moved those out of the class and into another that's responsible for all possible tunings.

Even if this class isn't reused or composed in other classes, there's value here. We've freed up complexity inside the `Guitar` class, while not taking away any of its ability. We can still tune the guitar to any number of tunings. We don't need to worry ourselves with the implementation details of *how* most of the time. And when we do need to dig into how a guitar gets tuned, we know where to look. We don't need to dig into the depths of various private methods in `Guitar`. We can start with our aptly-named `Tuner` class.

## Play Hard

We make sounds on our guitar by plucking the strings with one hand. The other hand presses down on the strings on the neck of the guitar. The neck has many sections called frets. Pressing down on each of these plays a higher frequency as you move up the neck towards your other hand.

Our `GuitarString` knows which note we play, based on what note it's tuned to and which fret our hand is on. But we don't say we play the guitar strings - we play the guitar.

```ruby
class Guitar
  def pick(string:, fret:)
    @strings[string -1].pluck(fret: fret)
  end
end
```

With this implementation, we get back the note of the sound the guitar makes.

```ruby
guitar = Guitar.new
guitar.tune
guitar.pick(string: 6, fret: 1)
=> :f
```

### Compose Collaborators

Much like with our tuning, our public interface is through the guitar. Again, the majority of our work isn't done by the `Guitar` class. Its responsibility is taking the input and passing it off to a collaborating class. Here, it figures out which of the strings is being played, and sends it the `pluck` method.

The `GuitarString` handles the hard work of which musical note comes out of the guitar. The `Guitar` knows how to work with its strings to achieve the result that the caller asked for.

## Floor It

It's not enough to know which note we're playing. It's a start, but doesn't tell the full story about how what we play sounds. We also need to know which *octave* of the note we're playing.

We'll change our `GuitarString` class to return not only the note, but also the octave. We return both elements in an [array](https://ruby-doc.org/3.1.2/Array.html).

```ruby
class GuitarString
  def pluck(fret:)
    [note, octave]
  end
end
```

Now let's display all this data.

```ruby
note = guitar.pick(hand_position)
"#{note.first}#{note.last}"
```

We know that the first element is the note, and the last element is the octave. We know that because we just wrote it, and it's sitting right above our use of it. However, it's not obvious in other cases what each of these elements refers to.

We'll address that by building a [custom Note class]({{< ref "enumerating-musical-notes" >}}) and returning it in `GuitarString#pluck`.

```ruby
class GuitarString
  def pluck(fret:)
    Note.new(
      starting_note: @tuning_note,
      starting_octave: @tuning_octave,
      offset: fret,
    )
  end
end
```

That will change our display logic.

```ruby
note = guitar.pick(hand_position)
"#{note.value}#{note.octave}"
```
Now it's clear what the data is that we're displaying. It's not the first element, it's the note value. It's not the last element, it's the note octave.

We could achieve a similar result by using a [Hash](https://ruby-doc.org/3.1.2/Hash.html). That will allow us to name the data elements we refer to in our display logic.

By moving to a separate class, we can also associate behavior with this data.

```ruby
class Note
  def value; end
  def octave; end

  def to_s
    "#{value[0].upcase}#{'b' if flat?}#{octave}"
  end
end
```

Now our display logic doesn't even need to know about the internals of the `Note` class. Instead, it just needs to ask it to display itself.

```ruby
guitar.pick(hand_position).to_s
```

### Elevate Primitives to Objects

We needed to return a collection of information out of our `GuitarString#pluck` method. Callers need both the note and octave to know what our output sounds like. We started with a primitive data structure from Ruby, an Array.

That worked, but wasn't very clear what all the data elements represented. We can bring clarity to that with another primitive, a Hash. Later on, we wanted to exercise custom behavior on top of this collection of data. To do that, we made a separate class that encapsulates this data __and__ related behavior.

## That's Technical Talk

We now have a system that knows how to play guitar *just like* Stevie Ray Vaughan did. Except for all the talent, the feeling, the creativity, and the humanity that went into his playing.

Along the way, we reinforced concepts by using domain terminology. We identified related behavior within a class and extracted it to a separate class. We collaborated with those extractions to build up our system. Our public interfaces (like the `Guitar#pick` method) don't need to house the complexity. And we built more classes to replace primitive data structures. When we identified behavior related to that data, we had a natural landing place for it.

I hope this Texas-sized flood of information helps in your next domain modeling exercise.

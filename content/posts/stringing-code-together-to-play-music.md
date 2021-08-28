+++
title = "Stringing Code Together to Play Music"
date = 2020-12-15T07:03:46-05:00
tags = ["ruby", "software-design", "software-design-concert-series"]
summary = "Anyone Can Play Guitar"
description = "In a previous post, we built an amplifier. Now, we'll learn to play the guitar with ruby code."
+++

## Ruby Software Design Concert Series

1. [Dependency Injection: Plug In]({{< ref "/dependency-injection-plug-in" >}})
2. [Shedding a Light on Duck Typing]({{< ref "/shedding-light-on-duck-typing" >}})
3. [Synthesizing Composition With Delegation]({{< ref "/synthesizing-composition-with-delegation" >}})
4. [Inheritance: Derivative Songwriting]({{< ref "/inheritance-derivative-songwriting" >}})
5. [Using Sonic Pi To Play Music With Ruby]({{< ref "/using-sonic-pi-to-play-music-with-ruby" >}})
6. __Stringing Code Together To Play Music__

## Setting the Stage

In our [last post]({{< ref "/using-sonic-pi-to-play-music-with-ruby" >}}), I talked about how I built an interface to [Sonic Pi](https://sonic-pi.net) when
I was preparing my [RubyConf 2020](https://youtu.be/EyLO0EEm3BQ) talk about Ruby's [Coverage](https://docs.ruby-lang.org/en/master/Coverage.html) module. At the
end of that post, we could send sounds to Sonic Pi. Today, we'll have our code
play the guitar, and send those sounds to our amplifier.

## String Theory

A guitar is a string instrument, and each of those strings make a sound when you
play them. For this example we'll focus on the happy path, which is that
plucking the string plays the expected note. The code I built also considers
that strings can break, and attempting to play broken strings won't work. You
can look at the [full implementation](https://github.com/kevin-j-m/ruby_cover_band/blob/09e7b72b38dac09d4968afe1468eda53caaf294c/lib/ruby_cover_band/instruments/guitar/string.rb#L20-L28)
to see how that works.

Plucking an individual string creates a new sound.

```ruby
class String
  def pluck(fret:)
    ...
    play_note(fret)
  end

  private

  def play_note(fret)
    StringSound.new(
      string_number: @number,
      tuning_note: tuning_note,
      fret_number: fret,
    )
  end
end
```

The `@number` variable is which string on the guitar it is, with index 0 being
the low E, and index 5 being the high E, in standard tuning. The `tuning_note`
is what note that string is tuned to, because any string __can__ be tuned to any
note. Again, for simplicity here, we'll assume standard tuning (EADGBE).

Our `StringSound` class converts that information into the command we'll send to
Sonic Pi. All notes in Sonic Pi are represented with a [number](https://sonic-pi.net/tutorial#section-2-1),
and we can also use "traditional" note names, passed to it as a symbol. We can
use that to figure out the note our string would play if you plucked it without
pressing down on a fret.

```ruby
class StringSound
  def playable_note_root
    playable_note_key.dig(@string_number, @tuning_note)
  end

  def playable_note_key
    {
      0 => { e: :e2 },
      1 => { a: :a2 },
      2 => { d: :d3 },
      3 => { g: :g3 },
      4 => { b: :b3 },
      5 => { e: :e4 },
    }
  end
end
```

The number next to the note (the `2` in `:e2` for the low E string) represents
the octave.

A helpful thing here is that the note is still a number to Sonic Pi. We can add
the fret number pressed on the string to the root note of the string and Sonic
Pi will know what note that is.
We'll construct a Sonic Pi command to send to our [amplifier]({{< ref "/using-sonic-pi-to-play-music-with-ruby" >}})
to play that note.

```ruby
class StringSound
  def amp_value
    "(note(:#{playable_note_root}) + #{@fret_number})"
  end
end
```

This is all in a string (the data type, not the part of the instrument),
because we're going to pass it to Sonic Pi via the
[sonic-pi-cli gem](https://github.com/Widdershin/sonic-pi-cli/).
This is going to execute the `note` method in Sonic Pi to play that single
tone.

## Plucking a Single String

Our guitarist is interfacing with the guitar as a whole, which is [composed]({{< ref "/synthesizing-composition-with-delegation" >}}) of
many strings. They'll first place their fingers on the neck of the guitar.

```ruby
class FingerPlacement
  attr_reader :fret
  attr_reader :string_number
end
```

And pluck an individual string with that placement.

```ruby
class Guitar
  def pick(finger_placement, duration: 1)
    result = strings[finger_placement.string_number].pluck(fret: finger_placement.fret)
    @amplifier.play(sound_output("play #{result.amp_value}", duration: duration))
  end
end
```

Here our guitar is adding details to the command that we'll send to
Sonic Pi. We have the information about the note to play from the string, but
now we want it to sound like a note from a guitar, and we'll rely on the
guitarist to say how long to play the note for (the duration).

We can do this in Sonic Pi by specifying the synthesizer to use when playing the
note, and we'll choose one that sounds like a guitar.

```ruby
class Guitar
  def sound_output(play_operation, duration: 1)
    [
      "with_synth :pluck do",
      "#{play_operation}, release: #{duration}",
      "end",
    ].join("\n").strip
  end
end
```

If you wanted to play this directly in Sonic Pi's IDE, it would look more
familiar:

```ruby
with_synth :pluck do
  play note(:e2 + 1), release: 1
end
```

However, we need to package this all up in a string to then send that command
over to Sonic Pi via the sonic-pi-cli gem.

Our amplifier, passed in via [dependency injection]({{< ref "/dependency-injection-plug-in" >}}),
then takes that command and sends it to Sonic Pi, producing a sound!

## Strike a Chord

Sonic Pi already knows how to play [chords](https://sonic-pi.net/tutorial#section-8-2),
so this could be a quick section; however, we're going to replicate that
functionality a little differently. We're doing this because of the reality I
mentioned when talking about strings - and that is, they can break. If a string
is broken, the note in the chord that string would regularly play shouldn't be
heard.

As such, we need to go string by string to determine the notes to play. Even
though the reasoning is to handle broken strings, we're not going to consider
that case in this explanation. You can view the [full
implementation](https://github.com/kevin-j-m/ruby_cover_band/blob/09e7b72b38dac09d4968afe1468eda53caaf294c/lib/ruby_cover_band/instruments/guitar.rb#L23-L41)
to see how that's handled.

We first need to know which notes we should play:

```ruby
class Guitar
  def strum(chord, duration: 1)
    notes = [
      strings[0].pluck(fret: chord.first_fret),
      strings[1].pluck(fret: chord.second_fret),
      strings[2].pluck(fret: chord.third_fret),
      strings[3].pluck(fret: chord.fourth_fret),
      strings[4].pluck(fret: chord.fifth_fret),
      strings[5].pluck(fret: chord.sixth_fret),
    ].map(&:amp_value)
  end
end
```

We'll then take all of those notes and pass them to our amplifier, using Sonic
Pi's `play_pattern_timed` [method](https://github.com/hashbangstudio/Sonic-Pi-Examples/blob/master/10-play-pattern-timed.rb).
This also allows us to define a time between each note, so we can place a small
amount of time in between each to simulate the time it would take your hand to
complete a downstroke across all the strings.

```ruby
class Guitar
  def strum(chord, duration: 1)
    notes = [...].map(&:amp_value)

    @amplifier.play(
      sound_output(
        "play_pattern_timed [#{pattern_notes.join(", ")}], 0.05",
        duration: duration,
      )
    )
  end
end
```

The 0.05 is our amount of time it'll take to pluck from one string to the next
when playing a chord.

## Rock On

Combining a few key software design principles, we were able to create a
flexible, extensible, and testable system for playing music over the course of a
few blog posts.

We're now armed with an amplifier that knows how to communicate with [Sonic Pi]({{< ref "/using-sonic-pi-to-play-music-with-ruby" >}})
that's passed in to our guitar via [dependency injection]({{< ref "/dependency-injection-plug-in" >}}) (but could send the notes anywhere as long as the injected class [responds]({{< ref "/shedding-light-on-duck-typing" >}}) to the right methods). Our guitar is [composed]({{< ref "/synthesizing-composition-with-delegation" >}}) of various strings, each of which are responsible for knowing what sound to make.

Given a songwriter who knows how to
[consistently write]({{< ref "/inheritance-derivative-songwriting" >}}) for our band, we can
play chords and individual notes on our guitar as the [song](https://github.com/kevin-j-m/ruby_cover_band/blob/09e7b72b38dac09d4968afe1468eda53caaf294c/lib/ruby_cover_band/songs/the_line_begins_to_blur.rb) requires.

{{< rawhtml >}}
<iframe width="560" height="315" src="https://www.youtube.com/embed/GncJGXdS6R8?rel=0" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
{{< /rawhtml >}}

If you listen closely at :14, you can hear a string break. Even with these
principles in place, mistakes and errors happen. Make sure your system is
prepared to handle errors in a fault-tolerant way - but that's a different blog
series altogether. Thanks for joining me in this exploration.

> This post originally published on [The Gnar Company blog](https://blog.thegnar.co/stringing-code-together-to-play-music).

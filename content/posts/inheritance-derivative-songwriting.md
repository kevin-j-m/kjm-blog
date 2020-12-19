+++
title = "Inheritance: Derivative Songwriting"
date = 2020-12-10T16:56:38-05:00
tags = ["ruby", "software-design", "software-design-concert-series"]
summary = "There are only 12 notes to choose from"
+++

## Ruby Software Design Concert Series

1. [Dependency Injection: Plug In]({{< ref "/dependency-injection-plug-in" >}})
2. [Shedding a Light on Duck Typing]({{< ref "/shedding-light-on-duck-typing" >}})
3. [Synthesizing Composition With Delegation]({{< ref "/synthesizing-composition-with-delegation" >}})
4. __Inheritance: Derivative Songwriting__
5. [Using Sonic Pi To Play Music With Ruby]({{< ref "/using-sonic-pi-to-play-music-with-ruby" >}})
6. [Stringing Code Together To Play Music]({{< ref "/stringing-code-together-to-play-music" >}})

## Setting the Stage

Inheritance sets up a relationship or a taxonomy between classes to allow for
code reuse. It is both a commonly reached for and commonly derided tool which
has its place, but must be wielded with care. We'll use inheritance to write new
songs for our concert setlist, an example which comes from my [RubyConf 2020](https://youtu.be/EyLO0EEm3BQ)
talk about Ruby's
[Coverage](https://docs.ruby-lang.org/en/master/Coverage.html) module.

## Song Structure

When you create a song, it needs a name (or at least a working title) and a
series of notes. The notes may change over time, and the title may be refined,
but for our purposes, we're not calling it a song until there's a bit more than
an empty page.

```ruby
class Song
  def initialize(notes: [], name:)
    @notes = notes
    @name = name
  end
end
```

When you're writing songs for a band or yourself, you need to be able to play the
song. In this example, our song is written for a band that has a known number of
instruments.

```ruby
class Song
  def play
    @notes.map do |note|
      composition = []

      composition << Thread.new { @guitar.play(note) }
      composition << Thread.new { @vocal.sing(note) }
      composition << Thread.new { @drum.hit(note) }
      composition << Thread.new { @keyboardist.program(note) }

      composition.map(&:value)
    end
  end
end
```

For every note (representing a beat or measure of the song) each member of the
band needs to play their part simultaneously. All of these instruments playing
together note for note comprise the song.

## On Repeat

A touring band is going to play the same song __many__ times night after night.
For each concert on the tour, the band needs to construct a setlist of all the
songs that they'll play that night, and in what order.

```ruby
class Setlist
  def add_song(song)
    @songs << song
  end
end
```

Transcribing all the notes for each song over and over again for every concert
would be tedious and unnecessary. To save all that work, each song that could
appear in the band's setlist is catalogued as a separate class.

```ruby
class TheLineBeginsToBlur
  def initialize
    @name = "The Line Begins To Blur"
    @notes = verse_1 + chorus + verse_2 + chorus + solo + outro
  end
end
```

We don't need to accept any arguments for the name of the song or the notes
because it's already a fully-formed song. We're not going to change the
arrangement in the middle of the tour. However, we __do__ need to be able to
play the song. As such, let's copy and paste the `play` method as something we
can do for our specific song here.

```ruby
class TheLineBeginsToBlur
  def play
    @notes.map do |note|
      composition = []

      composition << Thread.new { @guitar.play(note) }
      composition << Thread.new { @vocal.sing(note) }
      composition << Thread.new { @drum.hit(note) }
      composition << Thread.new { @keyboardist.program(note) }

      composition.map(&:value)
    end
  end
end
```

This is great because we now have a stable of songs we can pull from every night
when creating our setlist; however, rewriting the `play` method in each song is
not great. If the implementation of `play` needs to change, we need to
propagate that change across every song. If we forget to add a `play` method to
one of our songs, everyone is going to look foolish when the band is staring
blankly at each other, unsure of what to do.

## Composing a Song

Taking a note from our earlier post on
[composition and delegation]({{< ref "/synthesizing-composition-with-delegation" >}}), we can
build a class that's solely responsible for playing the song.

```ruby
class SongPerformer
  def initialize(notes)
    @notes = notes
  end

  def play
    @notes.map do |note|
      composition = []

      composition << Thread.new { @guitar.play(note) }
      composition << Thread.new { @vocal.sing(note) }
      composition << Thread.new { @drum.hit(note) }
      composition << Thread.new { @keyboardist.program(note) }

      composition.map(&:value)
    end
  end
end
```

All of our songs can then use that performer and delegate the responsibility of
playing to it.

```ruby
class TheLineBeginsToBlur
  def play
    SongPerformer.new(@notes).play
  end
end
```

We have now isolated the responsibility of playing the song to one place. If we
need to change the way in which songs are played in totality, we can do so in
the `SongPerformer` and that change will be reflected in all of our songs. We
can even [dependency inject]({{< ref "/dependency-injection-plug-in" >}}) the performer class
into the song, allowing us to set up different arrangements of the same song.
Even with those benefits, we *do* still have to remember to implement a `play`
method that calls our `SongPerformer`.

There is another option we can explore: inheritance.

## Playing the Hits

We can leverage our existing, generic, `Song` class and have all of our classes
about specific songs *inherit* the behavior of the `Song` class.

By doing this, our different songs don't need to implement the `play` method.
They'll get this behavior from `Song`.

```ruby
class TheLineBeginsToBlur < Song
  def initialize
    super(
      name: "The Line Begins To Blur",
      notes: verse_1 + chorus + verse_2 + chorus + solo + outro,
    )
  end
end
```

We denote that we're inheriting from the `Song` class with `< Song`. `Song` is
our "base class". In our constructor, we then call `Song`'s constructor with
`super`, passing in the title of the song and the notes that should be played
with the song. `TheLineBeginsToBlur` has no reference to `play` in its class
definition. It still responds to it because `Song` does, and we're
inheriting all of `Song`s behavior.

When we discussed composition, we mentioned Sandi Metz's [Practical Object-Oriented Design In Ruby](https://www.poodr.com/)
for her recommendation to use composition when modeling a *has a* relationship.
In that same section, she recommends using inheritance when you encounter an
*is a* relationship. In our case, a particular song is a __specialized__
version of our `Song` class.

Inheritance is a common design choice in Object-Oriented languages. Specifically
in Ruby, if you've worked with Rails, then you've likely used inheritance all
over the place. All of your models inherit from `ApplicationRecord` (ultimately
inheriting from `ActiveRecord::Base`) and all of your controllers inherit
from `ApplicationController` (ultimately inheriting from
`ActionController::Base`).

## A Measured Approach

Inheritance does come with some drawbacks. Enough that it's commonly recommended
to avoid. You may have encountered the phrase, "prefer composition over
inheritance" before. Let's discuss why that is.

### Transparency

Inheritance makes it more difficult to know what behaviors a particular class
has. None of our song classes that inherit from `Song` have a `play` method
in their class definition. However, because they all inherit from `Song`, they
all respond to `play`. Determining that is not obvious based on a quick reading
of the class.

### Limitations of Base Class

Any inheriting classes shouldn't necessarily do things differently than how the
base class does. Of course, you *can* do this, but it should be used very
judiciously. We could redefine the `play` method in a particular class - sharing
the rest of the behavior and redefining `play` for our one-off special
exception. The issue is that these exceptions start to pile up, we end up
chipping away at the commonality, and the shared understanding of what it means
to inherit from the base class gets eroded with each change that seems small
on its own.

For our songs, if we suddenly need to write a song for a string quartet, our
`Song` class isn't helpful. It assumes a guitar, vocalist, drummer, and
keyboardist. While particularly in Ruby we have an out by being able to redefine
any method definition, from a design perspective, we should be willing to accept
the limitations that inheritance places on us within the scope of our domain.
If those limitations cannot be respected, then consider another organizational
structure, like composition.

### Future Inflexibility

It's often impossible to know how your system will evolve over time. Inheritance
can lock you in to a very specific representation of how your system should be
modeled, and the assumptions that went into developing that structure may not
hold true as features are needed to be added and the needs that the application
must serve grow.

This rigidity over time ends up getting pushed and strained
enough that maintaining inheritance structures becomes difficult. In my
opinion, it is this long-view perspective that becomes the principal reason why
inheritance is sparsely recommended by practitioners. It can work great as
long as you have perfect knowledge about both the current and future state of
your system. The reality is, it's extremely rare to be in that situation.

In this example, our application is modeling a concert tour for one band, the
members and makeup of which __shouldn't__ change throughout the course of the
tour. We've made the bet that even if the guitarist we start the tour with is
replaced, there will still __be__ a guitarist, and we will not have picked up a
french horn player along the way to play two of the songs. From a practicality
standpoint, it's reasonable to be tied to this rigid structure of how to play
each of these songs on stage in the context of this application. However, from
the onset, we've already identified one way in which this structure may come
back to haunt us.

## Rock On

Inheritance is often reached for as a quick and easy way to achieve code reuse.
It does just that; however, it imposes limitations and constraints on your system
that can make it more difficult or painful to change over time. Those
limitations may be intentional and required guardrails - but often times, they
end up being factors that cause pain, tears, multiple "code spikes", and
"technical debt sprints" to allow for needed future functionality. Inheritance
shouldn't be avoided wholesale based on this, but it should be carefully and
judiciously applied in your systems.

Our next post will move a little further from theory and explore how to build an
[interface to Sonic Pi]({{< ref "/using-sonic-pi-to-play-music-with-ruby" >}}),
so that these principles can work together to actually make sounds on your
computer.

> This post originally published on [The Gnar Company blog](https://blog.thegnar.co/inheritance-derivative-songwriting).

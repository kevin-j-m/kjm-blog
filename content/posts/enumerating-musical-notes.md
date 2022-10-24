+++
title = "Enumerating Musical Notes"
date = 2022-10-13T17:00:10-04:00
tags = ["ruby", "conference", "rubyconf", "anyone-can-play-guitar"]
categories = []
summary = "Orchestrating an array of methods"
description = "This post utilizes a score of Array and Enumerable methods to model music notes."
+++

I'm teaching a computer how to play the guitar for [RubyConf Mini 2022]({{< ref "play-guitar" >}}). Let's understand some basics before focusing on the guitar. Guitars can play many musical notes - actually, all of them! In this post, we'll output which note we play with the help of Ruby's [Array](https://ruby-doc.org/core-3.1.2/Array.html) class and [Enumerable](https://ruby-doc.org/core-3.1.2/Enumerable.html) module.

## Note Notation

There are only so many notes in music. 12 in fact. This is not going to be a lesson in music theory, so I'll spare you the details. We'll store each of the notes in an array.

```ruby
notes = [
  :a_flat,
  :a,
  :b_flat,
  :b,
  :c,
  :d_flat,
  :d,
  :e_flat,
  :e,
  :f,
  :g_flat,
  :g,
]
```

## Chromatic Scale

Putting all these notes together forms the [Chromatic scale](https://en.wikipedia.org/wiki/Chromatic_scale). Most often, you'll see the scale start with the "C" note.

Our collection of notes doesn't start with "C" though.

```ruby
notes.first
=> :a_flat
```

We can search for where "C" is in our array using the [index](https://ruby-doc.org/core-3.1.2/Array.html#method-i-index) method.

```ruby
notes.index(:c)
=> 4
```

The scale starts at the fifth note in our collection. Ruby arrays are zero-indexed; the first element is at position, or index, zero. Index four, where our "C" is, is the fifth element. We'd like it to be the [first](https://ruby-doc.org/core-3.1.2/Array.html#method-i-first), and have all the other notes follow in the expected order. We can use [rotate](https://ruby-doc.org/core-3.1.2/Array.html#method-i-rotate) with our index to accomplish this.

```ruby
chromatic = notes.rotate(4)
=> [:c, :d_flat, :d, :e_flat, :e, :f, :g_flat, :g, :a_flat, :a, :b_flat, :b]
```

To make it a little more clear where that 4 came from, we can use the return from our call to `index` as the argument to `rotate`.

```ruby
chromatic = notes.rotate(notes.index(:c))
=> [:c, :d_flat, :d, :e_flat, :e, :f, :g_flat, :g, :a_flat, :a, :b_flat, :b]
```

## Octaves

Aren't there are more than 12 [sounds](https://en.wikipedia.org/wiki/Musical_note) that comprise the entirety of possible music? Yes! These notes repeat themselves at a higher frequency. The same note with double the frequency is an octave.

Our collection of notes has a definitive [end](https://ruby-doc.org/core-3.1.2/Array.html#method-i-last) though.

```ruby
chromatic.last
=> :b
```

We'd like to repeat the collection to play the next octave. We'll treat our notes as a series that we can [cycle](https://ruby-doc.org/core-3.1.2/Enumerable.html#method-i-cycle) through again and again.

```ruby
note_cycle = chromatic.cycle
```

Calling `cycle` with no block *and* not providing a value for the number of times to cycle returns an [Enumerator](https://ruby-doc.org/core-3.1.2/Enumerator.html) that cycles forever.

To demonstrate, let's ask for 40 elements from our cycle.

```ruby
result = []
40.times { result << note_cycle.next }
result.count(:c)
=> 3
```

Our `notes` array only has 12 elements. By calling `cycle` on it this way, we infinitely repeat this progression. "C" shows up many times in our result, as we progress from one octave to the next.

## Classical Music

Let's build a Note class to encapsulate this behavior. We'll accept a starting point that we'll call the starting note. We'll find the starting note in our notes collection, and start our infinite cycle with that note.

```ruby
class Note
  def initialize(starting_note:)
    @note_cycle = notes.rotate(notes.index(starting_note)).cycle
  end
end
```

Our Note class will also accept an offset - a number representing how far from our starting note we'll stray. This is how far into our cycle we should reach to determine which note value we're representing.

```ruby
class Note
  def initialize(starting_note:, offset:)
    @note_cycle = notes.rotate(notes.index(starting_note)).cycle
    @offset = offset
  end
end
```

We need to find our final note. We'll [take](https://ruby-doc.org/core-3.1.2/Array.html#method-i-take) the collection of the cycle we traverse to move from the starting note to our offset. We'll return the [last](https://ruby-doc.org/core-3.1.2/Array.html#method-i-last) element of that progression as the value of our note.

```ruby
class Note
  def note_progression
    @note_cycle.take(@offset + 1)
  end

  def value
    note_progression.last
  end
end
```

While we don't need the full progression now, it'll come in handy soon.

## Stringing Together Notes

Let's bring this back to our goal - playing a note on a guitar. We tune each string on a guitar to a particular note. Plucking the string and letting it ring out will play the note it's tuned to - its starting note.

We'll get that value by setting the starting note of our Note class to be the note the guitar string is tuned to with an offset of 0.

```ruby
Note.new(starting_note: :a, offset: 0).value
=> :a
```

{{< figure src="/img/a_note.png" class="mid" alt="Letting the A string ring out on a guitar plays an A note." >}}

You can play more notes on a guitar string than that. A guitar's neck has many sections, called frets. Pressing down on the string on a fret and plucking the string produces a higher frequency sound. That's the next note in our cycle.

Each fret will be a different offset from the same starting note. We can determine which note we play on a string tuned to "A", pressing down on the third fret, with our Note class.

```ruby
Note.new(starting_note: :a, offset: 3).value
=> :c
```

{{< figure src="/img/c_note.png" class="mid" alt="Playing the A string on a guitar with your finger on the third fret plays a C note." >}}

## Fretting About Octaves

The number of frets on a guitar varies. A typical guitar may have between 18-24. That means that a string will be able to play the same note over again.

```ruby
Note.new(starting_note: :a, offset: 0).value
=> :a
Note.new(starting_note: :a, offset: 12).value
=> :a
```

These are both "A" notes, but at different octaves. We want our Note class to keep track of the octave so these don't seem to be playing exactly the same frequency.

To do this, we first need to know the octave number of our starting note. We'll change our constructor again.

```ruby
class Note
  def initialize(starting_note:, starting_octave:, offset:)
    @note_cycle = notes.rotate(notes.index(starting_note)).cycle
    @starting_octave = starting_octave
    @offset = offset
  end
end
```

We progress to the next octave when we repeat the Chromatic scale - starting at a "C" note. This is where the full progression of notes from the starting note to our resulting value will help. We'll [count](https://ruby-doc.org/core-3.1.2/Enumerable.html#method-i-count) the number of "C" notes we encounter in our progression. That's how many octaves we traversed.

```ruby
class Note
  def octave
    @starting_octave + note_progression.count(:c)
  end
end
```

Playing the starting note will have the same octave as is passed in. Playing the same note 12 frets up is the next octave.

```ruby
Note.new(starting_note: :a, starting_octave: 1, offset: 0).octave
=> 1
Note.new(starting_note: :a, starting_octave: 1, offset: 12).octave
=> 2
```

## Can you "C" an error?

Our implementation is a bit naive. If we have a string tuned to "C", and we play the starting note, we expect it to return the same octave passed in. However, it does not.

```ruby
Note.new(starting_note: :c, starting_octave: 1, offset: 0).octave
=> 2
```

We're counting the number of "C" notes we encounter in our progression. When we start with "C", we're guaranteed to get one, but we haven't changed octaves.

We need to handle this special case. We'll check if our starting note is "C" and exclude the starting note from our octave count if that's the case.

```ruby
class Note
  def c_start?
    @note_cycle.first == :c
  end

  def octaves_progressed
    cs_passed = note_progression.count(:c)

    if c_start?
      cs_passed -= 1
    end

    cs_passed
  end

  def octave
    @starting_octave + octaves_progressed
  end
end
```

Now we account for this edge case, while still working for other starting notes.

```ruby
Note.new(starting_note: :c, starting_octave: 1, offset: 0).octave
=> 1
Note.new(starting_note: :c, starting_octave: 1, offset: 12).octave
=> 2

Note.new(starting_note: :a, starting_octave: 1, offset: 0).octave
=> 1
Note.new(starting_note: :a, starting_octave: 1, offset: 12).octave
=> 2
```

## Closing Notes

We now have a class that can tell us which note we play on any string of our guitar. We leveraged Ruby's standard library to handle most of our logic. The [Array](https://ruby-doc.org/core-3.1.2/Array.html) class and [Enumerable](https://ruby-doc.org/core-3.1.2/Enumerable.html) module worked in concert with our domain knowledge. We didn't need to write any algorithms or complex transformations ourselves. And now our computer knows how to play musical notes.

That sounds great to me.

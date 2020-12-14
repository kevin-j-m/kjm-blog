+++
title = "Synthesizing Composition With Delegation"
date = 2020-12-05T14:00:55-05:00
tags = ["ruby", "software-design", "software-design-concert-series"]
summary = "Orchestrating the sum so you don't need to know the parts"
+++

## Ruby Software Design Concert Series

1. [Dependency Injection: Plug In]({{< ref "/dependency-injection-plug-in" >}})
2. [Shedding a Light on Duck Typing]({{< ref "/shedding-light-on-duck-typing" >}})
3. __Synthesizing Composition With Delegation__
4. [Inheritance: Derivative Songwriting]({{< ref "/inheritance-derivative-songwriting" >}})
5. [Using Sonic Pi To Play Music With Ruby]({{< ref "/using-sonic-pi-to-play-music-with-ruby" >}})

## Setting the Stage

Any application will be comprised of multiple components - in Object-Oriented
languages, typically classes. Sometimes these classes even work together!
External users of one of these classes may not know that behind the scenes there
are more classes working together, nor do they care. The public API does what
they need it to, and anything else is an implementation detail. However, keeping
the specialization of these different classes apart, but using them together,
is beneficial.

To demonstrate using composition to model a complex system and using delegation
in that composition, we will explore how a synthesizer can handle memory
management to store presets of sounds. This example comes from my [RubyConf 2020](https://rubyconf.org/program/sessions#session-1044)
talk about Ruby's
[Coverage](https://docs.ruby-lang.org/en/master/Coverage.html) module.

## Noise Reduction

The [synthesizer](https://en.wikipedia.org/wiki/Synthesizer) is an instrument
capable of producing a wide array of sounds. A collection of sounds and effects are known as a patch.

```ruby
class Patch
  attr_reader :sound
  attr_reader :effect
  attr_reader :filter
  attr_reader :oscillator
end
```

You can save these patches on the synthesizer's memory and recall them later for
easy access.

```ruby
patch = Patch.new
synth = Synthesizer.new

synth.save_patch(location: :b1, patch: patch)
synth.set_patch(patch)
synth.play_key(note: :a, duration: 1)
```

## Save You the Trouble

Much like the actual instrument is comprised of various subcomponents, our
`Synthesizer` is made up of various classes that specialize in its area of
expertise.

For example, our synthesizer above doesn't know how to save a patch to its
onboard memory. It relies on its patch memory to handle that.

```ruby
class Synthesizer
  def save_patch(location:, patch:)
    @patch_memory.write(location: location, patch: patch)
  end
end
```

All the synthesizer itself knows is what *message* to send to the memory to
have it do that. The synthesizer is __delegating__ the responsibility of storing
these patches to the patch memory instance.

Anyone playing the synthesizer does not need to be concerned with how it's
storing these patches, just that it's doing it. Anyone using our synthesizer
class isn't aware that there is a separate patch memory class that the
synthesizer is using.

At the same time, our synthesizer doesn't know directly how to access its memory.
It relies on the `PatchMemory` class for that, and delegates any responsibility
related to memory management to that class. As Sandi Metz describes in
[Practical Object-Oriented Design In Ruby](https://www.poodr.com/), a synthesizer
*has a* patch memory, as it *has a* series of other parts, and those are
composed together to deliver all the functionality that a synthesizer
provides.

## Key Benefits to Delegation

Delegation provides a few important drivers that make it easier to wrangle
complex systems.

### Specialization

Our patch memory component is solely focused on interfacing with the onboard
memory of the instrument, which is where it saves and recalls stored sounds.
Its tests can dig into all of the edge cases and minutiae that need to be
accounted for. The implementation can make very specific decisions so that it
is extremely performant without other areas of the system needing to worry
about that.

A synthesizer itself is a complex system. The memory management is only one
small part of it. The strength and value-add of our `Synthesizer` class is in
organizing all of these components together, knowing the right messages to pass
to them, with a public API that doesn't require intimate knowledge of all those
details. If the internals of our `Synthesizer` class handled all of this
responsibility itself, it would quickly become unwieldy, difficult to navigate,
hard to read, a challenge to troubleshoot, a burden to test, and feared when
changes are required.

### Flexibility & Reuse

In reality, there are many different kinds of synthesizer, all of which have
different capabilities. Some may be able to store 1,000 different patches on
board. Others may only have capacity for four. Still more may have expandable
memory, where you can plug in a USB device for nearly infinite storage.

Rather than needing to create entirely different synthesizer classes to handle
any of these scenarios, instead we only need to model those differences in patch
memory classes. Our synthesizer can then use any of those and still maintain the
rest of its functionality, without needing to duplicate it across different
classes.

In this [example](https://github.com/kevin-j-m/ruby_cover_band/blob/09e7b72b38dac09d4968afe1468eda53caaf294c/lib/ruby_cover_band/instruments/synthesizer.rb#L41-L47),
our synthesizer changes its memory capabilities based on the brand that it is.

```ruby
def initialize_memory
  if @brand == :moog
    @patch_memory = MoogPatchMemory.new
  elsif @brand == :nord
    @patch_memory = NordPatchMemory.new
  end
end
```

Thanks to [duck typing]({{< ref "/shedding-light-on-duck-typing" >}}), as long as these patch
memory classes respond to the same messages, our `Synthesizer` class can use
either of them interchangeably.

## Rock On

Composing classes together allows us to create a fully-functional system. A
class that uses another class to handle a request or responsibility is
delegating that duty to the helper class. Delegation can encapsulate the
knowledge of different specialties for code organization without external
consumers needing to know or care about that implementation detail. Delegating
responsibility to different classes can also make it easier for the system to
change, making it more likely to promote code reuse.

Next we're going to play one of the greatest hits in software design:
[inheritance]({{< ref "/inheritance-derivative-songwriting" >}}).

> This post originally published on [The Gnar Company blog](https://blog.thegnar.co/synthesizing-composition-with-delegation).

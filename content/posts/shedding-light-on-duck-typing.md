+++
title = "Shedding a Light on Duck Typing"
date = 2020-12-03T12:10:31-05:00
tags = ["ruby", "software-design", "software-design-concert-series"]
summary = "It's time to light the lights"
+++

## Ruby Software Design Concert Series

1. [Dependency Injection: Plug In]({{< ref "/dependency-injection-plug-in" >}})
2. __Shedding a Light on Duck Typing__

## Setting the Stage

Duck typing is commonly used by Rubyists and other users of dynamic languages.
We'll demonstrate duck typing by helping a concert lighting team set up the
lighting for a band, which comes from my [RubyConf 2020](https://rubyconf.org/program/sessions#session-1044) talk about Ruby's
[Coverage](https://docs.ruby-lang.org/en/master/Coverage.html) module.

## Stage Design

To light the stage for our concert, we have a wide range of lights to
use.

We have our trusty can, or [PAR](https://en.wikipedia.org/wiki/Stage_lighting_instrument#PAR_lights), lights.

```ruby
class CanLight
  def trigger(color:, effect:)
  ...
  end
end
```

We have [spotlights](https://en.wikipedia.org/wiki/Stage_lighting_instrument#Spotlights) tasked on each band member.

```ruby
class Spotlight
  def trigger(color:, effect:)
  ...
  end
end
```

We have fancy [moving lights](https://en.wikipedia.org/wiki/Intelligent_lighting) for versatile coverage across the stage.

```ruby
class MovingLight
  def trigger(color:, effect:)
  ...
  end
end

```

We even have a [beam projector](https://en.wikipedia.org/wiki/Beam_projector)
for a more powerful spotlight effect.

```ruby
class BeamProjector
  def trigger(color:, effect:)
    ...
  end
end
```

What any of these lights do isn't important here. What is key to notice is that
you operate them all by calling the `trigger` method.

## A Light Touch

The stage lighting technicians, just like the band, are performers in the
concert. For every single note of every single song, they need to make sure that
the visual aesthetic of the stage is set *just so*.

All of these lights are managed by a central controller, from which they can
power on all the lights in preparation for a show.

```ruby
class LightingController
  def initialize
    @powered_lights = {}
  end

  def turn_on_lights
    @powered_lights[:beam_projector] = BeamProjector.new
    @powered_lights[:can] = CanLight.new
    @powered_lights[:moving_light] = MovingLight.new
    @powered_lights[:spotlight] = Spotlight.new
  end
end
```

As I mentioned, for every note of every song, they need to make sure the lights
look exactly as they're supposed to. This is tracked as the lighting's
composition.

```ruby
class LightingComposition
  attr_reader :light_name
  attr_reader :color
  attr_reader :effect
end
```

## Ducking into Lights on Stage

As the band is playing the show, the lighting technicians follow note-for-note
and need to apply the composition.

{{< highlight ruby "hl_lines=10" >}}
class Song
  def play
    @notes.map do |note|
      composition = []

      composition << Thread.new { @guitar.play(note) }
      composition << Thread.new { @vocal.sing(note) }
      composition << Thread.new { @drum.hit(note) }
      composition << Thread.new { @keyboardist.program(note) }
      composition << Thread.new { @lighting.set_lighting(note) }

      composition.map(&:value)
    end
  end
end
{{< /highlight >}}

Because each of our different lights respond to the same message (`trigger`)
with the same signature, the lighting controller doesn't need to care, or even
know, about which light it's operating. All it knows is that it needs to send
it the trigger signal and apply the required composition.

```ruby
class LightingController
  def set_lighting(note)
    lighting_composition = note.lighting

    trigger(@powered_lights[lighting_composition.light_name], lighting_composition)
  end

  def trigger(light, composition)
    light.trigger(
      color: composition.color,
      effect: composition.effect,
    )
  end
end
```

The `LightingController`'s `trigger` method is taking advantage of duck typing.
Ruby doesn't care what kind of object it's calling in its `light` argument. All
that matters is that it responds to `trigger`. We also used duck typing when we
discussed [dependency injection]({{< ref "/dependency-injection-plug-in" >}}). Our guitar
didn't care how the amplifier made sound, or even if it did make sound. All that
matters at runtime to satisfy Ruby is that the object we pass in responds to
`play` and accepts an argument.

## Static Lighting

If you're more familiar with static languages or different typing systems, and
you need to define common behavior for what a set of classes do, you may be
familiar with an interface. For example, let's use Java to define an interface
for our lights.

```java
interface Light {
  void trigger(Color color, LightingEffect effect)
}
```

Each of our lights would then implement this interface, defining their own
implementation of what they do when the light is triggered.

```java
class Spotlight implements Light {
  @Override
  public void trigger(Color color, LightingEffect effect) {
    // Turn the light on or off
  }
}
```

We can now set our `LightingController`'s `trigger` method to accept any kind of
light.

```java
class LightingController {
  public void trigger(Light light, LightingComposition composition) {
    light.trigger(composition.color, composition.effect);
  }
}
```

This satisfies Java's type system, because anything that implements
the `Light` interface is required to respond to the `trigger` method accepting
those types of arguments.

Because of duck typing in Ruby, defining this contract and enforcing it is
unnecessary. However, Ruby 3 will be [shipping](https://www.ruby-lang.org/en/news/2020/09/25/ruby-3-0-0-preview1-released/) with a way to define type
signatures, called [RBS](https://github.com/ruby/rbs). RBS includes a mechanism
to define interfaces, which you can read more about [here](https://developer.squareup.com/blog/the-state-of-ruby-3-typing/).

## Rock On

Duck typing is a core design feature of Ruby and other dynamic languages;
however, it does require a degree of trust. Because everything is determined at
runtime, there's nothing stopping you from passing in an object to a method that
doesn't respond to the methods it needs to. That will generate an
exception, but that may be too late to get that feedback. However, to many
Rubyists, the flexibility this approach provides often outweighs the cost.

If you're coming from a static typing system, or desiring more direction or
enforcement about what's expected to be provided as an argument, then
investigate defining interface types with RBS, which will be part of Ruby 3.

> This post originally published on [The Gnar Company blog](https://blog.thegnar.co/shedding-light-on-duck-typing).

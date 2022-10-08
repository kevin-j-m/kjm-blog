+++
title = "Composing Our Own Guitar Amps From Inherited Gear"
date = 2023-04-15T15:00:10-04:00
tags = ["ruby", "software-design", "anyone-can-play-guitar"]
summary = "No soldering or electronics experience required"
description = "This post explores inheritance and composition as a way to build and share behavior in a system to model guitar amplifiers."
+++

## Anyone Can Play Guitar Series

1. [Enumerating Musical Notes]({{< ref "/enumerating-musical-notes" >}})
2. [Revisiting Calling Sonic Pi From Ruby]({{< ref "/revisiting-calling-sonic-pi-from-ruby" >}})
3. [Programming Guitar Greatness]({{< ref "/programming-guitar-greatness" >}})
4. __Composing Our Own Guitar Amps From Inherited Gear__

## Getting Amped Up

I [previously proposed]({{< ref "/revisiting-calling-sonic-pi-from-ruby#power-amp" >}}) a simplified description of an amplifier. They amplify sounds with the help of two components: a pre amp and a power amp. The sound changes as it progresses through the pre amp and power amp. Finally, it projects through the speaker.


```ruby
class Amplifier
  def amplify(sound)
    pre_amp_stage(sound)
    power_amp_stage(sound)
  end
end
```

With this in place, our system can output sound. That'd make a pretty short blog post - but we're not done. Guitarists like fiddling with gear. There are many different kinds of amplifiers they may use.

## Tube Amp

A tube amp uses [vacuum tubes](https://en.wikipedia.org/wiki/Vacuum_tube) to output sound, which is how it gets its name. Guitarists love these amps because of the way they modify the sound of a guitar. The tone that comes out of a tube amp is often described as warm. And as you push the amp to higher volumes, it sounds even warmer.

```ruby
class TubeAmp
  def pre_amp_tone
    if low_volume? || mid_volume?
      "💡"
    elsif high_volume?
      "🔥"
    end
  end
end
```

Unfortunately, these vacuum tubes are relatively __heavy__ electrical components. It's a good thing they sound so good. So good that a guitarist is willing to risk the health of their back lugging them on and off stage.

```ruby
> amp = TubeAmp.new
> amp.weight
=> :heavy
```

Right now, our amp can change the sound to provide its signature warmth. It can also put you at risk of months of physical therapy. One thing it can't do yet is output that sound. We already built that functionality with our `Amplifier` class. We'll add it into our tube amp by inheriting from the amplifier we've already built.

```ruby
class TubeAmp < Amplifier
end

> amp = TubeAmp.new
> amp.respond_to?(:amplify)
=> true
```

## Solid State Amp

Solid state amps have a different tone profile than tube amps. They're described as having a clean sound - as clear as glass. That tone persists no matter how much you push the amp past breakup. It maintains its relative clarity, as opposed to tube amps that get even warmer.

```ruby
class SolidStateAmp
  def pre_amp_tone
    "🫙"
  end
end
```

A solid state amp uses [transistors](https://en.wikipedia.org/wiki/Transistor) to send signals to the speaker. These are much lighter than vacuum tubes, and easier on your back.

```ruby
> amp = SolidStateAmp.new
> amp.weight
=> :light
```

A solid state amp also needs to amplify sound like a tube amp. Let's do the same thing and inherit from the `Amplifier` class to gain that behavior.

```ruby
class SolidStateAmp < Amplifier
end

> amp = SolidStateAmp.new
> amp.respond_to?(:amplify)
=> true
```

## Inheritance

With both our tube amp and solid state amp, we inherited from an amplifier class. The `Amplifier` has all the behavior that's consistent between different types of amps. By inheriting from it, the amps gain that behavior and structure without having to rewrite it. Those amps then apply their specialization on top of it. Both amps produce a different tone. The different electrical components they use result in different weights.

Inheritance works well here because each of these amplifiers __are__ still an amplifier. They share the same basic data and behavior, and should continue to have that in common. They have pieces that make their interaction with, or use of, that data or behavior special.

We encode that specialization in that class, and still use the original behavior. `Amplifier#amplify` pushes our sound through its `pre_amp_stage` method. That stage modifies the sound with the amp's `pre_amp_tone`. That allows a tube amp to have its characteristic warmth and a solid state amp to apply its clarity.

## Hybrid Amp

Let's build one more type of amp - one that aims to be the best of both worlds. A hybrid amp takes parts of a tube amp and other parts from a solid state amp and combines them.

We want both tube and solid state behavior. Let's attempt the same inheritance approach we've used so far.

```ruby
class HybridAmp < TubeAmp, SolidStateAmp
end
```

When we load up our console we'll see an error about our class.

```ruby
(irb):1: syntax error, unexpected ',', expecting ';' or '\n' (SyntaxError)
```

Ruby doesn't support [multiple inheritance](https://en.wikipedia.org/wiki/Multiple_inheritance). We can only have a single class that we inherit from. But we know our `HybridAmp` needs behavior that's the same as each of these other classes. To consider a different approach, let's revisit our definition of an amplifier.

```ruby
class Amplifier
  def amplify(sound)
    pre_amp_stage(sound)
    power_amp_stage(sound)
  end
end
```

Each amplifier has a pre amp and a power amp. Let's build separate modules for each of these components' behavior.

A hybrid amp has a tube amp pre amp to replicate its warmth.

```ruby
module TubePreAmp
  def pre_amp_tone
    if low_volume? || mid_volume?
      "💡"
    elsif high_volume?
      "🔥"
    end
  end
end
```

By including this module in our amp, any `HybridAmp` instances will gain all the behavior in the module.

```ruby
class HybridAmp
  include TubePreAmp
end

> amp = HybridAmp.new(volume: 10)
> amp.pre_amp_tone
=> "🔥"
```

A hybrid amp uses the same power amp as solid state amps to save some weight.

```ruby
module SolidStatePowerAmp
  def power_amp_weight
    :light
  end
end
```

Unlike inheritance, we can include as many modules as we'd like. Let's also include the `SolidStatePowerAmp` behavior in our `HybridAmp`.

```ruby
class HybridAmp
  include TubePreAmp, SolidStatePowerAmp
end

> amp = HybridAmp.new(volume: 10)
> amp.power_amp_weight
=> :light
```

A hybrid amp is still a kind of amplifier, and it still needs the ability to amplify sound. As such, we are *also* going to use inheritance here. Our hybrid amp is a specialization of `Amplifier`, just like a tube or solid state amp. Its specializations come from the modules it includes.

```ruby
class HybridAmp < Amplifier
  include TubePreAmp, SolidStatePowerAmp
end

> amp = HybridAmp.new(volume: 10)
> amp.respond_to?(:amplify)
=> true
```

## Composition

We are unable to pull behavior from multiple places with inheritance. We couldn't apply our initial strategy to build our hybrid amp. We didn't want to duplicate the behavior. They are __intentionally__ the same. We needed a different approach.

We can combine sets of related behavior in modules, and share them between classes. We compose those modules together in our classes to use the behavior they provide.

With this composition approach, our tube and solid state amps become pretty small.

```ruby
class TubeAmp < Amplifier
  include TubePreAmp, TubePowerAmp
end

class SolidStateAmp < Amplifier
  include SolidStatePreAmp, SolidStatePowerAmp
end
```

The special behavior for each of the amps lives in the modules so that different amp types can use them. We use modules to share that behavior across classes. That allows our hybrid amp to operate the same way as parts of our other amps.

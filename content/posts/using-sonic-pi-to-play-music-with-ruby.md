+++
title = "Using Sonic Pi to Play Music With Ruby"
date = 2020-12-14T18:48:24-05:00
tags = ["ruby", "software-design", "software-design-concert-series"]
summary = "Amplifying the impact of your code"
description = "Sonic Pi is a fun and approachable way to make music with code. For a slight twist on using it, you can read about how I incorporated it into an existing ruby code base."
canonicalUrl = "https://www.thegnar.com/blog/using-sonic-pi-to-play-music-with-ruby"
+++

## Ruby Software Design Concert Series

1. [Dependency Injection: Plug In]({{< ref "/dependency-injection-plug-in" >}})
2. [Shedding a Light on Duck Typing]({{< ref "/shedding-light-on-duck-typing" >}})
3. [Synthesizing Composition With Delegation]({{< ref "/synthesizing-composition-with-delegation" >}})
4. [Inheritance: Derivative Songwriting]({{< ref "/inheritance-derivative-songwriting" >}})
5. __Using Sonic Pi To Play Music With Ruby__
6. [Stringing Code Together To Play Music]({{< ref "/stringing-code-together-to-play-music" >}})

## Setting the Stage

My [RubyConf 2020](https://youtu.be/EyLO0EEm3BQ)
talk about Ruby's [Coverage](https://docs.ruby-lang.org/en/master/Coverage.html) module
uses examples about playing live music. As such, I had the ambitious goal of
delivering a live performance of some music during the presentation. This ended
up getting cut for a variety of reasons (time, concern about the audio working
on the streaming platform, the reality of ambition turning into actual work to
do), but I built out the structure to support this for one instrument, the
guitar. This is the first of two posts that'll describe the work that I did to
support this.

First, I had to figure out if it was possible to make this happen. I wanted to
hook into my existing code samples and trigger musical notes from them somehow.
As such, I decided to build my first amplifier, virtually, without fear of
blowing up any capacitors.

## Parts List

In the earlier post on [dependency injection]({{< ref "/dependency-injection-plug-in" >}}),
I created a `PracticeAmplifier` [class](https://github.com/kevin-j-m/ruby_cover_band/blob/09e7b72b38dac09d4968afe1468eda53caaf294c/lib/ruby_cover_band/practice_amplifier.rb)
that did nothing so I could use it in tests, rather than the "regular" amplifier.

What the "regular" [amplifier](https://github.com/kevin-j-m/ruby_cover_band/blob/09e7b72b38dac09d4968afe1468eda53caaf294c/lib/ruby_cover_band/amplifier.rb)
does is interface with [Sonic Pi](https://sonic-pi.net/), which is awesome
software that'll make sound and music driven by code. Sonic Pi comes with
an [IDE](https://sonic-pi.net/tutorial.html#section-1-2) of sorts that you can
use to program the composition you'd like to play, and get immediate feedback
from hearing how your code is translated into audio. It's a great way to lose
track of time for a night or two (or more). However, I was envisioning
controlling my audio from the code examples directly. I didn't want to have to
work within the IDE.

To get around using the IDE directly, I found the [sonic-pi-cli](https://github.com/Widdershin/sonic-pi-cli)
gem. Its principal use case is to be used directly in the
[terminal](https://github.com/Widdershin/sonic-pi-cli/blob/c4280f98edcec4de99801d013ec946cc47787932/bin/sonic_pi).
However, it's a gem, and written in ruby, and the core functionality is
available in a [class](https://github.com/Widdershin/sonic-pi-cli/blob/c4280f98edcec4de99801d013ec946cc47787932/lib/sonic_pi.rb)
that you can use in any of your code.

## Wiring Schematic

With enough knowledge and conviction to be dangerous, I set out wiring up my
amplifier. The CLI requires that Sonic Pi itself is running, and first ensures
it can communicate with it - and to do so, it needs to know what port the
software is running on. Sonic Pi used to always run on the same port; however,
it has since changed to run on a [dynamically-determined](https://github.com/sonic-pi-net/sonic-pi/commit/d245d93c5b797ad76fa333f829c32d67480af96c) port.

The CLI already implemented the functionality to [find the port](https://github.com/Widdershin/sonic-pi-cli/blob/20a18f91b4aa24de9f4b187aa20c69334ddf0329/bin/sonic_pi#L13-L33)
to send to the `SonicPi` class, so for demonstration purposes, I copied that in
my constructor.

```ruby
class Amplifier
  def initialize
    @port = find_port
    @speaker = SonicPi.new(@port)
  end

  private

  def find_port
    # Code from sonic-pi-cli
  end
end
```

Needing to find the port is now something that the `SonicPi` class can do [by itself](https://github.com/Widdershin/sonic-pi-cli/pull/23)
as of version v0.2.0; however, this work preceded that.

The rest of the functionality in the `Amplifier` class is now to [delegate]({{< ref "/synthesizing-composition-with-delegation" >}}) commands to the `@speaker`.

```ruby
class Amplifier
  def play(sound)
    @speaker.run(sound)
  end
end
```

## Rock On

Using this amplifier still requires knowing all the correct
[commands](https://sonic-pi.net/tutorial.html#section-2-1) to send to Sonic Pi,
and Sonic Pi must be running; however, we can now trigger it to execute these
commands from outside of its IDE. We have a way to *send* sound
out of our ruby code.

In our next post, we'll take a look at how we *generate* the sound to send from
a [guitar]({{< ref "/stringing-code-together-to-play-music" >}}) to an amplifier.

> This post originally published on [The Gnar Company blog](https://blog.thegnar.co/using-sonic-pi-to-play-music-with-ruby).

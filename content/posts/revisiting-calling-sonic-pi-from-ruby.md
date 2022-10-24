+++
title = "Revisiting Calling Sonic Pi From Ruby"
date = 2022-10-23T07:43:24-05:00
tags = ["ruby", "software-design-concert-series", "anyone-can-play-guitar"]
summary = "Different year, different gear"
description = "This provides a crude mechanism for piping sound output from your ruby program into Sonic Pi to have it sound like playing a guitar."
+++

## Tweaking Amp Settings

[Sonic Pi](https://sonic-pi.net/) is software to make sounds and music driven by code. Sonic Pi comes with an [IDE](https://sonic-pi.net/tutorial.html#section-1-2) of sorts. You can program the composition you'd like to play in the IDE. With one button, you get immediate feedback hearing how your code sounds.

A few years ago I wrote about controlling [Sonic Pi with Ruby]({{< ref "using-sonic-pi-to-play-music-with-ruby" >}}) code without needing to code in the IDE directly. That relied on the [sonic-pi-cli](https://github.com/Widdershin/sonic-pi-cli) gem. For my [Anyone Can Play Guitar (With Ruby)]({{< ref "play-guitar" >}}) talk, I took a different approach.

## Updated Wiring

As of this writing, Sonic Pi has released version 4.3.0. It's progressed significantly since version 3.2, which is the version that I used in my original post. One of the consequences is that the sonic-pi-cli gem does not appear to work with later versions of Sonic Pi. I couldn't get earlier versions of Sonic Pi to work on my computer. I didn't want to deal with virtualization to see if I could get an earlier version of Sonic Pi working with the gem. I also didn't have the time to update the gem to work with later version of Sonic Pi. Sorry - I had a presentation to write!

I needed a quick way to achieve the same, or similar, results. So, I'll admit - I cheated.

## Input Jack

Sonic Pi reads an [init file](https://github.com/sonic-pi-net/sonic-pi/tree/stable/app/config/user-examples#initrb) every time that the application boots. You might use this file for helper methods to recall in the application's editor. Instead, I will store the code to play the song itself. It gets read when the application boots and starts playing the song.

It's not the greatest long-term solution. You don't want to hear the song every time you open the application forevermore. I know I don't. But it is good enough for demonstration purposes. That said, I still needed to construct the code to play the song.

## Speaker

Much like in my [original version]({{< ref "/using-sonic-pi-to-play-music-with-ruby" >}}), I built an amplifier to communicate with Sonic Pi. In my original post, my guitar class knew how to generate its [sound output]({{< ref "/stringing-code-together-to-play-music#plucking-a-single-string" >}}) to the amp. In this version, the amp knows how to do that.

```ruby
class SonicPiAmplifier < Amplifier
  def sound_output(play_operation, duration: 0.25)
    [
      "with_synth :pluck do",
      "  #{play_operation}, release: #{duration})",
      "end",
      "sleep(#{duration})\n",
    ].join("\n")
  end
end
```

For each note to play, this is using [Sonic Pi's DSL](https://sonic-pi.net/tutorial.html#section-2-3) to play a note with a [synth patch]({{< ref "synthesizing-composition-with-delegation/#noise-reduction" >}}) that sounds a bit like a guitar. It plays the note for the provided duration, and then sleeps for that same duration. That's so Sonic Pi won't immediately play the next note on top of this one.

## Power Amp

To simplify, let's assume that amplifiers work by progressing sound through two components. Sound moves through a pre amp to a power amp. The power amp is what's responsible for sending the sound to the speaker. That's what will use the `sound_output` method we've built.

```ruby
class SonicPiAmplifier < Amplifier
  def power_amp_stage(sound)
    play_operation = "play(:#{sound.to_s}"
    output = sound_output(play_operation, duration: sound.duration)
    @sounds << output

    output
  end
end
```

The `play_operation` string is again a command from [Sonic Pi's DSL](https://sonic-pi.net/tutorial.html#section-2-1). As you'd expect, it plays a sound. We retrieve the value of the sound to play from the `Note` class we constructed in a [prior post]({{< ref "/enumerating-musical-notes" >}}). We pass this into our `sound_output`. We store the result in our list of `@sounds` that the amplifier projects, and return it as well.

## Audio Loopback

The reason we're keeping track of our `@sounds` is that we want to be able to replay them again after the fact. Our amplifier will use this ability to write the song it played to a file.

```ruby
class SonicPiAmplifier < Amplifier
  def write_to_file(location)
    File.open(location, "w") do |file|
      @sounds.each do |sound|
        file.write(sound)
      end
    end
  end
end
```

We'll write this to Sonic Pi's init file. Then, when we open up the application, it will play the song from the amplifier on boot.

## Output

Time to take this for a rip. We'll grab a guitar, decide which song we want to play, plug in our amplifier we just built, and play the song.

```ruby
def play
  guitar = Blues::Guitar.new
  guitar.restring(gauge_set: :srv)
  guitar.tune(:down_half_step)
  song = Blues::Shuffle.new(guitar)

  amp = Blues::SonicPiAmplifier.new(volume: 10, on: true)
  guitar.plug_in(amplifier: amp)

  song.play { |measure| measure.map { |sound| puts sound } }
  amp
end
```

Running this method in isolation won't push any air through our speakers. Instead, we need our cheat. We'll write all the sounds from the song to our init file, and then have our script open Sonic Pi in a [subshell](https://ruby-doc.org/core-3.1.2/Kernel.html#method-i-60).

```ruby
init = "#{Dir.home}/.sonic-pi/config/init.rb"
play.write_to_file(init)
`open "/Applications/Sonic\ Pi.app"`
```

Sonic Pi will start, read the init file with all the instructions to play our song, and start playing.

{{< rawhtml >}}
<iframe width="560" height="315" src="https://www.youtube.com/embed/iQUNU36Vem4" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
{{< /rawhtml >}}

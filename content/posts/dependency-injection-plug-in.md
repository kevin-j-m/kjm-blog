+++
title = "Dependency Injection: Plug In"
date = 2020-11-24T14:10:31-05:00
tags = ["ruby", "software-design"]
summary = "These go to eleven"
+++

## Setting the Stage

Dependency injection is a fancy term. It __sounds__ intimidating. The purpose of
this post is to explain what dependency injection is, how to use it, and why it
can be beneficial. To illustrate, let's talk about playing a guitar in a
concert, which comes from my [RubyConf 2020](http://rubyconf.org/program/sessions#session-1044)
talk about Ruby's [Coverage](https://docs.ruby-lang.org/en/master/Coverage.html) module.

## Sound Check

A guitarist in a band uses an amplifier when playing a live concert.

```ruby
class Guitar
  def initialize
    @amplifier = Amplifier.new
  end
end
```

When the guitar is played the sound travels through the amplifier, so the
audience can hear the notes being played.

```ruby
class Guitar
  def strum(chord)
    chord.phrasing.each do |string_sound|
      @amplifier.play(string_sound.amp_value)
    end
  end
end
```

You can play a great show with this setup! Your guitar uses the amplifier it
defines, and all is well. Until...

## Wall of Sound

Some guitarists experiment with gear - a lot. Different amplifiers are going to
make different sounds. However, we've made it very difficult for our guitar to
be plugged in to different amplifiers.

Right now, our dependence on the amplifier class to play the sound from the
guitar is hard-coded in the `Guitar` class. The initializer sets up an explicit
dependency with the `Amplifier` class.

If we want to plug the guitar into a `LouderAmplifier`, we can't do that without
changing our `Guitar` class. Every different amplifier will require a change to
our `Guitar` class.

## Plug and Play

We can resolve this limitation by instead passing in the amplifier that'll be
used with the guitar when we make a new guitar.

```ruby
class Guitar
  def initialize(amplifier)
    @amplifier = amplifier
  end
end
```

With this small change, our `Guitar` can work with any amplifier that responds
to the `play` method. Rather than being coupled to the `Amplifier` class, we
require that any users of the `Guitar` class instead explicitly pass in this
collaborating class. This is a form of dependency injection, specifically
[Constructor Injection](https://martinfowler.com/articles/injection.html#ConstructorInjectionWithPicocontainer).

## Sound Engineering

Now that we've seen an example of what dependency injection is, let's discuss
why we would want to use it.

### Flexibility

This is the motivation described in the example above. By removing the
hard-coded dependency as an implementation detail of our class, we can instead
use any dependency desired, as long as it responds to the methods that we need
to use within the class. For us, this means that guitars can use any amplifier
they'd like; the guitarist isn't limited to the amp they had when first buying
the guitar.

### Testing

Testing may be the first situation where the value of this flexibility can be
appreciated. Tests are the first consumers of your implementation, and it's
important to listen to the implicit feedback they give you. If a class or a
method is hard to test, it very likely will be hard to use - or at least complex
to understand.

In reality, the difficulty in testing the `Guitar` [class](https://github.com/kevin-j-m/ruby_cover_band/blob/09e7b72b38dac09d4968afe1468eda53caaf294c/lib/ruby_cover_band/instruments/guitar.rb)
is what led to the decision to inject the amplifier in. That's because the
`Amplifier` [class](https://github.com/kevin-j-m/ruby_cover_band/blob/09e7b72b38dac09d4968afe1468eda53caaf294c/lib/ruby_cover_band/amplifier.rb)
is essentially a wrapper around [Sonic Pi](https://sonic-pi.net/). Sonic Pi
describes itself as a "code-based music creation and performance tool", so
playing the guitar with this amplifier will actually play a sound on your
computer.

As exciting as that is, I don't want to have Sonic Pi running just to execute my
tests. And even if I did, I don't need to hear the sound it would generate every
time I run my tests. And so, I created a separate amp for testing: a
`PracticeAmplifier`. What does that amp do? [Absolutely nothing](https://github.com/kevin-j-m/ruby_cover_band/blob/09e7b72b38dac09d4968afe1468eda53caaf294c/lib/ruby_cover_band/practice_amplifier.rb)!
And that's perfect for my unit tests. They're not concerned with the sound the
amplifier makes when playing the guitar. They're interested in exercising the
logic that's within the `Guitar` class only.

More generally, maybe your class is collaborating with another class that makes
an API call or performs file I/O. You don't want to have to execute or mock out
those actions in your class's tests - it's the collaborator's tests that should
be concerned with that. You can instead pass in another class that doesn't do
those things, providing speedy and relevant feedback in your tests.

### Complexity Identification

The responsibility of systems tends to grow over time. This is true not only for
your entire application, but the different components of it, down to the
individual classes or methods. As functionality continues to get added to
classes, you may need to add in more and more collaborators. If each of these
changes are made piecemeal over time, it can be difficult to step back and
realize not only how coupled a class is to other classes, but how __many__
classes it's coupled to.

Injecting dependencies explicitly makes it more clear *what* this class is
dependent on, and *how many* things. As the list of things you need to pass in to
a constructor or a method grows to support new features, it can serve
as a proxy to gauge how complexity within the class or method is growing. This
may exert more natural pressure to identify different abstractions or
refactorings to implement.

## Rock On

Ruby's inherent flexibility can make dependency injection a less-likely tool to
reach for, particularly if [testing](https://dhh.dk/2012/dependency-injection-is-not-a-virtue.html) is when you would notice that pain initially, given the [tools](https://www.youtube.com/watch?v=iEfpAp2sqiw) at our
disposal to make testing interactions with dependencies easier.

Dependency injection is also a daunting term that often carries the assumption
that you need a heavyweight [framework](https://en.wikipedia.org/wiki/Dependency_injection#Dependency_injection_frameworks) to implement it. However, if you can pass in an object as an argument to an initializer (constructor) or even an individual method - congratulations, you've injected a dependency!

Using dependency injection can lead to less tightly-coupled code, which
allows for more flexibility in collaborating with others, reduces the burden of
testing, and makes it more clear when classes are growing to the point where
their current design needs to be reconsidered.

> This post originally published on [The Gnar Company blog](https://blog.thegnar.co/dependency-injection-plug-in).

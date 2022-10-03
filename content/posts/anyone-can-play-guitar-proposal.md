+++
title = "Anyone Can Play Guitar (With Ruby) - The Proposal"
date = 2022-10-02T20:26:22-04:00
tags = ["conference-proposal", "conference", "rubyconf"]
summary = "Presented at RubyConf Mini 2022"
description = "The proposal for a talk I gave at RubyConf Mini 2022."
+++

# Abstract

I've got the blues. I've been looking for the perfect guitar tone, but haven't found it. To amp up my mood, let's teach a computer to play the guitar through an amplifier.

Let's string together object-oriented principles to orchestrate a blues shuffle. We'll model our domain with the help of inheritance, composition, and dependency injection. This talk will strike a chord with you, whether you've strummed a guitar before or not.

# Details

To build a guitar and amplifier, this talk will introduce software design concepts. As we unwrap different requirements and intricacies, we'll explore new concepts.

We'll begin by __modeling__ some objects in our domain. This will focus on how we can identify objects in our system. [7 - 10 minutes]

We'll find different objects in our system the following ways:

- Some objects mirror the real-world counterpart that we're implementing. We can decompose the overall object into its component parts. We could have a `Guitar` class that does everything you need a guitar to do. In our exploration, we'll model our guitar as having a set of `String`s (not the data type) that work together. 
- Some objects become clear after identifying a need to pass around related data and behavior. Rather than using a hash with the individual notes played, we'll make a `Sound` class. That will be able to tell you if the notes combine to form a particular chord. This new behavior can live with the raw note data in this new class.
- A bundle of private methods only related to part of the entire class can become a separate object. It takes some work for a `String` to determine which note it's playing. We need that information to construct our `Sound`. It's also a bit misplaced within the full context of the `String`'s responsibilities.  By introducing a `Fret` class, we can relieve the `String` class from that work. We can rigorously test that part of the code in isolation. We also create a dedicated location for related behavior to live.

Our electric guitar is only one part of our system. We need to hear the sound with the help of an amplifier. When building our amplification needs, we'll explore __inheritance__. [7 - 10 minutes]

- We'll start with an `Amplifier` class that will make a noise when you call the `project` method to it, passing it a `Sound`. Of course, it only makes a noise if the amp is on. The `project` method is the last step to `amplify` a sound. `amplify` is the method that's called externally.
- There are different types of amplifiers. The different components affect the way they sound. We'll build amplifiers that inherit from the core `Amplifier` functionality. They'll extend that base to exhibit their differences.
- A `TubeAmplifier` uses electrical circuitry like a light bulb. They need to warm up to sound their best. They need to account for how warmed up they are as part of the `amplify` process.
- A `SolidStateAmplifier` uses more modern transistors and semiconductors. Because of its components, applying too much distortion results in harsh audio clipping. You wouldn't hear that from a tube amp.

Our amplifier design must incorporate __composition__ to build the next type of amplifier. [5 minutes]

- A `HybridAmplifier` has a tube pre amp, and a solid-state power amp. It needs elements of both the `TubeAmplifier` *and* the `SolidStateAmplifier`. We can't inherit from both in Ruby. Instead, we'll compose `TubeAmplification` and `SolidStateAmplification` modules together to produce a single `HybridAmplifier`.
- The value in composing with modules is in sharing behavior generally. It's not specific to addressing the lack of multiple inheritance in the language. We'll apply this concept back to guitars. An acoustic guitar has a sound hole that will `project` the sound. An electric guitar cannot do that without the help of an amplifier. An acoustic guitar is __not__ an amplifier though. Instead, we'll have a `SoundProjection` module. Both an acoustic guitar and an amplifier can use this module. The amplifier will use it as the basis of its amplification of the sound. The acoustic guitar will use it only to project its sounds.

We'll combine our guitar and amplifier with the use of __dependency injection__. [3 minutes]

- Initially, our guitar will create an instance of an `Amplifier` on object construction. It'll use that amplifier so we can hear the sounds we're playing.
- Our `Amplifier` will `put` the note that it plays to the screen. We don't want everyone to hear that noise when we practice. We'll build a separate `PracticeAmplifier` that outputs nothing. Now when we practice (in our tests), we don't hear (or see) the result of our practice.
- To use our `PracticeAmplifier`, we need to add mocks in our tests. When our guitar makes a new `Amplifier`, it'll be a `PracticeAmplifier`.
- We can make this easier by instead passing an instance of the amplifier we want to use to the guitar instance. Now our tests can use the practice amp without mocking.
- This *also* improves the usage of our guitar generally. Now we can plug our guitar into the different amplifiers we've created. We're not stuck with the base model. Though it started as a testing benefit, this change helps our application as well.

At the end, we'll use the code we've written in our presentation, and some we haven't, for a performance. We'll show how we can play a blues riff, either through [Sonic Pi](https://sonic-pi.net/), [sheet music](https://en.wikipedia.org/wiki/Sheet_music), or [guitar tablature](https://en.wikipedia.org/wiki/Tablature#Guitar_tablature).

### Notes on time estimates

The __modeling__ and __inheritance__ section estimates are longer to introduce the business domain. __Composition__ will use terminology that's already introduced.  We'll reuse and change code we've already written. __Dependency injection__ ties together the two subsystems we're already familiar with.

# Pitch

This talk will serve a wide audience, though in different ways. For early career developers, this can introduce these concepts. It'll apply them in a (hopefully) fun way, and should demystify some of the fancy terminology.

For intermediate developers, the domain modeling section will have the most value. Identifying and extracting objects out of an existing system may be new to them. They may be familiar with seeing tutorials or writing their own work from scratch. They likely have had fewer opportunities to work with existing code in need of a change.

Even if you know everything about object-oriented design, this talk may be for you. Experienced developers will leave with another ready-made analogy for these concepts. They can use this example to explain the topics to others with a more novel problem space that the classics. There will be no discussion of animal and species taxonomies here.

I've had the pleasure to speak at RubyConf and RailsConf in the past. I've used (different) musical equipment to speak about (different) technical concepts effectively before.

# Bio

Kevin Murphy lives near Boston, Massachusetts, where he is a Software Developer at BookBub. Kevin __owns__ two guitars, but feels it's an insult to the instrument to say he knows how to __play__ the guitar.

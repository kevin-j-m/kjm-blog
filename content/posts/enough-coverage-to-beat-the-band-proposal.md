+++
title = "Enough Coverage To Beat The Band - The Proposal"
date = 2022-08-03T21:26:22-04:00
tags = ["conference-proposal", "conference", "rubyconf"]
summary = "Presented at RubyConf 2020"
description = "The proposal for a talk I gave at RubyConf 2020."
+++

## Abstract

The lights cut out. The crowd roars. It’s time. The band takes the stage. They’ve practiced the songs, particularly the *covers*. They’ve sound checked the *coverage* of the speakers. They know the lighting rig has the proper colored gels *covering* the lamps. They’re nervous, but they’ve got it all __covered__.

Similarly, code coverage can give you confidence before your app performs on production and also tell you how live code is used (or not used). We’ll cover how to leverage ruby’s different coverage measurement techniques in concert to assist your crew and delight your audience.

## Details

This talk will be an explanation of the different modes ruby’s [Coverage](https://ruby-doc.org/stdlib-3.0.0/libdoc/coverage/rdoc/Coverage.html) module provides. It will use the theme of staging a concert as the through line. We’ll progress through the band’s setlist, where each song is one of the modes you can pass to `Coverage.start`.

For each mode, we will examine how it works and what it tells you. We’ll examine the coverage results of code that models performing a concert as example scenarios where that code coverage mode yields actionable insights. While code coverage is frequently talked about in regards to an application’s test suite, this talk will also explore how coverage can be used for instrumenting a running application in production.

### Code Coverage Modes

Each mode answers a different question about the code that was run under coverage:

* Lines - how many times was each line executed?
* Oneshot Lines - which lines were executed?
* Branches - was this code path of a conditional executed?
* Methods - was this method executed?

#### Lines

This is the “classic” implementation of providing coverage. Each relevant line, that is those that aren’t things like empty lines or “end” statements, has a counter that is incremented each time the line is visited in code execution while coverage is running. At the conclusion, you will see how many times each line is executed.

##### Benefits

* This is the default mode for coverage.
* Most of the time, this option will provide you with the information you want.

##### Instructive Example

The band’s guitar technician needs to place an order for guitar strings but needs to know how many to buy. Each guitar is restrung before each show; however, strings can break during the performance. Knowing the correct buffer to purchase is the unknown. We can’t run out of strings, but we also can’t be loading and unloading unnecessary equipment every night.

We already have one big box of strings that should last a while. We’ll track how many strings break throughout the first 10 shows of the tour and use that number to place an order for the rest of the tour. In our code that powers this concert, there is a line in the method to pluck a guitar string that will break the string. We will run coverage over the first 10 shows in production to see how often that line is executed, and know how big of an order to place.

#### Oneshot Lines

Similar to Lines Coverage, this also documents that a relevant line was executed while coverage was running. However, it’s a binary report of whether it was executed or not. It will not tell you how often. This may be sufficient in many cases, and comes with the benefit of being more performant every subsequent time a particular line of code is executed under coverage.

##### Benefits

* Oneshot provides you with nothing more than if a line of application code is executed in a test suite.
* As long as being constrained to knowing if something ran or not, and not knowing how often, is sufficient, Oneshot Lines Coverage provides the same feedback as Lines Coverage with better performance.

##### Instructive Example

This band uses a lot of synthesizers and keyboards in their music, and the different sounds that they can make are stored as “patches”. These are stored in memory on these instruments, and the memory storage is finite. The band has been booked to play a festival, which will be great for exposure to a larger audience. However, because of the number of other acts performing and the shortened time to set up their equipment, they can’t load in and set up all of their synthesizers.

No single synthesizer they have contains enough memory to store all their patches. However, they can’t play these songs without the right patches. It will very obviously sound wrong, both to them and the audience. Whether the sound or effect is used once or a million times, it’s equally vital to have it available for the live show.

To find out what patches they need for this one show, they run through the entire festival setlist beforehand, keeping track of whether a patch was used. After all the songs have been played, any patches that are used will be loaded on the fewest number of synthesizers possible. The other patches can be safely left off the instruments used for the festival.

#### Branches

Branches Coverage tracks execution of different conditional paths and documents how often those different paths are run. The unique benefit that this provides over Line Coverage is in conditionals that execute multiple code paths in a single line, such as ternary statements. You may have a part of that conditional that’s never run or tested, but you wouldn’t know that if you’re relying on Lines Coverage alone.

##### Benefits

* It provides a different frame of reference than Lines Coverage, which ends up being either coarser or more granular than Lines coverage in different situations.
* For conditionals that lay out multiple code paths on a single line, this provides feedback on their individual execution where Lines Coverage only considers whether any part of the line was run.
* When interested in conditionals, and only conditionals, it has less noise than Lines Coverage.

##### Instructive Example

The band is rehearsing a new cover song to add to the setlist; however, something isn’t right. They can’t pinpoint what it is, so they haven’t played the song live for an audience yet, but they keep working at it. Finally, taking a very analytical approach to one of their rehearsals, they find it - one of the vocal effects they use isn't being applied correctly.

This effect is set in a ternary statement in the method that runs the music for each of the choruses for this song. Using Lines Coverage alone, it appears in tests (their rehearsal) that they are getting to that part of the code, or song; however, when tracking Branches Coverage they realize they’re never setting the effect to be the value it should through all the choruses.

#### Methods

Methods Coverage brings the granularity of Lines Coverage up to a coarser grain. Rather than tracking individual lines, it’s concerned with whether a particular method is executed. It can be a 10 line method where the first line is the only line ever executed. Methods Coverage will still consider that as executed the same as a 20 line method where each line is executed.

##### Benefits

* This has a targeted focus to be able to answer a more specific question - is this method executed? - with easier to process feedback.

##### Instructive Example

The lead singer’s main spotlight is cutting out at times during the set, which makes for a less than optimal experience for everyone involved. The engineer’s belief is that many of the venues don’t have the power capacity to handle the needs of the show, and unfortunately, the most obvious spotlight is what’s losing power.

This tour features an extensive stage production with many screens, lighting rigs, and smoke machines. Unfortunately, as the setlist has progressed throughout the tour, the crew isn’t certain how much of it is being used on a night-to-night basis. However, each of those mechanisms are plugged in for every show, drawing power that could be going towards the primary spotlight.

Each lighting rig is triggered by calling a different method in our code that powers our concert. Given that the band has found their groove with a relatively stable setlist from night to night, the crew will instrument the code for an entire show to see which methods are triggered and which aren’t. Those that aren’t can be deleted - which means they can be unplugged, meaning more power for the spotlight.

## Pitch

Code coverage is a topic that often seems to be applied in a narrow sense. In discussions I’m involved in, code coverage tends to mean whether there are tests that execute a given line of application code. This is used as a proxy for code quality, or as a heuristic that can be used to measure the health of a project.

However, I see less discussion on using coverage in production for instrumentation, or using these different modes in any context. Moreover, speaking to my own experience, I didn’t even know there was a Coverage module for a long time as a rubyist, never mind knowing that it provides these multiple modes of tracking and reporting coverage.

My goal with this talk is to expand upon potential use cases of coverage that go beyond measuring how many lines of application code a test suite runs while introducing and explaining the different mechanisms ruby provides to track code coverage.

## Bio

Kevin lives near Boston, where he is a Software Developer at The Gnar Company. He’s doing the best he can to cope with a year of concerts that could have been.

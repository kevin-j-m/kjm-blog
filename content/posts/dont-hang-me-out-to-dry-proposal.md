+++
title = "Don’t Hang Me Out To DRY - The Proposal"
date = 2022-08-03T21:24:22-04:00
tags = ["conference-proposal", "conference", "rubyconf"]
summary = "Presented at RubyConf 2019"
description = "The proposal for a talk I gave at RubyConf 2019."
+++

## Abstract

Close your eyes and imagine the perfect codebase to work on. I bet you’ll say it has complete test coverage. It’s fully-optimized, both in terms of performance and architectural design. And, of course, it contains only DRY code. Surely we can all agree that this is an aspirational situation. But...do we __really__ want that?

Don’t get me wrong; these qualities are all beneficial. However, if we also think we should value everything in moderation, when should we push back on these ideals? What problems can they introduce? Let’s talk about the exceptions to some of the “rules” we all hold dear.

## Details

This talk will not argue against any of the principles of adequate test coverage, optimized code, or DRY code in the broad sense. The intention is not to dissuade the audience from using any of these rules, principles, or guidelines. They’re all incredibly valuable, and we, as craftspeople, are better off for having them overall. This talk will discuss the ways these important ideals can fall short in specific ways and what alternatives are available in those situations.

The talk will initially introduce each of the principles, talking about why it’s important and what benefits it provides. Code examples will be used to illustrate how we can write code or architect software solutions that adhere to these rules. Those examples will then be used to demonstrate some shortcomings of the principle, or situations where following the standard could have negative consequences. Along with identifying some of the downsides, possible solutions and alternatives will be suggested.

## Outline

### Full Test Coverage

#### Shortcomings

* 100% test coverage does not mean all code paths are fully exercised, just that all lines are hit at least once in the execution of the test suite.
* Testing every line of code can have costs that are felt initially in the time to conceive of the tests and the test strategy. However, the larger costs are felt over time if it leads to a long test suite run time, flakey or inconsistent tests, and a large suite that needs to be continually maintained as both the technologies and requirements of the application change.

#### Alternatives

* Consider the value and impact of a test before committing it to the mainline branch of your repository.
* Avoid treating code coverage solely as a vanity metric to be increased and maintained at all costs. Instead, consider trends over a long period or more specific information than the overall coverage number, such as what units are less thoroughly tested and why.
* Listen to your tests when adding features to help guide the design and implementation. Listen to your overall test suite to aid in developer happiness and ergonomics. Is that flakey test pulling its weight? Do your longest-running tests add value commensurate with the time they tack on to every CI run?


### DRY Code

#### Shortcomings

* DRY code may introduce “mystery guests” in the form of variables that you don’t know where they came from, or state that is difficult to understand how it occurred because it’s decoupled from its caller in a way that obscures the work that’s done.
* Not repeating yourself can lead to premature optimizations or abstractions that end up saddling the codebase with a difficult-to-change architecture when it’s discovered that use cases aren’t as similar as initially thought.

#### Alternatives

##### DAMP code (Descriptive and Meaningful Phrases)

This is particularly helpful in tests. If common setup about the state of the world has been extracted into a central location, it may be more difficult to tell the story of the test and understand why the current state is important or relevant to the situation being tested. Repeating this setup can draw attention to what’s relevant in the test and why.

If it’s critical to the value of your test that a struct has an attribute called “foo” which returns “true”, but that struct is defined in a helper method at the very top of the test file, it can be difficult to parse why the test is reacting the way it is. Repeating the definition of that struct closer to the exercise section of your test can improve clarity and understanding.

##### WET code (Write Everything Twice)

Explicitly duplicating functionality or sections of code for additional use, even when you feel confident it’s the same, can have many benefits. If you’re explicitly rewriting it, rather than a copy and paste operation, you can re-explore the implementation and may discover a different solution to the same problem. Even without rewriting it, having the same code multiple places can help you move faster initially without needing to consider the various call sites that a DRY solution may require. It can validate your assumptions regarding just how similar these use cases really are. This could prevent an incorrect abstraction from being chosen, and the two data points can help better inform the design decisions once that third occurrence of the functionality comes up.

### Optimized Code

#### Shortcomings

* Code that is preemptively performance-tuned may or may not be necessary or accurate. Without the data and benchmarking, under load, to illustrate the performance impact, any improvements made in service of performance are done so based on conjecture. This could lead to unnecessary time in developing the believed performance benefit or may even introduce subtle bugs.
* Calling back to choosing the wrong abstraction when focusing on delivering DRY code, working to present the perfect design pattern or architecture to serve the problem at hand can handcuff future feature development while also having an up-front cost to initially introduce the pattern for the first use case.

#### Alternatives

* Ensure the [initial focus](http://wiki.c2.com/?MakeItWorkMakeItRightMakeItFast) is on delivering code that solves the problem. Then layer on performance improvements aided by benchmarks and observations from APM or another instrumentation tooling, should it prove to be called for.
* Certainly reach for design patterns and well-known architectural principles when the opportunity presents itself. But, consider how one might back out of such an introduction should it turn out that the choice was premature - or hold off on introducing it entirely until it’s more clear or there’s sufficient churn in that area of the codebase to warrant the attention.

## Pitch

Quality code is highly valued, but difficult to measure. And though we, as a community, have many ideals, they don’t all fit in every scenario. You may be hard-pressed to find a Ruby developer who might take the position that testing is bad, or DRY code is problematic. Most times, they’d be right not to take that stance. However, writing our code in service of these ideals without considering their implications has costs that may not be explored or understood at the time.

I’ve had the fortune (or misfortune) of seeing the unintended consequences of adherence to some of these rules on a team, codebase, and a project. Many of these principles can be easy to spot and follow, leading to comments that are easy to make in a pull request, changes that are easy to make, and refactorings that feel straightforward and successful enough in that time. The downsides to these changes can be more difficult to identify.

I will share examples of where, particularly in the long-term, following these rules can have negative effects in specific situations, and propose suggestions to address those effects or prevent them from occurring in the first place.

## Bio

Kevin lives near Boston, where he is a Software Developer at The Gnar Company. Though he believes he holds strong opinions, his most frequent response to a question starts with, “it depends.”


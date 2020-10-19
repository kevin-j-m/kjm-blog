+++
title = "Enough Coverage To Beat The Band"
date = 2020-10-19T08:12:10-04:00
tags = ["ruby", "conference", "rubyconf"]
categories = []
summary = "Using Coverage To Stage A Concert Tour"
aliases = ["/coverage"]
+++

## Presentation Resources

Kevin's slides are available for review on SpeakerDeck. The code examples that accompanied the talk are available on [Github](https://github.com/kevin-j-m/ruby_cover_band).

## Code Coverage Modes

Each mode answers a different question about the code that was run under coverage:

* Lines - how many times was each line executed?
* Oneshot Lines - which lines were executed?
* Branches - was this code path of a conditional executed?
* Methods - was this method executed?

### Lines
{{< rawhtml >}}
<p class="float-image coverage-emoji">
🎸
</p>
{{< /rawhtml >}}

This is the “classic” implementation of providing coverage. Each relevant line, that is those that aren’t things like empty lines or “end” statements, has a counter that is incremented each time the line is visited in code execution while coverage is running. At the conclusion, you will see how many times each line is executed.

#### Benefits

* This is the default mode for coverage.
* Most of the time, this option will provide you with the information you want.

### Oneshot Lines

{{< rawhtml >}}
<p class="float-image coverage-emoji">
🎹
</p>
{{< /rawhtml >}}

Similar to Line Coverage, this also documents that a relevant line was executed while coverage was running. However, it’s a binary report of whether it was executed or not. It will not tell you how often. This may be sufficient in many cases, and comes with the benefit of being more performant every subsequent time a particular line of code is executed under coverage.

#### Benefits

* Oneshot provides you with nothing more than if a line of application code is executed in a test suite.
* As long as being constrained to knowing if something ran or not, and not knowing how often, is sufficient, Oneshot Line Coverage provides the same feedback as Line Coverage with better performance.

### Methods

{{< rawhtml >}}
<p class="float-image coverage-emoji">
💡
</p>
{{< /rawhtml >}}

Method Coverage brings the granularity of Line Coverage up to a coarser grain. Rather than tracking individual lines, it’s concerned with whether a particular method is executed. It can be a 10 line method where the first line is the only line ever executed. Method Coverage will still consider that as executed the same as a 20 line method where each line is executed.

#### Benefits

* This has a targeted focus to be able to answer a more specific question - is this method executed? - with easier to process feedback. 

### Branches

{{< rawhtml >}}
<p class="float-image coverage-emoji">
🎤
</p>
{{< /rawhtml >}}

Branch Coverage tracks execution of different conditional paths and documents how often those different paths are run. The unique benefit that this provides over Line Coverage is in conditionals that execute multiple code paths in a single line, such as ternary statements. You may have a part of that conditional that’s never run or tested, but you wouldn’t know that if you’re relying on Line Coverage alone.

#### Benefits

* It provides a different frame of reference than Line Coverage, which ends up being either coarser or more granular than line coverage in different situations.
* For conditionals that lay out multiple code paths on a single line, this provides feedback on their individual execution where Line Coverage only considers whether any part of the line was run.
* When interested in conditionals, and only conditionals, it has less noise than Lines Coverage.

## Presentation Photo Credits

+++
title = "Don’t Hang Me Out To DRY"
date = 2019-12-30T20:57:49-05:00
tags = ["ruby", "conference", "rubyconf"]
categories = []
summary = "RubyConf 2019"
+++

## Abstract

Close your eyes and imagine the perfect codebase to work on. I bet you’ll say it has complete test coverage. It’s fully-optimized, both in terms of performance and architectural design. And, of course, it contains only DRY code. Surely we can all agree that this is an aspirational situation. But...do we __really__ want that?

Don’t get me wrong; these qualities are all beneficial. However, if we also think we should value everything in moderation, when should we push back on these ideals? What problems can they introduce? Let’s talk about the exceptions to some of the “rules” we all hold dear.

## Presentation Resources

{{< rawhtml >}}
<iframe width="560" height="315" src="https://www.youtube.com/embed/b960MApGA7A" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
{{< /rawhtml >}}

* [Video](https://www.youtube.com/watch?v=b960MApGA7A)
* [Slides](https://speakerdeck.com/kevinmurphy/dont-hang-me-out-to-dry)
* [Sample App](https://github.com/kevin-j-m/ivory-tower)
* [Code Examples](https://github.com/kevin-j-m/ivory-tower#code-examples)

## Common Guiding Principles

Full test coverage, DRY code, and optimized code are all incredibly valuable, and we, as craftspeople, are better off for having them overall. But when should we push back on these ideals? What problems can they introduce?

### Code Coverage

Code coverage provides a valuable signal about the extent to which some code is tested, but it is not sufficient for a quality metric. 100% test coverage does not mean all code paths are fully exercised, just that all lines are hit at least once in the execution of the test suite. An application with 100% test coverage can still have bugs and can still have sections of code that don't have all scenarios tested. Additionally, testing every line of code has a cost that is felt initially in the time to conceive of the tests and the test strategy. However, the larger costs are felt over time if it leads to a long test suite run time, flakey or inconsistent tests, and a large suite that needs to be continually maintained as both the technologies and requirements of the application change.

### DRY Code

DRY code helps to ensure you don't have to make a change in multiple places when the way the system should work changes; instead, everything is in the isolated abstraction. However, not repeating yourself can lead to premature optimizations or abstractions that end up saddling the codebase with a difficult-to-change architecture when it’s discovered that use cases aren’t as similar as initially thought. Certainly reach for design patterns and well-known architectural principles when the opportunity presents itself. But, consider how one might back out of such an introduction should it turn out that the choice was premature - or hold off on introducing it entirely until it’s more clear or there’s sufficient churn in that area of the codebase to warrant the attention.

### Performant Code

Performant code is objectively better than non-performant code, right? As always, the answer is, "maybe." Code that is preemptively performance-tuned may or may not be necessary or accurate. Without the data and benchmarking, under load, to illustrate the performance impact, any improvements made in service of performance are done so based on conjecture. This could lead to unnecessary time in developing the believed performance benefit or may even introduce subtle bugs.

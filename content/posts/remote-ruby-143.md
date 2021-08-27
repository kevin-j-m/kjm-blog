+++
title = "Best Practices Are The Best For Whom?"
date = 2021-08-26T19:13:09-04:00
tags = ["ruby", "self-promotion", "podcast"]
summary = "Appearance on the Remote Ruby Podcast"
+++

I had a great time talking to Andrew Mason, Jason Charnes , and Chris Oliver on Remote Ruby. The episode was just released, and you can listen to it [here](https://remoteruby.transistor.fm/143).

There's something bothering me that I feel I didn't say all that clearly. I mention that these rules (test coverage, DRY, and performant code are the examples given in the conversation) feel easy, but are hard to apply in practice.

Who am I saying they're easy for? You may think I'm talking about the people who we ask to use them. And that may be true - it may be easy, or easier, to follow the rule.

However, that's not what gets me worked up about this. Instead, I think it's easy or easier for the people who __say__ the rule, and does a disservice to those who we ask to apply it, unless it's backed up with a lot more support.

## Testing for coverage

My thoughts here apply in a lot of scenarios, but let's use a tangible example. I'll choose test coverage because we spend a lot of the episode talking about it, and metrics in general, and I think it's pretty illustrative for what I want to say.

I like writing tests. They give me fast feedback. They allow me to validate my architecture and design. They give me confidence to make future changes. I'm bought in. Writing tests is part of my workflow. Having a test coverage bar or metric that we track isn't likely to change the way I write code - at least, not in what I consider a materially valuable way. No, the apps I work with don't have 100% test coverage. My goal is to have sufficient coverage that I feel confident making a change and documenting current behavior. That's a lot harder to quantify - but if you know how to quantify confidence - let me know!

But what about the person who doesn't write tests? Mandating tests will be good for the project, right? People who aren't as familiar with writing tests will do so, and the project will be better off because there are tests now, right? Maybe.

## Impacts of best practices as mandates

The person who isn't familiar with or comfortable writing tests likely still has ways that they get confidence that their code works or validates their assumptions or gets feedback on their progress. A new tester isn't going to start getting that comfort from tests solely because they have to write them.

Why is the team writing tests now? Because a computer will yell at them if they don't, and/or their code won't get merged. It's a box to check. Without the training and backing of why testing is important and valuable, and giving people the space to learn, grow, and experiment - they can resent it.

What good are the tests you have then? They made your metric look good, but what are you going to use them for? It's unclear, beyond demonstrating that they exist and giving your CI provider more money if you're billed by minute usage.

Are these tests going to help you catch a bug before it gets into production, or fix one that's already there? Did these tests help inform the choices developers made while writing the code that accompanies the test? Did these tests allow for immediate, small steps with fast feedback?

I don't know - and neither do you probably. Those things are hard to measure. They're also a lot of the value people believe they get out of testing. Capturing or enforcing test coverage metrics in a vacuum does nothing to help inform how much better off you are than if you had no tests at all. It's a proxy measure that's difficult to draw conclusions from without digging deeper.

## Why bother following the rules?

This doesn't mean don't write tests. It can give you insights you didn't have before! Chris Oliver talked about the pay gem, and how different vendors may support features differently, or not at all. Testing can reveal that to you in a quicker/safer way than other explorations. Not only because a test may fail - but because the work Chris does, and things he researched, in order to have that test fail, gives more information about how the vendor's feature works. If you want to know more about that, I have another talk about testing (and designing interactions with) third-party dependencies [here]({{< ref "/railsconf-2020" >}}).

I'm on "team testing". My process of getting there came from a lot of opportunities to learn from testing, experiencing it, and having the chance to be "saved" by them. Forcing tests may increase the likelihood those things happen, but I think you'll have a harder time convincing people on the value of tests if they exist to appease a number.

A test you wish you'd written may have more impact on your career than one you wrote perfectly. The experience of not having that safety net, of not having that immediate feedback, and coming to terms with the consequences of that over the life of a project will stick with you. That doesn't mean don't write tests. But recognize it might not click for someone until they *know* this concept exists, they *know* they didn't use it, and now they *know* how much worse off they are right now without it.

This doesn't mean don't track test coverage. It can be motivating! Jason Charnes mentioned using it to drive an increase in coverage on a project. Once he got the value he sought out of the tests, the metric/number itself didn't matter as much, so he moved on. Jason got the benefit he wanted, and the metric may have helped drive him there, but I'm betting that the benefit wasn't seeing a bigger number on his test coverage %.

## Put in the work

It's easy to say these rules and walk away feeling like we've shared some wisdom. What's harder is putting in the work to help a person grow so they happen upon their own wisdom. We need to have support structures in place so people can learn about these things, and wrestle with them, and gain familiarity. Then they're not rules to be followed - they become shared values we benefit from, and they're used when we see a benefit.

Thanks Andrew, Jason, and Chris for having me on.

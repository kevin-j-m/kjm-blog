+++
title = "How I Read A Pull Request"
date = 2026-01-16T20:04:22-05:00
tags = ["career", "productivity", "testing"]
summary = "A Special News Bulletin"
description = "This describes the process I follow to review a pull request"
+++

An [episode](https://bikeshed.thoughtbot.com/448) of the Bike Shed podcast convinced me to dig this out of my drafts. Thanks Stephanie and Joël.

I read a lot of pull requests (PRs) for work. It's been a huge accelerator for my growth. It helps push work towards production. Work that's in a pull request is likely closer to completion than what I'm working on. Alternatively, the PR may exist because the author is stuck. That also deserves my attention. I hope my reviews help share knowledge and provoke thought from time to time.

I've developed a bit of a system to work through it. I'm going to share that with some help from the [reporters' questions](https://www.thoughtco.com/journalists-questions-5-ws-and-h-1691205). Note that this is not going to focus on how I comment on changes. Only the mechanic I use to read them.

## When?

I tend to review pull requests continuously throughout the day. I often react quickly to notifications for review. I have an unhealthy relationship with my inbox, and I don't recommend mimicking it. However, I do have some more reasonable suggestions for times to review a pull request:

### Starting The Day

This can help you get some momentum to push you into your own work for the day.

### Between Meetings

Do you have that awkward 22 minutes from when one meeting let out until when your next starts? Maybe you're unlikely to get serious traction on some development of your own. But you could dig into some work ready for review.

### Before Lunch

Gift yourself the treat of a satisfying lunch...after you review a pull request from a peer.

### After Lunch

Much like the start of your day, this may help you get back into the swing of looking at code.

### After You Submit A PR

You've just pushed some work into existence that you would like some review on. Help yourself rack up some karma by reviewing someone else's pull request at that time. Now that you've landed that unit of work to review, you're likely looking to context switch anyway. That makes this a time that is hopefully less disruptive to review other code.

### Ending The Day

As long as you're not forced to rush your way through it, this can be a satisfying way to conclude your day. You get a sense of accomplishment and help out a colleague along the way.

## What?

What is the reason for this pull request to exist? If this is intended to merge to production, then I need to be mindful of ALL THE THINGS that entails.

Is this an early draft where the author is looking for some feedback? Then I want to hone in only on their area of concern. For example, unless they want feedback on tests, I'm not going to scrutinize the test coverage. I'm not concerned about style at all. Everything surrounding what they want feedback on may help tell the story. But it's off-limits (to me) from providing notes on. Not that everything needs to be [stellar](https://youtu.be/-nqRkAsZumc?si=dS6HKVUW4fhZPF8-) anyway. But especially the bits that aren't what they want to hear about.

## Why?

I need to check my motivation for reviewing this unit of work. Have I been explicitly asked and tagged as a reviewer? Am I familiar with this area of the code? Do I have some expertise to bring to the table?

Alternatively, am I inserting myself into the situation for my own benefit? Is this a piece of functionality or technology I'm looking to learn more about?

This will affect the type of review comments I leave, but also sets the scene for how I read the PR. I may breeze through something that I'm purely interested in catching up on how the project is going. In this case I have no intent on leaving a single comment or providing a review. Or I may pore over line after line of something I want to learn more about. I may need to remind myself of something I did days or years prior if it's something I worked on in the past. 

## Who?

If this is someone I know well, I can load up their code style or preferences in my head. I try to recall conversations we've already had and remember their positions on them. Once it comes time to leave feedback, I don't want to sound like a broken record.

Even if it's a bot pushing a dependency update I need to consider my audience. The author isn't the only audience. Who else is likely to encounter this PR?

## Where

I organize my focus in a linear fashion that mirrors Github's current tab structure. I start, perhaps predictably, with the title and description. I prefer the description to include information about why this change is being made. If they considered alternatives, having them listed here is great. Should there be areas of concern that require scrutiny, call them out here. Ultimately these are the same things I look for in a commit message. I use this to set the stage for what I read and how.

I try to avoid the existing comments on the PR until just before I submit a review. Unless the reason I'm drawn to this pull request is because I was tagged in a specific comment.

I then look briefly at the commits. Even if a PR author recommends to read this commit by commit, I will admit that I rarely do it. I do look at the commits to see if it can raise awareness about any structure or order for how the work came together.

I spend most of my time reviewing the files changed. Surprise!

## How?

Now that I'm looking at the contents of the files changed, I start by...not really reading the changes. Instead, I scroll from top to bottom. The alphabetical file arrangement is pretty arbitrary for code organization. I'm scrolling to get a sense of the general shape. I'm reading the file names affected. I'm seeing where the bulk of the work exists. I'm basically doing the [squint test](https://youtu.be/8bZh5LMaSmE?si=4Tvs7GUqSEsebB-G&t=312) on the changes.

When scrolling through tests, I may read the description and not much else during this pass.

### Test-Driven Review

After having scrolled to the bottom, in ruby projects, I now tend to be exactly where I want to be. You see, much like you may write tests first (or you've heard some people do) I tend to READ tests first. In ruby projects, these tests are going to be in the `test` or `spec` directories. Those are likely towards the bottom of the alphabetical sort.

I read the tests as an indication of where to focus. Areas where there are tests can signal things that the author thought were important. Areas with a lot of tests could mean this is *really* important. It could also mean it's *really* confusing, or was difficult for the author. In which case, they wrote a lot of tests to make sure it works.

This is not a guarantee. Maybe there are no tests for a crucial area of the code. And the reasons for that could be many. By reading the tests first, I've started to inform my brain about what I expect the implementation to look like. If I find part of the implementation that's a surprise - that's also signal in my reading. Why isn't there a test for this? Did I miss it? Should there be one?

I'm not encumbered with knowing the implementation that may justify the test's complexity. If a test has lengthy setup, I have no supporting code to rationalize it. Instead, I see the complicated setup alone. The same goes for heavy mocking and stubbing. None of these things are necessarily barriers to merging. However, I can spot them more easily without the burden of knowledge to explain why they might be there. I can spot it as a potential discussion point. There may be changes we can make to the test. This may highlight potential opportunities for a different design.

### Implementation

Once I've looked at the tests, I return to the implementation in detail. As mentioned, I can use the knowledge from reading the tests to guess generally how the code looks.

I tend to look at the implementation alphabetically. Unless there are tests that seem particularly confusing or interesting to me. Then I may hop right into that area of the implementation to learn more. Otherwise, I read alphabetically from top to bottom. It makes it easier for me to know that I have read everything in detail. I know I can mark files as viewed in Github. For whatever reason, I don't find myself frequently using that feature.

### Big Picture Focus

Once I've read all of the changes, I go back to the description to ensure it's meeting the goals outlined there. This is also when I'll look at existing comments and discussion.

If the changes are large enough, I consider taking another scroll. In this pass, I'm thinking about the overall structure and organization. Are there idioms or design patterns we should consider? Have we done something similarly in this application or another that we can learn from? These are the questions I'm asking myself as I read the code for this last pass.

## Why Not?

I will admit, I do not employ this thorough approach every time. Not because I'm lazy. Well, not only because I'm lazy. Sometimes the change doesn't warrant it. Sometimes the approach doesn't apply.

I may be reviewing a one line change to fix a nasty bug that took days for the author to resolve. That shouldn't require the scrutiny I've placed here on the files changed. However, I expect to be spending a lot of time reading the description or commits. I need to understand the context for the change. I may be reading documentation or source code of a related dependency. The approach changes based on the needs of the code change itself.

## How About You?

Now that you've read about my process, consider how you read a code change. Maybe you have a standard approach and you're not even aware of it. Perform some investigative journalism on your own tools and methods that you use. Think about why you do things the way you do and the value it adds. Report back so we can all learn from you!

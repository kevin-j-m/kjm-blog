+++
title = "Core Competency: A Developer's Business"
date = 2022-02-14T07:19:33-05:00
tags = ["business", "soft-skills", "leadership", "management", "productivity", "career"]
summary = "What specific qualities drive your team's success?"
description = "Core competencies are unique capabilities that a firm delivers. Developers can consider their group's core competencies to make a build vs. buy vs. borrow decision."
+++

## A Developer's Business

1. [Situational Leadership]({{< ref "/situational-leadership" >}})
2. [Competitive Advantage]({{< ref "/competitive-advantage" >}})
3. __Core Competency__
4. [Business Process Management]({{< ref "/business-process-management" >}})

Knowing a thing or two about the mindset of someone on the "business" side of your company can be very valuable. That doesn't mean you need to spend two years of your life getting a [MBA]({{< ref "/engineering-mba" >}}) to learn about this (but don't let me stop you).

This post won't be about accounting, marketing, entrepreneurship, or making quarterly revenue projections. Instead, it will focus on a concept I learned in business school that helps me in my day-to-day as a developer.

## Core Competency

A firm's [core competencies](https://link.springer.com/chapter/10.1007/978-3-662-41482-8_46) are the specific qualities or characteristics driving its success. A core competency must meet the following criteria:

* Provides access to various markets.
* Contributes to perceived customer benefit.
* Difficult for competitors to imitate.

This is [often](https://www.techtarget.com/searchcio/definition/core-competency#:~:text=what%20it%20can%20do%20better%20than%20any%20other) referred [to](https://www.investopedia.com/terms/c/core_competencies.asp#:~:text=what%20it%20can%20do%20better%20than%20anyone%20else) as what the organization can do better than any other group. After identifying a core competency, invest in it to achieve and keep that advantage. Focus more on improving core competencies and less on other areas of the business.

## Developing Core Competencies

Knowing your company's core competencies helps to answer your next build or buy or borrow conundrum. Developers face this when deciding *how* to build a feature.

We'll consider upcoming features from our [previously mentioned]({{< ref "/competitive-advantage#competitive-advantage" >}}) blogging platform for medical professionals. Keeping an eye on our core competencies, let's decide how to deliver changes to our comment system.

## Build

Our last article introduced a comment moderation system to automatically flag HIPAA violations. We delivered that knowing we're [focusing]({{< ref "/competitive-advantage#focus" >}}) on the medical market. That feature allows us to stand out in the marketplace. The innovative functionality can be a core competency of the firm. As such, we should be building all the work related to that feature in-house.

Our developers need a keen understanding of HIPAA violations, or how to identify them. Anything we can invest to build up that knowledge we should. It has immense value by making it available to customers through our product. Developers improving the functionality of HIPAA violation identification is of critical importance. The more time they can spend focusing on that, the better this unique capability will be.

## Buy

We won't get far only having __one__ feature in our comment system. To drive re-engagement, we want to send emails to authors when someone comments on a post. We need a way to *send* these emails to the authors. But, we want our developers focusing on HIPAA violations, not email delivery.

Email delivery is not a core competency of the company. We may not even want to know what a SMTP server is, never mind how to configure it. Luckily, there are other companies who *do* have that as a core competency. We can engage in commerce! In exchange for some of our money, we can enjoy their knowledge of and expertise in email delivery.

We're best served paying a vendor to handle email delivery for us. We'll access this capability from their API in our application. Their price is significantly less than what it would cost to build and maintain this functionality ourselves. Even if not, the opportunity cost of developers working on email delivery may be too great. That's time they're not improving the identification of HIPAA violations! We should buy this functionality. Our customers will receive emails, and our team can focus on our core competencies.

## Borrow

Blog posts on our platform are likely to be the talk of the town. We'll generate a lot of conversation in the form of comments. Loading all those comments on initial page load will negatively affect page performance. Pages that are slow to load will be slow to generate further conversation and engagement. To mitigate the performance issue, we're going to paginate the comments. A subset of comments will load at first, and readers will be able to request more comments by pressing a button.

Comment pagination is not a core competency of our organization. Even though it's something we need to do, we're not in the business of paginating comments. As such, like we learned from the last section, we should look to buy this capability. It turns out, there aren't a lot of other companies that are in the business of paginating comments. Even if we wanted to give someone money to do this for us, there don't seem to be many takers. Fortunately, as developers, we have another option.

We can use the kind work of open source developers who maintain pagination libraries. That will allow us to add this capability to our application with minimal work on our end. From a developer's perspective, more considerations must go into [adding a dependency](https://www.mikeperham.com/2016/02/09/kill-your-dependencies/). The same is true whether buying capabilities or pulling in a library. Strictly considering our core competency, adding the pagination library is a justifiable choice. Any opportunity we have to add capabilities while focusing on HIPAA violations is a win.

## Core Lesson

A company's core competencies help it stand out in the marketplace. The same is true for you. What unique talents and skills do you bring to bear? How can you use them so that a company will reward you with a new job, title, or raise? Think about what your awesome qualities are and how you can leverage them.

For the final lesson in our series, we'll discuss [business process management]({{< ref "/business-process-management" >}}).

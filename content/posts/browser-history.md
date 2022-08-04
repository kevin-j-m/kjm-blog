+++
title = "Browser History Confessional: Searching My Recent Searches"
date = 2022-04-13T08:55:10-04:00
tags = ["ruby", "conference", "railsconf"]
categories = []
summary = "Searching for my latest conference talk"
description = "Presentation resources for a talk about why we search for help in our day-to-day lives as software developers."
aliases = ["/browser-history", "/alluvial"]
+++

## Abstract

We all only have so much working memory available in our brains. Developers may joke about spending their day composing search engine queries. The reason it's a joke is because of the truth behind it. Search-driven development is a reality.

Join me, and my actual search history, on a journey to solve recent challenges I faced. I'll categorize the different types of information I often search for. You'll leave with tips on retrieving the knowledge you need for your next bug, feature, or pull request.

## Presentation Resources

{{< rawhtml class="float" >}}
<iframe width="560" height="315" src="https://www.youtube.com/embed/R7LkHjJdH9o" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
{{< /rawhtml >}}

* [Video](https://youtu.be/R7LkHjJdH9o)
* [Slides](https://speakerdeck.com/kevinmurphy/browser-history-confessional-searching-my-recent-searches)
* [Code Examples](https://github.com/kevin-j-m/browser-history)
* [Blog Post]({{< ref "/searching-for-a-reason" >}})
* [Proposal]({{< ref "/browser-history-confessional-proposal" >}})

## Search-Driven Development

I find it's important to consider *why* I'm searching for something. It helps guide the path I'll take to resolve my query and helps frame the problem in a familiar context. In my own work, I've found there are four main categories of reasons why I search. They all relate to how much experience I have with the problem I'm working on.

### Solve Direct Problems

When I'm trying to do something for the first time, I may lack any context and perspective to have a starting point. I may get stuck mid-way through. And, being a novice at this task, I need to seek outside perspectives. I'll often build that up by searching for how others have accomplished the same task.

With a specific error that I'm looking to fix, there's one thing I almost always do first. I __copy and paste the error__ into a search engine. This is a high-leverage, low-effort move. It's unlikely you're the only person to have encountered this issue before. It requires validating that what you find are people experiencing the same problem. You must also determine for yourself if their solution works for you. However, it is still a go-to first step for me.

That may not be a great approach if the error is fairly generic. It may not yield results if a lot of the error message is about code specific to your application. In that case, I review the error message, and consider the context where it arived. Search the README of the framework, dependency, or tool you're working with. Visit open and closed issues. Search their documentation. Find their forums. Use log messages to add to your search criteria.

You may conduct a thorough search using all these avenues and still not have an answer. You might start feeling frustrated, or angry. You might be doubting your abilities. When that creeps in, the most important thing I can recommend is that you stop.

__Stop.__

Move away and do something else for a few minutes, even if it's get a new glass of water. Give your brain some time to wander. Ponder what you're experiencing in a different physical location. Do something else to take your mind off of it.

When you're ready to regroup, re-review what the problem is and what steps you've taken to address it. Look at your search history to see what you've already looked for. Try to find any gaps in what you've already explored.

At this point, remember that you don't have to solve this alone. You don't need to use the computer by yourself to solve the problem. If the computer could fix itself, we wouldn't have a job as developers! Reach out to your friends and your coworkers for help. Their fresh perspective may bring a quick resolution. They might suggestion other things to search for. They may help you with a different strategy to take.

### Recall Details

Another class of problem is one I've encountered before, but I don't remember exactly what to do about it. This doesn't mean that I'm starting from a fresh search though. I do my best to keep some notes, or a pointer in my brain, so I can have a very targeted search and quickly resolve my issue.

I don't have the characters memorized that tell a date how to format itself into a specific string. But, I do know that the [ruby strftime documentation](https://ruby-doc.org/stdlib-3.1.0/libdoc/date/rdoc/DateTime.html#method-i-strftime) has them exhaustively defined. When I need to format a date in a specific way, I don't remember how to do that. But I do remember where to go to find that out.

### Revisit Assumptions

I also find myself searching when I already *know* a way to solve my problem. That may sound wasteful, but it's been beneficial for me. I take a time-boxed approach to learn how other people solve familiar problems. It gives me new insight. It may introduce me to a new tool. I could learn a new design pattern. I may discover something I like about my approach that I didn't even have an appreciation for before.

We work in an industry that's ever-changing. There are new tools, products, and mindsets coming and going. We can learn from all these. I may still use the approach I was going to when I started. But I'm doing so not __only__ because it's how I know how to do it. It's because I've identified a set of alternatives. I've developed a series of criteria to evaluate them on, and made an intentional choice on how to proceed.

### Share Communal Knowledge

I'll also search for things I already know about to provide that information to others. I don't want people to only take my word for it. I want to provide some social proof, documentation, or other primary source. Now this person has that available to them for bookmarking or their notes. It can serve its purpose beyond the scope of a conversation with me.

It's very unlikely that I learned about what we're talking about by figuring it out on my own. I likely read, watched, or heard about it somewhere else. That original source deserves the credit. Sharing that with others allows them to gain the same context I have and recall it in the future.

## Search Experience

When I'm solving a direct problem or recalling details, I search when I feel that I cannot resolve my issue alone. I've determined it'll be a quicker and more effective use of my time to seek outside perspectives.

There are many times where I search for items that I do know the answer to. I investigate new ways to solve the problem. Through that I gain more experience and make informed choices on the approach I take. I search to bolster my presentation of knowledge I have with information from others. Revising this information is beneficial to me, and by sharing it, I hope others benefit as well.

The next time you're searching for something, consider *why* you're searching for it. Use that answer to inform the approach you take. And I hope that you find what you're looking for.

+++
title = "Browser History Confessional: Searching My Recent Searches - The Proposal"
date = 2022-08-03T21:28:22-04:00
tags = ["conference-proposal", "conference", "railsconf"]
summary = "Presented at RailsConf 2022"
description = "The proposal for a talk I gave at RailsConf 2022."
+++

## Abstract

We all only have so much working memory available in our brains. Developers may joke about spending their day composing search engine queries. The reason it's a joke is because of the truth behind it. Search-driven development is a reality.

Join me, and my actual search history, on a journey to solve recent challenges I faced. I'll categorize the different types of information I often search for. You'll leave with tips on retrieving the knowledge you need for your next bug, feature, or pull request.

## Details

The situations where I reach for external knowledge are generally one of the following:

### Syntax and Implementation Details

Do you remember how to format a date as a string in a specific convention? I sure don't! But if I need to do so, I revisit the documentation for [strftime](https://apidock.com/ruby/DateTime/strftime). It includes a great list of directives to reference.

Even if I'm pretty sure I remember the correct directives, I still take a moment to review this list. There may be a shorthand or convention that I can use to achieve the format I need.

ActiveRecord's API surface is large, and includes many methods that look to do the same thing. Even though they may give you the same data, they may have different side-effects. One of these side-effects is in performance. Reviewing posts like Nate Berkopec's "[3 ActiveRecord Mistakes That Slow Down Rails Apps: Count, Where and Present](https://www.speedshop.co/2019/01/10/three-activerecord-mistakes.html)" can remind me, in the moment, of the trade-offs. Re-reading it forces me to consider what my situation demands, and what will be most performant.

I don't have the capacity (or interest) in committing this information to memory. But I do know exactly where I need to go to find it. It's not an introductory search each time - I know exactly what I'm looking for __and__ where to get the answers I need. I can't hold the state of the world in my head, but I can keep breadcrumbs or bookmarks to know how to retrieve the data I need.

### Solving Direct Problems

I recently got a new computer for work. It's the first time I've had to set up a computer in four years. I installed everything I thought I needed, cloned a repository, and bundled the application. It didn't succeed. In the backtrace, I saw:

```
fatal error: 'libpq-fe.h' file not found
```

I did not have the first clue how to fix that. But I did trust that I wasn't the only person to ever see that. So, I did the most productive next step I could think of. I copied the error, exactly as written, and pasted it into a search engine. It turns out that the pg gem has a much easier time installing on a computer that has postgres installed. One search result clicked on, one command issued to my package manager, and I was on my way to the next problem, never to consider "libpq-fe.h" again.

The story of these errors isn't always so neatly resolved. However, I'm hard-pressed to consider a next step with a higher success rate. Searching the internet for the *exact* error message can be a life saver! We'll talk about where to source this information from, such as:

* The project's issue tracker.
* The project's published mailing list.
* A discussion forum. 
* A trusty generic search query.

Now with some answers, we need to determine which solutions might actually work. We'll keep track of different things we've tried so we don't find ourselves going in circles. We'll practice stepping away to take a break. We'll even consider reaching out to an actual human being for another perspective.

### Revisiting Assumptions and Approaches

For help with identifying N+1 queries, I reach for the [bullet](https://github.com/flyerhzm/bullet) gem. Like any tool, it's not perfect, has its particular quirks, and has a learning curve. But, I'm comfortable with it and mostly know how to fight with it, if I need to. That familiarity carries a lot of value, but must not be the only consideration when reaching for a tool. By taking a few minutes to see the current state of the world for evaluating N+1 queries, I may discover the new(er) [n_plus_one_control](https://github.com/palkan/n_plus_one_control) gem.

There are many tools, many ways, and many approaches to solving problems. Reaching for what you know can have immediate efficiencies and comfort. Our world as developers is rapidly-changing. It's important to keep an eye out for any new (to you) philosophies or approaches. This doesn't mean chasing "the new hotness". It does suggest awareness of what's possible, what exists, and what warrants further evaluation. Now may not be the time to introduce that new tool or that new design pattern. That five minute investigation may pay off months later when you think, "what was that thing I was looking at a while ago? That will apply perfectly right now."

### Sharing Communal Knowledge

Calling `update_all` won't trigger ActiveRecord [callbacks](https://apidock.com/rails/ActiveRecord/Base/update_all/class), and that includes the `updated_at` timestamp. RSpec includes a `have_enqueued_job` [matcher](https://relishapp.com/rspec/rspec-rails/docs/matchers/have-enqueued-job-matcher) that can verify you enqueued ActiveJob jobs. One of Sandi Metz's [rules](https://thoughtbot.com/blog/sandi-metz-rules-for-developers) is to have methods that are no longer than five lines. Not to brag, but these are things that I happen to know.

Why am I searching for them then? I want to share that information with others, and provide original sources for them. This often arises in code review. I could recommend using `have_enqueued_job` in a spec. If the author hasn't heard of that before, I'm now putting *more* work on them. I should bolster that by adding a link to the documentation in my comment, along with a code snippet. I'm giving them a path forward, and a reference to learn more the implementation details.

This may become one of their items that they never remember the details of. Much like me and `strftime`. But now they know where to go each time to get that information. I've benefited from this kindness of reviewers in my past.

## Pitch

Stumbling through __real__ searches I've made, people may feel better about their own ability to solve problems. We can't always do it alone; we need to seek external resources to help some times. At the least, we can commiserate on how we find ourselves looking up the same things every time. We may all have a different "thing" we continually look up, but I'm convinced we all have at least one.

Beyond the awkward goal of embarrassing myself so others can relate, there is the real truth that developers need to search for things. A lot. The search methodology I use, and the reasons I'm searching at all, should help others in their next search.

## Bio

Kevin lives near Boston, Massachusetts, where he is a Software Developer.

Consulting a search engine for Kevin Murphy will show luxury hair care products, an actor, and a University of Chicago Economist. None of that information has much to do with me, unfortunately.

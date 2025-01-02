+++
title = "Working Iteratively"
date = 2025-01-01T14:04:22-05:00
tags = ["career", "productivity"]
summary = "Taking it one day, or one hour, at a time"
description = "This discusses how I like to break up work to satisfy my brain."
+++

I like to plan, design, work on, review, and ship small units of work. The definition of "small" can vary. I like to do this for me. I do believe that it has advantages for the team. I will freely admit that my motivation is to satisfy how my brain works best. Let's explore what that means for me.

## Daily Focus

I like to have an answer to the question, "what's your one main focus for the day?" The key word there is one. That doesn't mean it's the only thing that's going to get done today. However, I want to identify what the item is that is going to give me satisfaction that I've done my job today.

And then I want to break that down even further and find the piece of that I can deliver. Shipping an entire feature every day is likely not sustainable. Even if it is, I want to sequence the steps to get there.

This blog post is one example of that. I didn't say I wanted to write a blog post. I said I was going to write about working iteratively. But that was too big, so I identified the areas of that which were meaningful to me. That became my outline, and you can see them as the headings of this post.

## Interrupt-Driven Development

Identifying those steps gives me breaks. It provides pressure release points. I can feel more comfortable about context switching. I know I'm going to get interrupted throughout the day. It may be by planned meetings. It may be someone asking for help. It may be my inbox piling up and my brain feeling the need to address it. It may be my body telling me it needs to move and walk around the house.

I've built small, discrete delivery points towards the larger goal I have for the day or week. As a result, I give myself more permission to lean into those switches. When I have those forced upon me, like a production fire, coming back is easier. When I'm able to transition back, I have less to load up in my working memory. It's easier for me to get back into. I'm not trying to draw an [entire owl](https://www.pinterest.com/pin/220465344245182399/). I can focus on one of the two circles to start.

{{< figure src="/img/owl.jpg" class="mid" alt="A sketch of two circles followed by a fully-drawn and detailed owl" >}}

Again, with this blog post, I gave myself permission to wander after finishing a section. Maybe that was checking my RSS feeds. Getting some water. Just staring at the wall for a few minutes. Whatever it was I wanted or needed to do, I built in places where it was expected to do that.

## Momentum

Working in these discrete steps helps me feel more accomplished. It's a bit of personal gamification. Quantity does not equate to quality. But when I'm not delivering value I can perceive on a regular cadence, I start feeling stuck. Shipping something can start making me feel like what I'm doing is on the right path. It's possible. I'm one step closer, given the work I just finished. Now I can move on to the next step, which doesn't feel so daunting. After all, I just did the last piece. Maybe I'm even *excited* to see where it goes from here.

It also has the external benefit of people seeing the work. Or having it explained to them. In a stand-up update, I'm not, "working on xyz feature." I'm, "writing the database migrations to allow us to store the data needed for the xyz feature. After that I'll start capturing it when users do the thing."

## Stacked Diffs

One thing that varies with this approach is when I present the work to others. I may be iteratively working on three branches, each of which builds off the last. However, I may not submit any of those branches for review until the entire work is done. Or, I may open a PR for the first branch immediately, but hold off on the second and third until it's all done.

This is a delicate balancing act. For me it depends on if the work is understandable on its own and also if it adds value on its own. That is a judgement call. I don't always get it right. My tastes may not align with the preferences of my team. As I build a working relationship with them, I calibrate my delivery to align with their needs.

You may be reading this and be thinking, "what you want is commits Kevin. Each of these is a different commit. Not a branch." And you MAY be right. Again, this is all done to taste. But sometimes I want to draw attention to a very specific part of the work. And the best way to do that, I've found, can be to very deliberately have that be the sole focus of the code review. Not something you can roll up or ignore with a set of other changes. Maybe it does end up shipping only as part of a larger deliverable. But I want it *reviewed* separately.

I'm also colored by my own perspective with reading code reviews. I don't often review commit by commit, even when that's asked for. I may also want these items of work to ship independently. I may want them to have their own commit message and justification. If you happen to be working in a squash and merge shop, separate commits don't end up that way once you merge. I can have three PRs, the branches of which merge into each other. I can merge them starting at the one targeted against the main branch of the repo.  And work my way out towards the end.

## Pulling Out Incidental Finds

Even though I may set out with a plan, things don't always neatly follow that plan. Surprises unveil themselves. I may uncover an unknown bug in the related code. I may want more tests for existing code to better understand how to develop my feature. Maybe I stumble across that code comment that exists in every sufficiently-old code base:

```ruby
# Please delete this hack by <SOME DATE THAT WAS YEARS AGO
# OR CONDITION THAT HAS BEEN TRUE FOR A LONG TIME>.
```

I want to address those in service of completing my work. However, it's not part of my work. I may even do it as part of completing the current step of my iterative development. However, before submitting it for review, I separate it out as a different unit of work. The tests that didn't exist before can ship and live without my new feature. And my new feature can work without them. They don't need to be bundled. Reviewers don't need to figure out how they're related.

If you're writing a commit message and write a new paragraph starting with something like, "This change also...," stop. Ask yourself if that can be a separate deliverable. This can also be framing you can bring to a pairing session when you're not driving. When you see someone else doing it, you can call attention to it. Agree as a group whether to go down the path or not together at this time. But if you do, consider introducing it separately.

## Looping Back Around

So, now that you've read this, what's your one main focus for the rest of the day? Or tomorrow? And how can you cut that in half? What would it look like if you cut it in half again? Think about that until it feels small enough to be uncomfortable and then think about one more split.

You're not committing to delivering the work in that fashion. Try it to see if the exercise brings some clarity to your focus. Check in with yourself to see if it makes progress feel more attainable. Ask if it gives you comfortable break points in your day. If it doesn't, take what you learned from it and try something different tomorrow. You know, iterate.

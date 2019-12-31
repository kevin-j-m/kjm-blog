+++
title = "Coding for Fun (Not Profit)"
date = 2019-12-31T08:56:04-05:00
tags = []
categories = []
+++

I originally wanted to call this post "Coding for Fun ~~And Profit~~" with And
Profit in strikethrough, but hugo wouldn't recognize markdown in the title of a
new post, so I gave up immediately and renamed it, which is a perfect
encapsulation of what I'm looking to convey here.

## Background

I've been paid to write code for almost 15 years now. I think I'm decent at it,
and I enjoy doing it as a profession, but it's not something I do much of in my
free time. I'm not against people having side projects, spending their free
time how they'd like and doing whatever energizes them, but I write enough code
during the day at work to satisfy my interests there. I'll never get a job
somewhere that requires an extensive personal portfolio to show as proof of
"passion". I'm comfortable with that, and privileged enough to be able to say
that.

Before this, I've also never had a personal website - ever. Sure, I maintain
various profiles on different social networks, but I've never had a place of my
own on the internet. I'm also probably the only Rails developer who's never
typed `git push heroku` in a terminal, but that's a different story.

## Coding For Work

I'm a pretty methodical person in general, and I take that with me to work. It
comes in handy in this business, but as with everything, it has a cost. It takes
time and effort, but my belief is that the value for my employer or client
outweighs that cost. Some of these items include:

* Requirements vetting
* Alternatives analysis
* Automated testing, whether through TDD or not
* Code review
* Maintenance and upkeep of dependencies
* Clean commit history throughout the project

Because the results of my actions affect the overall product, my team members,
and all of my business partners, I'm careful and deliberate in the work that I
do.

## Coding For Me

I decided to stand this site up because I had some content I wanted to maintain,
and I had the blessing of my employer to do so. I needed something quick and
fast that I wouldn't need to put a lot of effort into maintaining.

I also wanted to try out something new, so knowing I only needed a static site
generator, rather than reaching for my comfortable [Middleman](https://middlemanapp.com/)
tool, I instead gave [hugo](https://gohugo.io/) a try. After all, it is:

> The world's fastest framework for building websites

In doing so, I of course ran into some hiccups and problems, but what I'll talk
about here is the process, and how this was different than how I regularly work.

### What Goes

#### Alternatives & Upfront Analysis

I went with hugo on name recognition only. I didn't even identify the core
feature set I needed or didn't need; I just dove right in. Similarly, I could have
arduously reviewed the vast number of [themes](https://themes.gohugo.io/) in
their gallery to find the perfect one, or customized my own. Instead, I chose
three that seemed "fine" above the fold, and picked the one that I liked best
after a one minute review.

#### Focus

When I work from home, I do so from my office, with the door closed. I control
the environment and am particular about what noises or stimuli are present.
However, this is just for me, so instead I sat down on the couch next to my wife
while she was watching something on TV. It's ok if it takes a bit longer, or I'm
a bit distracted, as long as I'm making that trade for me and me alone.

### What Stays

#### Verification

Testing brings comfort to me and I dislike working without tests. It helps guide
my initial implementation, gives safety for future refactors, and provides
confidence in delivering functionality. And most of the work that I like doing
the most, and as such gravitate towards, rarely has a direct user-facing
component.

However, that's __all__ that this site is. I don't want to spend time futzing
with configurations that I don't need to, or meddling with the theme to eek out
performance improvements, or evaluating updating, changing, or removing
dependencies. But I still need to make sure it works. I spent a ton more time in
the browser than I typically do developing for work, but it's the same process:

* Identify a thin slice of functionality
* Focus on implementing it
* Document other observations for future work
* Iterate to the next thin slice
* Deliver

The mechanism by which I worked that process changed (refreshing the browser vs.
running a test), but the process itself stayed the same.

#### Documentation

Sure, my commit messages are definitely not to the degree that I would expect,
hope, or want them to be. However, that doesn't mean that they're worthless.
Particularly on these projects where I'm moving fast, their value is more as
"save points" than documentation (and I'm not cleaning them up prior to moving
them into a mainline branch), but they can still provide valuable information on
*why* a change was made.

Additionally, particularly because this is a project that I'm likely to be
revisiting infrequently, I made sure to take the time to add a README that
includes quick examples of all the things I'm going to want to do. For this
project, that includes:

* Creating a new post.
* Running the server locally.
* Deploying the changes (which there's a script for).

*Because* it's something I won't touch day-to-day, it's even __more__ important
to take the few minutes to document those quick commands to save myself the time
of needing to search for it every time that I want to edit this.

### What Emerges

#### Quick Assessment of "Good Enough"

There's always an ideal version of what you're looking to accomplish. That
likely manifests itself both in the code itself, as well as the user-facing
product. And we all want to hit that ideal; however, it's not always feasible.
And while I work diligently to reach that at work, I also do my best to be
pragmatic about at least explaining the implications of getting to that ideal,
presenting that to the decision-makers, and working with them to come to an
agreement on how to proceed.

However, in this example, my goal was speed and speed alone. For example, I had
a post to add which had some images that I wanted formatted just so. I couldn't
figure out exactly how to position or size them to my liking. I think from
searching for about 30 minutes I could have gotten there by having the images as
[page resources](https://gohugo.io/content-management/page-resources/), then using
hugo's [image processing](https://gohugo.io/content-management/image-processing/),
and having those available in a [shortcode](https://gohugo.io/content-management/shortcodes/)
that I reference in my post.

I could have done that, but instead - I deleted the images. They weren't
tremendously important to the post. They certainly made things look nicer, but
they didn't provide any explanatory value. So I removed them, and freed myself
of the problem. And the page doesn't look exactly how I wanted it to, but it's
done. And if I feel the need to scratch that itch in the future, I know where to
start.

Remember the context under which you're developing, who the decision-makers are,
and what choices you should make based on that. But work to know what you
__aren't__ willing to give up in your process, regardless of the context.

+++
title = "RubyConf 2023 Recap"
date = 2023-11-17T21:00:24-04:00
tags = ["ruby", "conference"]
summary = "An attendee's perspective"
description = "A summary of my time at RubyConf 2023"
+++

## RubyConf 2023

RubyConf recently wrapped up in San Diego, California. This post is meant to highlight the great work from all involved. I hope you'll seek out the full videos of all the sessions that interest you once they are available. Unfortunately, I couldn't be everywhere, so this covers what I saw.

## Preparing

If you'd prefer, you can [skip]({{< ref "#community-day" >}}) right to my commentary on the start of the conference.

This is a weird brag, but this is the first conference I've attended in a long time as an attendee. It's been a while since I haven't been a speaker or on the speaker waiting list. Being a wait-listed speaker means you prep and have a talk ready. You only get to give it if you need to substitute for another speaker on short notice.

This left me with a lot more free time during the conference. More importantly, I had more time before the conference. I still had the "conference season" itch though. So I channeled that energy in a few different directions.

### CFP Coaching

First, just as I did for RailsConf, I joined Ruby Central members for the CFP coaching sessions. We helped others with their proposals in small groups. This was a great opportunity to meet with prospective speakers and workshop ideas to submit for the conference. I'm glad that Ruby Central has started organizing these events. I hope they continue.

### Podcasting about RubyConf

After the CFP process wound down, I joined Brittany Martin as a temporary [co-host]({{< ref "ruby-on-rails-podcast-cohost" >}}) of The Ruby On Rails podcast. We talked with Allison McMillan and Chelsea Kaufman about the plan for RubyConf 2023. I hope that it helped get people excited about the conference. Even better if it helped convince someone to come after listening.

I also joined Julie and Andrew on the [Ruby for All]({{< ref "ruby-for-all-guest-2023" >}}) podcast. The discussion centered around conference speaking.

### Writing

I expanded on my blog post from last year where I shared [past proposals]({{< ref "sharing-past-conference-proposals" >}}). I turned it into a series on preparing for conferences. I wrote about [my process]({{< ref "building-conference-talk-content" >}}) for turning an accepted proposal into a talk. I followed that up with [how I practice]({{< ref "preparing-conference-talk-delivery" >}}) and prepare for the stage.

### Helping Speakers

Lastly, I had more time to help friends who were speaking with their talk preparation. Notably, I was a speaker mentor for the first time. I've been hesitant to do it in the past when I've had a talk of my own to prepare. I was really happy to have the time to devote to this. I paired with [Paul Reece](https://ruby.social/@paulreece_). Paul gave a great talk about [Ruby Polars](https://rubyconf-2023.sessionize.com/session/530118). Paul was very prepared and definitely made the process easy for me. It was super fun to see the evolution of the talk over time from an outsider's perspective.

## Community Day

Even after talking about this on [The Ruby on Rails podcast]({{< ref "ruby-on-rails-podcast-cohost" >}}), I'll admit that I was still pretty unsure how this would all go. I didn't attend any of the workshops as I wanted to see the implementation of the hack space events.

After an introduction from Allison McMillan and Chelsea Kaufman, the group separated into workshops and the hack space. I spent my day in the hack space. In the morning I spent time at the Hanami table. I'll be honest, I didn't do much of any Hanami work, but I had a great conversation there.

After lunch, I parked at the Ruby LSP table. I worked with [Julie](https://codewithjulie.com/) and [Drew](https://www.drbragg.dev/) on [HEREDOC endings](https://github.com/Shopify/ruby-lsp/pull/1181). We also found an [issue](https://github.com/Shopify/ruby-lsp/issues/1182) that we spent some time digging into, but ran out of time before we had a solution. It was fun spelunking through the code with friends. [Ufuk](https://ufuk.dev/) joined us in looking at the cursor indentation issue. His sage wisdom helped us navigate through an unfamiliar codebase.

I enjoyed my time at Community Day. For me, I'm not sure if it's worth removing a full day of talks. I'm a person that really enjoys talks at conferences. I'm a person who will gladly join talks in person. But, I have a difficult time committing to watching recorded talks after. I'm also biased, because I'm a potential speaker (I did not submit to RubyConf this year). One fewer day and fewer tracks means less opportunity for people to have an accepted talk.

That said, I talked to a handful of people who organized sessions after. And the vibe I got from them was that it was a positive time. They were happy to work with the community directly, in-person, on their project. They liked meeting new people interested in helping and giving them a starting push. They enjoyed the experience and what it meant for their project.

I'm not sure what the right mix is here. At the least, I think moving Community Day to later in the conference makes sense. That way the organizers can communicate in person with attendees on expectations. Any talks about subjects that will be at Community Day can turn into a call to action to join the hack space.

Could there be a hack space throughout the conference that people could  join? That would have a similar cost that workshops have in the past, where people need to miss other sessions to join. It would mean having another room at the conference that Ruby Central needs to rent. Maybe having it spread out throughout the conference would mean a smaller room? It would be a bit like the hallway track, but structured towards particular goals.

Like I said, unfortunately, I have no solid answers here. And I recognize that the way I enjoy conferences is different from many others. I heard resounding praise for Community Day at the conference!

I do like that Ruby Central is responding to feedback and trying out new experiences. I'm interested to hear a retrospective from them. How do they feel about Community Day and what form(s) it might take going forward?

## Day 2

### Opening Keynote

[Yukihiro "Matz" Matsumoto](https://rubyconf-2023.sessionize.com/session/560574) started the conference after the opening remarks. This was a pre-recorded message. He talked about learning from the history of language design and evolution, to inform the future of Ruby.

{{< figure src="/img/rubyconf_2023_matz.jpg" class="mid" alt="Matz at RubyConf 2023" >}}

### Which Time Is It?

[Joël Quenneville](https://rubyconf-2023.sessionize.com/session/531537) started the talk sessions. He provided a distinction between looking at a moment in time versus a duration of time. He cautions to think through what types of time can interact with each other (the way that you want or expect). Consider the operator you're using and what it means. Catch the full session to learn more about why you can't add two time instances together.

{{< figure src="/img/rubyconf2023_joel_stage.jpg" class="mid" alt="Joël Quenneville at RubyConf 2023" >}}

### Get your Data prod ready, Fast, with Ruby Polars!

[Paul Reece](https://rubyconf-2023.sessionize.com/session/530118) gave an introduction to the Polars library. This started with an introduction to the Series and Data Frame data structures. Paul then walked through data cleaning operations available in Polars. He shared a [repository](https://github.com/paulreece/polars_resources) of resources and code samples at the end of his session.

{{< figure src="/img/rubyconf2023_paul_stage.jpg" class="mid" alt="Paul Reece at RubyConf 2023" >}}

### Ruby Content Creators

During Community Day, Peter Cai asked me to participate in this [open space session](https://rubyconf-2023.sessionize.com/session/186fce40-218a-4d5b-a3f5-01d133b1f7c5). I sat at the written content table with other bloggers and authors. There were other tables for podcasts, live streaming, and pre-recorded videos. The written content table talked through our processes. We lamented over the difficulties in getting feedback. We shared how to build sustainable writing habits.

### The Future of Understanding Ruby Code

[Kevin Newton](https://rubyconf-2023.sessionize.com/session/531482) talked about [Prism](https://github.com/ruby/prism), a new Ruby parser. He discussed the challenges that come with parsing Ruby. He shared what's next, and what we can and should expect from our Ruby tooling in the future. He ended with an impassioned discussion about building a contributor community around a single tool.

{{< figure src="/img/rubyconf2023_kevin_newton.jpg" class="mid" alt="Kevin Newton at RubyConf 2023" >}}

### The Unbreakable Code Whose Breaking Won WWII

[Aji Slater](https://rubyconf-2023.sessionize.com/session/529111) took us back to Bletchley Park to model an Enigma machine with object-oriented principles. This session features monumentally brilliant illustrations and fantastic storytelling.

{{< figure src="/img/rubyconf2023_aji.jpg" class="mid" alt="Aji Slater at RubyConf 2023" >}}

### Closing Keynote

[Sharon Steed](https://rubyconf-2023.sessionize.com/session/560579) ended the first day of talks with a keynote on actionable empathy. We heard about its relationship with vulnerability and how trust is the bedrock to empathy.

{{< figure src="/img/rubyconf2023_sharon.jpg" class="mid" alt="Sharon Steed at RubyConf 2023" >}}

## Day 3

### Opening Keynote

[Saron Yitbarek](https://rubyconf-2023.sessionize.com/session/560580) opened the last day of RubyConf discussing our superpower. Loving what we do is an underrated source of competitive advantage for us to wield.

{{< figure src="/img/rubyconf2023_saron.jpg" class="mid" alt="Saron Yitbarek at RubyConf 2023" >}}

### Meet the Pragmatic Bookshelf

[Noel Rappin](https://rubyconf-2023.sessionize.com/session/560583) joined Dave Thomas and Dave Copeland for a panel discussion on writing. This particularly focused on books with Pragmatic Programmers. We heard about the entire process, from proposal to publication.

One note I found interesting was how the voice of their books changes over time. It intentionally starts with an authoritative tone, telling the reader what to do. As the book progresses and the reader is more familiar with the domain, the tone shifts. Now the tone is more collaborative. The author consults with the reader and uses "we" more.

{{< figure src="/img/rubyconf2023_prag_prog.jpg" class="mid" alt="Noel Rappin, Dave Thomas, and Dave Copeland at RubyConf 2023" >}}

### The Secret Ingredient: How To Understand and Resolve Just About Any Flaky Test

[Alan Ridlehoover](https://rubyconf-2023.sessionize.com/session/527141) walked through a progression of steps to resolve tests failing due to non-determinism, order dependence, and race conditions. With an eye to keeping specs to have one, and only one, reason to exist, he proposes tools and methodologies to trace down these test failures.

{{< figure src="/img/rubyconf2023_alan.jpg" class="mid" alt="Alan Ridlehoover at RubyConf 2023" >}}

### Livin’ La Vida Hanami

[Tim Riley](https://rubyconf-2023.sessionize.com/session/518035) introduced Hanami as the everything framework. The framework is not our app, but enables our application logic through its layers. While it has support for web essentials, you can choose to not include them and still benefit from the underlying framework benefits.

{{< figure src="/img/rubyconf2023_tim_stage.jpg" class="mid" alt="Tim Riley at RubyConf 2023" >}}

### Open Source Ruby

In this [open space session](https://rubyconf-2023.sessionize.com/session/decd439d-f9ec-45be-b016-49ea2f99cd3c) I sat at the Sidekiq table. Mike Perham held an informal chat. We discussed open source sustainability and succession planning. There were tables representing other projects, but I'm not sure what they covered.

### Closing

[Adarsh Pandit](https://rubyconf-2023.sessionize.com/session/3298cf7c-e3ad-4ade-bb53-61ae24e97b2c) wrapped up RubyConf. We learned that RubyConf 2024 will be in Chicago in the fall. That's following a visit to Detroit in the spring for RailsConf.

{{< figure src="/img/rubyconf2023_adarsh.jpg" class="mid" alt="Adarsh Pandit at RubyConf 2023" >}}

## Appreciation

Thank you to my employer, [Pubmark](https://www.pubmark.com/), for supporting my attendance this year.

Thank you [Ruby Central](https://rubycentral.org) for your work to organize these events and supporting the Ruby community all year round.

Thank you to the Program Committee, Scholarship Committee, volunteers, and venue staff for bringing this conference to life.

Thank you sponsors for supporting the Ruby community and making it possible for us to come together.

Thank you Jonan Scheffler and Marty Haught for your years of service and dedication on the Ruby Central board.

Thanks to all the [#RubyFriends]({{< ref "ruby-friends-rubyconf-2023" >}}), old and new, that I met in San Diego.

Thank you for reading to the end. Maybe we met at RubyConf 2023. Maybe we didn't. I hope to see you in 2024. I hear Chicago is lovely in November.

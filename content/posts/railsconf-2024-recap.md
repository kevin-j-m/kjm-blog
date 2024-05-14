+++
title = "RailsConf 2024 Recap"
date = 2024-05-12T09:00:24-04:00
tags = ["ruby", "conference", "rails"]
summary = "How I spent my time in Detroit"
description = "A summary of my time at RailsConf 2024."
+++

## RailsConf 2024

RailsConf recently wrapped up in Detroit, Michigan. This post is meant to highlight the great work from all involved. I hope you'll seek out the full videos of all the sessions that interest you once they are available. Unfortunately, I couldn't be everywhere, so this covers what I saw.

I've separately compiled a list of [brief takeaways]({{< ref "4-things-railsconf-2024" >}}).

## Preparing

I had a bit of work to do leading up to the conference. It was my honor and
pleasure to serve on the [Program Committee]({{< ref "railsconf-2024-program-committee" >}}) for RailsConf this year. Mainly this meant reading all of the proposals. And then the even more difficult work to whittle that down to our [program](https://railsconf.org/speakers/).

The committee broke into smaller groups. I was on the tracks group, even though we didn't have [formal tracks]({{< ref "tracks-not-at-railsconf-2024" >}}). We helped put the schedule together. We provided feedback to those who asked for it about their proposal. I also tried to be as available as possible to any other groups who needed a hand.

Just as I've done in the past, I joined Ruby Central members for the CFP coaching sessions. We helped others with their proposals in small groups. This was a great opportunity to meet with prospective speakers and workshop ideas to submit for the conference. I'm glad that Ruby Central runs these events. They're a great opportunity for prospective speakers. If you're considering submitting to a future conference, I hope you'll join one in the future.

Julie J, co-host of the Ruby for All podcast, was also on the Program Committee. Julie and I joined her co-host Andrew Mason to [talk about]({{< ref "ruby-for-all-guest-2024" >}}) our experience on the committee.

I was also happy to be a speaker mentor. I paired with [Dawn Richardson](https://railsconf2024.sessionize.com/speaker/45d44f90-cd1f-4678-9a87-17c44e3d9900). Dawn had a [terrific talk about](https://railsconf2024.sessionize.com/session/623026) her experience as a Principal Engineer. Dawn was ready with a great topic, talk structure, and plans for delivery before we met. I would have never guessed she was a first-time speaker unless she told me! Definitely check out her talk if you didn't see it at RailsConf.

## Day 1

### Opening Keynote

[Nadia Odunayo](https://railsconf2024.sessionize.com/session/648347) got us started with a reminder of how far Rails can take you. And more generally, how to decide on where to allocate risk. When starting a business, there is so much unknown. Keep the tech simple.

{{< figure src="/img/railsconf_2024_nadia.jpg" class="mid" alt="Nadia at RailsConf 2024" >}}

### So writing tests feels painful. What now?

[Stephanie Minn](https://railsconf2024.sessionize.com/session/628784) walked us through an illustrative example of when testing can be painful. We built the test together, allowing us to understand the justification for why the test was written this way. We felt what it's like to realize we need a change and why. And of course, we worked to improve the situation. I am patiently awaiting the launch of Petreon, so I can sign up as Hickory's Club President.

{{< figure src="/img/railsconf_2024_stephanie.jpg" class="mid" alt="Stephanie at RailsConf 2024" >}}

### Ruby on Fails - effective error handling with Rails conventions

[Talysson Oliveira Cassiano](https://railsconf2024.sessionize.com/session/626598) opened with an exceptional introduction to errors and exceptions. Later they introduced some anti-patterns for error handling before wrapping up with suggestions for effective error handling.

{{< figure src="/img/railsconf_2024_talysson.jpg" class="mid" alt="Talysson at RailsConf 2024" >}}

### Crafting Rails Plugins

[Chris Oliver](https://railsconf2024.sessionize.com/session/626652) gave a tour on how to construct a Rails plugin for distribution. He demystified this process, coming at it from the perspective of someone already familiar with Rails. It turns out, you may be more prepared to build a plugin than you think.

{{< figure src="/img/railsconf_2024_chris.jpg" class="mid" alt="Chris at RailsConf 2024" >}}

### From Cryptic Error Messages To Rails Contributor

[Collin Jilbert](https://railsconf2024.sessionize.com/session/618628)'s teaching style translated just as well to the stage. He comfortably walked through an explanation of an issue all the way through to the fix.

{{< figure src="/img/railsconf_2024_collin.jpg" class="mid" alt="Collin at RailsConf 2024" >}}

### Riffing on Rails: sketch your way to better designed code

[Kasper Timm Hansen](https://railsconf2024.sessionize.com/session/630464) explored code design live on stage. Working within the constraints of a single file (not necessarily class) focused our efforts. Removing the tie to any existing code allowed us to freely consider a world of possibilities.

{{< figure src="/img/railsconf_2024_kaspar.jpg" class="mid" alt="Kasper at RailsConf 2024" >}}

### Closing Keynote: Startups on Rails

[Irina Nazarova](https://railsconf2024.sessionize.com/session/629524) ended our day with stories of current founders using Rails. Irina shared how Rails has been a [competitive advantage]({{< ref "competitive-advantage" >}}). We also explored what needs these founders have that aren't being met by the technology or the community.

{{< figure src="/img/railsconf_2024_irina.jpg" class="mid" alt="Irina at RailsConf 2024" >}}

## Day 2 - Hack Day!

I left this day open in my planning. I did not attend any of the workshops. I started the day meandering through the hack day tables. I asked people what they were working on. I observed the conversation.

Later, I sat down with a friend to look at [code](https://github.com/kevin-j-m/clockwork-test) I had written nine years ago. We revisited the project with multiple sets of new eyes. Some who've never seen it before. Others haven't seen it in a long, long time. We considered if this code even needs to exist. We had a good discussion on alternative implementations. We went through an interesting thought experiment to consider how I would or wouldn't write this code differently now.

## Day 3

### Revisiting the Hotwire Landscape after Turbo 8

[Marco Roth](https://railsconf2024.sessionize.com/session/630406) fit the past, present, and future into a tight package. It didn't feel rushed or consolidated. Because of this, I better understand where we are as a community for building complex UI interactions. I'm equipped with more tools and resources to develop them. I'm informed on where we're going.

{{< figure src="/img/railsconf_2024_marco.jpg" class="mid" alt="Marco at RailsConf 2024" >}}

### The Very Hungry Transaction

[Daniel Colson](https://railsconf2024.sessionize.com/session/626442) brought some delightful storytelling to explain a technical topic. Small changes in transactions may seem innocuous but have drastic consequences. Code in transactions needs additional consideration we don't normally apply outside a transaction.

{{< figure src="/img/railsconf_2024_daniel.jpg" class="mid" alt="Daniel at RailsConf 2024" >}}

### Dungeons & Dragons & Rails

[Joël Quenneville](https://railsconf2024.sessionize.com/session/630449) and Glittersense the gnome demonstrated the magic of Turbo (and stagecraft). The talk is worth the watch for the entertainment value alone. That it provides a delightful survey of the tools in the Turbo toolbox is a bonus.

{{< figure src="/img/railsconf_2024_joel.jpg" class="mid" alt="Joël at RailsConf 2024" >}}

### From RSpec to Jest: JavaScript testing for Rails devs

[Stefanni Brasil](https://railsconf2024.sessionize.com/session/623122) dared to bring us out of our comfort zone. We accepted the reality that JavaScript exists and explored how to bring our rubyist testing practices to that environment.

{{< figure src="/img/railsconf_2024_steffani.jpg" class="mid" alt="Stefanni at RailsConf 2024" >}}

### Beyond senior: lessons from the technical career path

[Dawn Richardson](https://railsconf2024.sessionize.com/session/623026) walked us up the career ladder. By explaining her experience as a Principal Engineer, we're now better equipped for our own progression.

{{< figure src="/img/railsconf_2024_dawn.jpg" class="mid" alt="Dawn at RailsConf 2024" >}}

### Closing Keynote

[Aaron Patterson](https://railsconf2024.sessionize.com/session/648380) explained changes in Ruby, both current and coming, that can inform how we structure and write our Ruby code in a Rails app.

{{< figure src="/img/railsconf_2024_aaron.jpg" class="mid" alt="Aaron at RailsConf 2024" >}}

## Appreciation

Thank you [Ruby Central](https://rubycentral.org) for your work to organize these events and supporting the Ruby community all year round.

Thank you to Andy and Ufuk for inviting me on the Program Committee. Thank you to the rest of the Program Committee: Julie, Ifat, Mayra, Aji, Gary, John, and Zarela.

{{< figure src="/img/railsconf_2024_program_committee.jpg" class="mid" alt="The Program Committee Slide at RailsConf 2024" >}}

Thank you to the Scholarship Committee, volunteers, and venue staff for bringing this conference to life.

Thank you sponsors for supporting the Ruby community and making it possible for us to come together.

Thanks to all the #RubyFriends, old and new, that I met in Detroit.

Thank you for reading to the end. Maybe we met at RailsConf 2024. Maybe we didn't. I hope to see you in 2025. Let's wrap up the long tradition of RailsConf with the send-off it deserves.

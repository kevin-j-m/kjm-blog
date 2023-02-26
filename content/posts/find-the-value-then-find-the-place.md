+++
title = "Find The Value. THEN Find The Place."
date = 2023-02-26T12:23:22-04:00
tags = ["ruby", "software-design"]
summary = "Avoiding fitting a round peg in a square hole"
description = "Rather than starting with a choice of where to house some code, this proposes focusing on what it does. After ensuring it meets your needs, then decide where it should live."
+++

Recently I was pairing with someone. They were currently driving, and asked for my opinion on a decision. I said something that was dangerous. Words that may have sounded abrupt. Words that could paralyze a pairing session. Words that could show an unfixable difference of opinion. The words I said were:

> I don't care.

## Care To Explain

My pair and I were wrangling some logic in a controller. We decomposed what was a long public action into smaller private methods. We discovered we could bundle a few of these methods up into a domain concept. We thought of creating a class for these. We even had a name for it! Then my pair asked the 🔥burning🔥 question that prompted my uncaring response:

> Where should we put this class? What directory should it live in?

## Find The Value

The reason I didn't care was because I wasn't prepared to answer that question. And the reason for __that__ is because I didn't know if we needed to care yet.

What we had was an idea. We thought this extraction would make the controller more understandable. We knew it would be easier to test. We thought it could *possibly* have a use outside the controller (but had no hard evidence of that).

I wanted to prove out those hypotheses before considering where this class lives. I wanted to make sure we actually wanted to keep it around first. I wanted to play with the structure of the code. I wanted to confirm the design that existed in our head. I wanted to see if it would live up to the expectations we had for it. I wanted to see if we'd like it.

I didn't want to stop in our tracks to find the right pattern, location, or directory for this code. We had other decisions to make first.

I suggested that we start by inlining the class in the controller.

```ruby
class WidgetsController < ApplicationController
  class YouCanJustDoThis
    def initialize
      @location = :anywhere
    end
  end

  def index
    YouCanJustDoThis.new.and_play_with_it
  end
end
```

If that was uncomfortable to my pair, I recommended we put it wherever their intuition said to put it to start.

I, of course, then explained what my goals were and why I said I don't care. I may only have a little, but I do have *some* tact.

Unsurprisingly, the class did not end up looking exactly as we envisioned it. It needed different arguments passed to it. We added more methods. We removed some. We drew parallels to other constructs in the codebase and looked to mirror their form.

The shape, context, and even the name of this class changed. Our class living inside the controller handled that change swimmingly. The co-location between the class and where it was being called made it easy to see the impact of our changes.

Eventually, we were comfortable with what we had hammered out of the controller. We found it *valuable*. We knew we wanted to keep it. We were ready to share this with the rest of the application.

## Find The Place

With that hard work out of the way, now we had to find a place for it to rest. And it turns out, there was a natural place for it. I'll grant you, that's not always the case.

We started without needing to think about what kind of THING this was. We were free to explore the API structure, unconstrained by a directory name or convention. Was it a model? Was it a service? A decorator? A presenter? Any other constructs that live in your Rails app defined by what kind of box all these objects fit in?

By deferring that decision, we let it grow into what it wanted to be. We did not force it into a structure prematurely.

## Value Assessment

Maybe we could have avoided this entirely if the app already made different [domain-driven design](https://en.wikipedia.org/wiki/Domain-driven_design) choices. Or if we organized our files [by meaning](https://www.codewithjason.com/organizing-rails-files-by-meaning/). But we needed to start where we were at. That was a pretty standard Rails application structure. And rather than be paralyzed by this choice, I wanted to write some code. We needed to validate our idea against our current use case for it.

Inlining it in our controller broke down another barrier preventing [blank page syndrome](https://writingcooperative.com/the-blank-page-syndrome-6e6e5d1250fe). Thinking about where something lives may frequently be the start of the process. We didn't want that to slow us down. Especially when we didn't know if this idea would survive our experiment.

If it helps, I give you permission to put code you're toying with anywhere you want to start. Wherever will give you the comfort and confidence to move forward with your problem. If you find a more natural fit, don't fret. Changing your mind is only a `mv` away.

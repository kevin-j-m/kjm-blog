+++
title = "Building Conference Talk Content"
date = 2023-09-15T18:23:22-04:00
tags = ["conference"]
summary = "Turning an outline into slides"
description = "This documents my methodology of taking an accepted talk proposal and building the slides and content around it."
+++

## Conference Talk Preparation Series

1. [Sharing Past Conference Proposals]({{< ref "/sharing-past-conference-proposals" >}})
2. __Building Conference Talk Content__
3. [Preparing Conference Talk Delivery]({{< ref "/preparing-conference-talk-delivery" >}})

Have you recently had a conference proposal accepted? Congratulations! Are you wondering, "what do I do *now*?" Here is the process I follow to turn my proposal into a full-length talk.

## Content Generation

My first step is to gather all the content I want to talk about as quickly as possible. I know that sounds reductive. Oh, the first step is JUST to put together all your content? How easy! I might as well tell you [how to draw an owl](https://knowyourmeme.com/photos/572078-how-to-draw-an-owl) next.

<<<<<<< HEAD
But, here's the thing - for a conference talk, I'm not starting from scratch. I've already put a lot of time, and thinking, into the [proposal]({{< ref "/sharing-past-conference-proposals" >}}). For me, that almost always includes a full outline. So I start by revisiting that and seeing what I need to add.

I completed most of my research in the proposal, but I may need to brush up on some details. There may be a book or research paper that I skimmed that I need to go back to and dig into.

Will I be using code examples to demonstrate some points? I'll start writing some code samples, in my usual code editor, that show what I want. I'll make sure they run. At this point, I'm not concerned about how terse or verbose they are. I'm not worried what the domain is. I need something that shows the point I want to make. I will admit that sometimes this code generation process is procrastination. I try to call myself out on that and not put too much effort into it at this point.
=======
But, here's the thing - for a conference talk, I'm not starting from scratch. I've already put a lot of time, and thinking, into the [proposal]({{< ref "/sharing-past-conference-proposals" >}}). For me, that almost always includes a full outline. So I start by revisiting that and see what I need to add.

I completed ost of my research in the proposal, but I may need to brush up on some details. There may be a book or research paper that I skimmed that I need to go back to and dig into.

Will I be using code examples to demonstrate some points? I'll start writing some code samples, in my usual code editor, that show what I want. I'll make sure they run. At this point, I'm not concerned about how terse or verbose they are. I'm not worried what the domain is. I need something that shows the point I want to make. I will admit that sometimes this code generation process is procrastination. I try to call myself out on that and not put too much effort into them at this point.
>>>>>>> bdb7b2b (post about delivering a conference talk)

## Shameless Green

My next step is to put my thoughts together on slides. I use the default theme. White background with black text. Whatever the default font is. I do not care about slide design __at all__ at this point.

Some slides are one word. Some are an emoji. Some are a dense block of code. Some start as bullet points. Sometimes it's a thought about what I *want* the slide to eventually be, like, "insert picture of an owl". Everything is a placeholder at this point.

I call this process "Shameless Green". I borrowed this term from the book ["99 Bottles of OOP"](https://sandimetz.com/99bottles-sample-ruby#_shameless_green) by Sandi Metz, Katrina Owen, and TJ Stankus. It doesn't match exactly, but it's stuck in my head, so I continue to use it. Intentionally reminding myself to be shameless gives me more license to do that. There's no polish here. This isn't even a first draft; it's only for me - there's no chance of another human seeing this.

## First Delivery

After this, I deliver the talk. No, I don't get on stage with it. I don't have an audience. I close the door to my office, stand up, and speak out loud as if I were on a stage - with my slides in front of me on my computer.

This will be *rough*. I'll feel uncomfortable. I stop to write down when those feelings hit. Not enough to pull me out of my rhythm, but I don't want to forget it.

Some things I might write:

* Slide 7 - awkward intro
* Slide 13 - example doesn't work
* Slide 16 - missing context
* Slide 32 - repeating info from earlier

After I've gone through everything, I take a few minutes to review my notes. Only to make sure everything is legible and makes sense. I take the time to add more details where needed.

And then I walk away. I'll revisit it later with a fresh perspective.

## Editing

I take a few hours to a day away from my initial attempt. I then review the notes while iterating through the slides. What I'm looking for here is any *structural* changes I need to make.

Is there a different order I should present information in? Maybe the section in Slide 32 needs to happen closer to Slide 16. That'll add more context immediately and eliminate the need to repeat myself later on.

If there are any transitions that feel awkward, I dig into why. Am I missing information? Do I have unnecessary information? Do I need to reorder the content? Is there a connection that I need to make more explicit, for myself and the audience?

These changes likely don't get done in one sitting. I revisit a section, or a transition, at a time. Then I review everything in total again. That might spark another round of smaller sections to focus on.

## Theme

Now I'm comfortable with the general flow of information I want to share with the audience. Next, I build my __theme__. I still don't mean slide design here. I mean the real or imaginary [world]({{< ref "sharing-past-conference-proposals/#go-all-in-with-a-theme" >}}) that my talk will be about. Some talks have a theme as part of their proposal. For others, I need to develop it at this point.

A theme is important for me in a talk because it turns my presentation into a story. I have a hard time delivering information over that period of time in hard, technical terms. Other people can, and that's great! It doesn't work for me. I need to have some real or imaginary scenario that the content relates to. Most of the time, this involves creating an imaginary product or company. That will illustrate the topics I'm sharing with the audience.

I may need to change all my examples to match the theme. The code samples now should reflect that theme. Let's say a talk is using a theme of managing scientific study participants. Code that was using ActiveRecord callbacks to send an email is now sending an [informed consent](https://en.wikipedia.org/wiki/Informed_consent) document.

## Second Delivery

I've reworked my transitions, moved sections around, and built up a story around my content. Now it's time to run through the presentation again. This time, I take fewer notes, but I also add something.

In this practice session, I start timing the presentation. Where I have discernible sections, I track those as splits/laps (depending on your timer of choice). Talks need to fit within a defined schedule. This is nowhere near polished. Still, the timing gives me a helpful sign of how far off I am from where I need to be.

Most of the time, I discover I'm way over on time.

## Editing

In this editing session, I'm incorporating the same process from the first edit, but I'm adding to it. What am I going to cut entirely? Usually at this point I know what I want to say will take too long and I need to remove pieces. Even if I'm on time, I may recognize that there are pieces that don't complement my goal.

This is why I focus on "shameless green" to start. I haven't invested in picking colors or graphics for my slides. I haven't pored over detailed animations and visuals. Getting rid of content isn't *easy*, but it's harder (for me) the more work I've put into it.

This doesn't mean you've lost all that work you cut forever. First off, it was __necessary__ to get what remains to where it is now. You learned something as part of putting that together that'll help you.

You can also reuse it elsewhere. I've turned sections of presentations I've had to cut into entirely different talks. Some have become blog posts. If you're not willing to let it go, but it doesn't fit in this format, it doesn't need to be the end. It needs to go somewhere else.

## Slide Design

I am by no means a designer, but I do my best to deliver pleasant slides. Here are some things I look for.

Does the slide complement what I'm saying? My speech and the slide should work together, but not be the same. They should highlight what's important without being repetitive. How can I replace words with visuals? I've turned sentences of text into a single emoji. I've replaced them with a picture. I try to limit the text on the screen. I want my audience mostly listening to what I'm saying, not reading what's behind me.

Are the slides cohesive? I use slide templates to use consistent fonts, color schemes, and layouts. This also makes incorporating large design changes easier later on. Rather than needing to change 80 slides individually, I change the template.

Are my slides accessible? I run my color scheme through a [contrast checker](https://webaim.org/resources/contrastchecker/). I strive for a WCAG AAA level of contrast between my colors. I typically pick no more than four colors. A consistent background color. A consistent text color. No more than two colors for highlighting or drawing attention to parts of the slide.

Are my slides readable to the audience? I avoid putting any information you need the audience to see on the bottom third of the slides. Someone's head might be in the way. I distill code down to the essentials, even if that means it isn't syntactically valid or complete. I use as large of a font size and as few words as necessary.

<<<<<<< HEAD
I have what's showing on the slide be what you want the audience to focus on. Not what you're going to tell them in two more minutes. I don't have the entire code method on the slide. I have the one line that I'm talking about right now. Then the next slide adds the second line to it.

If I ever use any slide builds, each build is a separate slide. I don't use an animation that happens within one slide. It makes it easier to go back and forth between slides, if  necessary.

## Next Steps

I have my slides built at this point, but my work isn't over. Next I go through the laborious, repetitive process to provide a consistent delivery.
=======
I have what's showing on the slide be what you want the audience to focus on. Not what you're going to tell them in two more minutes. To help me have more control over this, if I ever use any slide builds, each build is a separate slide. I don't use an animation that happens within one slide. It makes it easier to go back and forth between slides, if  necessary.

## Next Steps

I have my slides built at this point, but my work isn't over. Next I go through the laborious, repetitive process to provide a [consistent delivery]({{< ref "/preparing-conference-talk-delivery" >}}).
>>>>>>> bdb7b2b (post about delivering a conference talk)

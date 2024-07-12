+++
title = "Site Search Now Available"
date = 2023-12-10T07:30:22-05:00
tags = ["personal"]
summary = "A bold product announcement for this site"
description = "This site now has search site powered by Pagefind"
+++

I'm no stranger to [search]({{< ref "searching-for-a-reason" >}}). Give me [25 minutes]({{< ref "browser-history" >}}) and I'll tell you my thoughts on how and why we search for things. All that said, one thing you couldn't do on this site for a long time was search. I'm proud to announce on behalf of the entire team responsible for this site (me) that an innovative new feature is now available: [search]({{< ref "/search" >}}).

Any questions?

## What took so long?

Two main reasons: self-esteem and laziness. I don't think anyone else will use this functionality. I don't think people will read *this*. Many times, I write for myself. So I want to reference my own posts on occasion. And for myself, I know what I'm looking for and when I published it in comparison to other posts. So I've just been paginating through my results until I find the post.

The other piece is that I don't want to spend a lot of time working on my website design/layout/features. That's time I could be writing for my website, or doing literally anything else. My site is intentionally in Hugo, because while I've written Golang, I'm not apt to change code in there. The templating engine is easy enough to do what I want, but I've long resisted the urge to revamp the theme I chose when starting the site.

## Why now?

I was inspired by Justin Searls, who [recently wrote](https://justin.searls.co/links/2023-11-29-shout-out-to-pagefind-static-search/) about adding search to his site. If Justin is actually trumpeting some tech and not sharing cursed bugs with it, then it's worth my attention.

And as a side-benefit, maybe I wouldn't have to page through my own site as often.

## How did the implementation process go?

Just like Justin, I'm using [Pagefind](https://pagefind.app/). The process of implementing the index was really straightforward. I followed their [Quick Start guide](https://pagefind.app/docs/) and the indexing was done.

With a one-line change to my deploy script, I was set to have this indexing process update for each change to my site. Great work Pagefind!

The part that took time was my inability to know how my own site works and how to make it look nice. My theme has a light and dark mode. After adding Pagefind, I had to wrap up the initial implementation by turning off dark mode, otherwise the Pagefind results were illegible. This felt like a fine trade-off on a Friday night, as I imagine it would only affect me - and I was going to sleep.

The Pagefind docs clearly suggest how to [change the Pagefind styles](https://pagefind.app/docs/ui-usage/#customising-the-styles), even including recommendations for a dark mode. Well done, Pagefind!

The trouble was I needed to understand how my site even implements dark mode. Through this, I learned about CSS's `prefers-color-scheme` [feature](https://developer.mozilla.org/en-US/docs/Web/CSS/@media/prefers-color-scheme). I told you, I don't really want to know implementation details in my site so I'm not tempted to change them. I also revisited CSS [custom properties](https://developer.mozilla.org/en-US/docs/Web/CSS/Using_CSS_custom_properties) and styling input [placeholder values](https://developer.mozilla.org/en-US/docs/Web/CSS/::placeholder).

Did you know Firefox's inspector has the ability to [toggle light mode and dark mode](https://www.linuxadictos.com/en/firefox-87-adds-an-option-to-its-inspector-that-allows-us-to-switch-between-light-and-dark-mode-if-the-web-allows-it.html) for a site? I didn't, but now I do!

{{< figure src="/img/firefox_inspector_light_dark.png" class="mid" alt="The Firefox inspector with arrows pointing to how to change to light and dark mode" >}}

## How do you feel now?

I'm satisfied. I can [search]({{< ref "/search" >}}) my site. All with very little effort on my behalf. Again, thanks Pagefind! And Justin for the suggestion. I could dig into [modifying](https://pagefind.app/docs/weighting/) my index, or [sorting](https://pagefind.app/docs/sorts/) results, but I'm going to stick with the default experience for now. Those incremental improvements wouldn't have the value add compared to what this already provides. Before there was no search, and now there is!

Is it my greatest work? Something I would put into production at work? Not the way I wrote it. This is not a reflection on Pagefind. I *would* recommend that for a static site. However, I've committed some [terrible atrocities](https://github.com/kevin-j-m/kjm-blog/commit/a36f1cbcc6b665ce69c281addd8e31a90fb56b50) to HTML and CSS to fit it into my site's theme.

But, I did that styling while [Emily's Wonder Lab](https://www.netflix.com/TITLE/81128389) was playing on TV. And they were making a tornado. So I wanted to wrap this up and watch it with my daughter. So, it's good enough for me, for now. I hope you enjoy it too.

Happy [searching]({{< ref "/search" >}})!

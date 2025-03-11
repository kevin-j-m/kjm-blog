+++
title = "Hugo Update"
date = 2025-03-10T10:48:04-04:00
images = []
tags = []
categories = []
summary = "From tragedy comes rebirth (the bare minimum)"
description = "I updated the version of hugo and the deployment story for this site"
+++

I try to do the bare minimum of development work on this *actual* site. That way my time spent here is on actually blogging. That's why I chose [hugo](https://gohugo.io/) over something like [Bridgetown](https://www.bridgetownrb.com/). By intentionally choosing a static site generator written in a language I'm less familiar with, I'm less likely to tinker.

Due to a tragedy with a git submodule this morning (that is...deleting it), I ran into an issue where my custom deploy script would no longer deploy. Attempts to reconfigure the submodule were not responsive. I'm sure I could have figured it out, but...I didn't want to.

I found there is a more modern deployment story with Hugo now that is in their documentation - using [GitHub Actions](https://gohugo.io/host-and-deploy/host-on-github-pages/)! I wanted to use that instead going forward.

That created a situation where the git submodule for the theme I use wasn't recognized in the GitHub Action. Updating the theme required updating Hugo itself and...here we are.

I've noticed at least one visual regression, which is that my fancy social link icons aren't available on the top header (Update: they should be back now). If you're seeing this and missing them that much, I do keep track of [where to find me on the internet]({{< ref "/about/#elsewhere-on-the-internet" >}}).

If you find anything else that's not working or seems off, could you let me know?

+++
title = "Scraping Buy: Scripting for a Purchase"
date = 2022-05-11T16:53:59-04:00
tags = ["ruby"]
categories = []
summary = "An email that was music to my ears"
description = "I wrote a small script to scrape a website to tell me when guitars were in stock."
+++

## An Opening Tune

The piece of code I've written in the last two months most interesting to me isn't anything I've done for work. It isn't anything I wrote to [find]({{< ref "/posts/available-for-hire-2022" >}}) a new job. It doesn't have tests. It doesn't even have a class. It's dirty, not up to my standards, and doesn't handle edge cases. But it does what I need it to.

## Sad Song

I don't (and likely will never) consider myself a guitarist, but I like the guitar. I'd like to have a Les Paul to go along with my Stratocaster, but I also like having money for things like food and shelter. I can't justify in the budget, and that's fine. But maybe I could find something close enough for less money.

I zeroed in on a copy, and I decided on a Firefly. There was only one problem - I couldn't buy one. At least, not from them. Keeping them in stock was problematic. Not only because of supply chain issues, but because of their business model. They batch up a small number of guitars a few times a year for sale. I wasn't the only person with the same plan to buy one. They sell out quickly most of the time.

You can find them on the [secondhand market](https://reverb.com/marketplace?query=Firefly%20FFLPS). Typically, they're priced higher than their new price direct from the manufacturer. The demand is there. They don't make them often enough. And if you want one, you've got to be fast.

## Composing A Number

This is about the furthest thing from an important use of my time. Still, I became a bit...overtaken by the prospect of ordering one. I'd refresh the site many times a day, just to check in. Once, I happened to refresh the site when they were uploading their fresh inventory. I was in! But, I did not get to the model I wanted in time. It sold out before I could find it.

Finally, I remembered that computers can do repetitive work for you, and that I knew how to request them to do so. I resolved to stop refreshing their product listing page. At least, I'd stop refreshing it myself. I wrote a small script to do it for me.

```ruby
require 'nokogiri'
require 'open-uri'

doc = Nokogiri::HTML(URI.open("#{root_url}/collections/fflp-electric-guitars"))

guitars = doc.search("a").
  map { |a| a[:href] }.
  compact.
  select { |a| a.include?("/collections/fflp-electric-guitars/products") }.
  uniq.
  map{ |a| "#{root_url}#{a}" }.
  sort

puts guitars
```

I wouldn't put this in production at work. It doesn't have consistent formatting! It's not clear exactly what this is doing! It's not tested. Literally. I don't mean it doesn't have automated tests. I mean I had no idea if this would work. When I wrote it, I couldn't test it against a page that had any products. I found one of their other products that did have inventory in stock, and built it against that. I hoped that the page for this product line would work the same, and that I didn't have a typo.

This doesn't keep track of their full inventory. It has no awareness of if there are pages of results. But, it's good enough to tell me that there *are* guitars for sale. That's a start. Hopefully. I tired of seeing the "sold out" page. And I wouldn't even know if it would work until that "sold out" page went away.

## Automated Playback

Having the script is great, but if I have to run it myself, I might as well refresh the page in my browser. Once again, I needed to remind myself what I do for a living, and how I can ask computers to do things for me. Of course, that only happened when I came across [this post](https://simonwillison.net/2020/Oct/9/git-scraping/). In it, the author uses GitHub Actions to automate diff checks of websites. Now sure, their example was for trivial things like forest fires in California. But I could take the same approach for important things like guitars being in stock.

I modified my script to dump its output to a file, set up the GitHub Action, and subscribed to the repository. When the action pushed a commit because the page changed, I would get an email with a link to the commit. I could use that to then see the list of guitars for sale.

In theory. I still needed to see it happen. Ever.

## Outro

It finally happened. And it just arrived.

{{< figure src="/img/firefly_goldtop.jpg" class="mid" alt="A Firefly FFLPS Goldtop" >}}

Code doesn't have to be perfect. It doesn't always need to adhere to your normal standards. Sometimes, it can be good enough to solve a problem you have - especially if it's temporary. Even better if you *know* it'll be temporary. I've written lots of "temporary" code that I'm sure is still in production, years later. Writing code like this is a gamble.

But the stakes were incredibly low here. I wouldn't put this in my portfolio if I were looking for a job. It served its purpose though. I'm happy to retire this code. Thanks to it, I have other things to play with rather than maintain this.

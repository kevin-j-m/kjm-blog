+++
title = "Nokogiri Notoriety"
date = 2021-01-13T18:20:36-05:00
images = []
tags = ["ruby", "self-promotion"]
summary = "Installing nokogiri with MY instructions"
+++

Last week, I wrote a [post]({{< ref "caching-all-native-gem-platforms" >}}) on 
my employer's [blog](https://blog.thegnar.co/caching-all-native-gem-platforms) 
about how to set up the correct bundler configuration 
to support caching multiple platforms of gems. This came up with the latest Nokogiri 
release, where pre-built binaries for various platforms are now available, so 
you don't need to install from source. 

That post has been featured in Ruby Weekly [#534](https://rubyweekly.com/issues/534)! 

{{< figure src="/img/ruby_weekly_534.png" class="mid" >}} 

I was also humbled to receive a tweet reply from Nokogiri maintainer Mike Dalessio, which says:

> I've linked out to your post from the "Troubleshooting" section of the installation guide. Thanks again! 

And [here I am](https://nokogiri.org/tutorials/installing_nokogiri.html#using-vendorcache-to-deploy-to-another-architecture) 
in the Nokogiri tutorials! 

{{< figure src="/img/nokogiri_tutorial.png" class="mid" >}} 

The commit adding me to the guide is [here](https://github.com/sparklemotion/nokogiri.org/commit/173ecdc110c738d0c5708934eb51d03e8e9f418d). 

Thanks to Peter Cooper for featuring me in Ruby Weekly, and Mike Dalessio for 
adding me to Nokogiri's guides! 

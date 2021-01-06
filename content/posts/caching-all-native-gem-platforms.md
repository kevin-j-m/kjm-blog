+++
title = "Caching All Native Ruby Gem Platforms"
date = 2021-01-06T12:20:07-05:00
tags = ["ruby"]
summary = "Installing nokogiri WITHOUT native extensions"
+++


## TL;DR

Some dependencies, like nokogiri, ship with multiple libraries for different
architectures. If you cache your gems, you may need to cache multiple platforms,
because your development team is spread across various platforms or you deploy
to a different platform. To do this, you can use:

```sh
bundle cache # cache gems in vendor/cache
bundle lock --add-platform x86_64-linux # add additional platforms
bundle package --all-platforms # cache multiple platforms
```

On bundler version 1.x, add:

```sh
bundle config specific_platform true
```

## Native Nokogiri

Nokogiri 1.11.0 has been released, and one of the exciting [updates](https://github.com/sparklemotion/nokogiri/blob/007662fc216902a5ae186cb78b0d46f7f48b8d92/CHANGELOG.md#v1110--2021-01-03)
is the inclusion of pre-compiled native gems for various platforms! If you're
using a supported platform, your days of installing nokogiri with native
extensions may be over. These [changes](https://nokogiri.org/tutorials/installing_nokogiri.html#installing-native-gems)
result, "in much faster installation and more reliable installation". Many
thanks to the maintainers and contributors for this great update.

Updating to these pre-compiled gems should be a seamless experience. Bundler
will grab the appropriate pre-compiled `.gem` file, if you're on a
supported version, and use that. However, if you [cache](https://bundler.io/man/bundle-cache.1.html)
your gems, and you'd like to cache multiple platforms, you have a few more
steps to complete.

## Cache Hit

Gem dependencies can be cached along with your app and then you can use that
cache to retrieve your application's dependencies, rather than RubyGems.
We take advantage of this on a number of projects for various reasons, but the
most important one that requires all gems to be vendored is that some
applications are deployed to, and the deployments are created in, environments
where they **cannot** access [RubyGems](https://rubygems.org/) directly.

We need to tell bundler to cache our gems.

```sh
bundle cache
```

Running that on an existing application will add `.gem` files into the `vendor/cache`
[directory](https://github.com/kevin-j-m/bundler_2_cache_all_platforms/commit/75c7ddb8f569e33cdd09e0ee1c4a377885318416).

## Platform Dependence

You need to tell bundler that you require multiple platforms. In the case of
this example, I'm developing on a computer running macOS, so installing nokogiri
will [give me](https://github.com/kevin-j-m/bundler_2_cache_all_platforms/commit/61b73d01e929ec3b43c43ae9ea1af00df273a4b1)
the pre-compiled gem for that architecture. That's great, but I also need the
linux native gem for my deployment environments.

First, I need to tell [bundler](https://bundler.io/v2.0/bundle_lock.html) to
add the platform.

```sh
bundle lock --add-platform x86_64-linux
```

After doing that, the `Gemfile.lock` file is [updated](https://github.com/kevin-j-m/bundler_2_cache_all_platforms/commit/d36495a715c26fb1f674021ffc19dc61c1787e4f)
to list that platform.

## Platform Independence

However, even if you add the platform _before_ installing the dependency, adding
the platform will still not retrieve and cache both platform's `.gem` files.
We also need to tell bundler to cache those
[other platforms](https://bundler.io/man/bundle-cache.1.html#SUPPORT-FOR-MULTIPLE-PLATFORMS).

```sh
bundle package --all-platforms
```

Now our other platform is [cached](https://github.com/kevin-j-m/bundler_2_cache_all_platforms/commit/a9ef611df04114921e71b4d8e4100894d00e2925),
along with the existing platforms.

If you are using Bundler version 1.x, you may also need to set the
`specific_platform` configuration [setting](https://github.com/rubygems/bundler/issues/5863#issuecomment-315800951).

```sh
bundle config specific_platform true
```

Now you should have all your gem dependencies cached across all platforms
specified in your `Gemfile.lock`. You no longer need to compile nokogiri!


> This post originally published on [The Gnar Company blog](https://blog.thegnar.co/caching-all-native-gem-platforms).

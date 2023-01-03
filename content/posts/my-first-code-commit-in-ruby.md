+++
title = "My First Code Commit in Ruby"
date = 2022-12-29T12:04:22-05:00
tags = ["personal", "ruby"]
summary = "Next step: commit bit (not really)"
description = "I had my first commit with a code change introduced into the Ruby programming language."
+++

## Kevin Murphy: Ruby Committer

I recently had my first [code change](https://github.com/ruby/ruby/commit/b3d330c39ebbf27cefc2d83109dad9e0b3b0e94f) accepted to the Ruby programming language! Ruby's `Coverage` module now responds that it supports oneshot_lines mode.

Though it may be a small change, I'm more excited at the symbolism of having committed to the Ruby code base. As a celebration, I figured I'd share what led to this.

## Covering New Changes

As I write this, we're in that period between Christmas and New Year's Day. In the past few years, I've taken that off of work. That also lines up with the release day for a new version of Ruby. This year, [3.2.0](https://www.ruby-lang.org/en/news/2022/12/25/ruby-3-2-0-released/) came out on Christmas day.

Having some time where I'm not working, I set some very healthy boundaries. Instead of working, I would instead *read* about work-related things. While I was catching up on the changes in 3.2.0, something caught my eye - there were some changes to Ruby's Coverage module.

Coverage can [now support](https://github.com/ruby/ruby/pull/6396) measuring code coverage of code in `eval`. I've written more about that in a [later post]({{< ref "evaluating-more-coverage-in-ruby-3-2" >}}). As a follow-up to conversation in the [initial issue](https://bugs.ruby-lang.org/issues/19008), [another PR](https://github.com/ruby/ruby/pull/6462) adds controls for whether `eval` coverage is running.

Part of that change adds a [new method](https://bugs.ruby-lang.org/issues/19026) to the `Coverage` API. `Coverage.supported?` is a method that accepts a symbol as an argument. It tells you if the Coverage module supports the mode or type of coverage asked about.

I pulled up the [documentation](https://ruby-doc.org/3.2.0/exts/coverage/Coverage.html#method-c-supported-3F) for the method, and noticed this sentence:

> The mode should be one of the following symbols: :lines, :branches, :methods, :eval.

I also happen to know another mode of coverage: [oneshot lines](https://ruby-doc.org/3.2.0/exts/coverage/Coverage.html#module-Coverage-label-Oneshot+Lines+Coverage). More on why I know about that later. Wondering if there was an opportunity to update the documentation, I checked the [source code](https://github.com/ruby/ruby/blob/a7d467a792c644a7260d6560ea2002fdb8ff6de3/ext/coverage/coverage.c#L40).

```c
static VALUE
rb_coverage_supported(VALUE self, VALUE _mode)
{
    ID mode = RB_SYM2ID(_mode);

    return RBOOL(
        mode == rb_intern("lines") ||
        mode == rb_intern("branches") ||
        mode == rb_intern("methods") ||
        mode == rb_intern("eval")
    );
}
```

It turns out, the source doesn't mention oneshot lines either. I asked Samuel, who made these contributions, on [Mastodon](https://ruby.social/@kevin_j_m/109591612453458271) about it. After that, I wrote my [first issue](https://bugs.ruby-lang.org/issues/19279) on Ruby's issue tracker, and had [my PR](https://github.com/ruby/ruby/pull/7040) merged in.

So how do I know about oneshot lines coverage? Or Ruby's Coverage module at all? To answer that, we need to go back much further in time.

## Covering My History With Coverage

### RubyConf 2019

The year is 2019. I'm speaking at RubyConf for the first time. I'm very excited about this. It'll be my second time speaking at a conference. Earlier that year I also spoke at RailsConf.

My talk is about different [best practices]({{< ref "rubyconf-2019" >}}) - specifically when adhering to them breaks down. One of those best practices is high [test coverage]({{< ref "rubyconf-2019/#code-coverage" >}}). I start to work on the content for my presentation by building the code samples that I want to use in the slides. For the code coverage section, I'm writing some code with some tests. I'm using [SimpleCov](https://github.com/simplecov-ruby/simplecov) to generate code coverage results.

As I'm doing this, I find a perfect opportunity for a distraction: how does SimpleCov work? I do some quick investigation. The second sentence of the [README](https://github.com/simplecov-ruby/simplecov/blob/216b2d530bee7c4b8a8fe2898684924bfccfa79a/README.md) states:

> It uses Ruby's built-in Coverage library to gather code coverage data...

That gave me a new distraction - what is this Coverage library that's built in to Ruby? I started with the documentation, that at the time looked something like [this](https://ruby-doc.org/stdlib-2.6.5/libdoc/coverage/rdoc/Coverage.html). Wanting to learn more, I pulled up Ruby's source code, and pretended I knew how to read C well. In reviewing the `coverage.c` file, which looked something like [this](https://github.com/ruby/ruby/blob/ruby_2_6/ext/coverage/coverage.c), I learned that it supports [different modes](https://github.com/ruby/ruby/blob/ruby_2_6/ext/coverage/coverage.c#L42-L48) of coverage.

I spent an afternoon playing with the different modes. I built different examples that I could use in my presentation. None of it actually made the talk though - I didn't have enough time, and it wasn't the primary focus.

That doesn't mean I wasted all that effort though.

### RubyConf 2020

At least with the people I knew, there wasn't a lot of awareness of the Coverage module. People were very familiar with the concept of code coverage. No one I spoke with knew that Ruby has a built-in facility to measure it or how it worked.

Because of that, I submitted a [proposal]({{< ref "enough-coverage-to-beat-the-band-proposal" >}}) to talk about Coverage at RubyConf 2020. I already had some code samples documenting how to use it. I thought it would be a unique proposal. I had a ready-made theme to tie everything together. I had a lot of time in my house to work on it. If you're wondering why, consult some resource about what was going on in the world then.

The result is, as of this writing, my [favorite talk]({{< ref "coverage" >}}) I've put together. I had so much fun with it, that I brought it back out in 2021, still at home, for a "Ruby's Got You Covered" [world tour]({{< ref "speaking#enough-coverage-to-beat-the-band" >}}). I'm very thankful to all those groups for accepting a virtual, not local, speaker at their event.

### Documentation

At the beginning of this post, I was very specific in my wording to say this is my first "code change" accepted to Ruby. That's because it's not my first __commit__ to Ruby.

At RubyConf 2020, [mame](https://github.com/mame) was in the audience for my talk. Another benefit of a virtual conference: I couldn't see that, so I had no idea. As a result, I couldn't be nervous about it. They just so happen to have been the person that wrote most of the Coverage module's code.

We talked afterwards in the conference's chat platform, and that led to my [first commit](https://github.com/ruby/ruby/commit/0026f644d739efed0d69911b434a1012ad55c393) into Ruby. I added what I learned in preparing that talk into the documentation of the module. Now people don't have to look into the source code to discover the different modes.

In preparation for that 2021 world tour, I also created a [blog post]({{< ref "rubys-got-you-covered" >}}) explaining coverage.

## Covering Why This Matters

And *that* is the very long explanation about how I'm interested in the Ruby 3.2.0 Coverage changes. And more specifically, how I noticed oneshot lines was missing as a supported mode.

Beyond my excitement about having a change (admittedly small) accepted into Ruby, I hope this inspires you to think about what you have to share. That may sound daunting. I know I never expected to commit anything to Ruby.

It started with me reading a lot of code and playing around in my terminal. Telling some friends turned into a talk. The talk led to an encounter with a Ruby committer that resulted in a documentation commit. Turning that into a blog post got me thinking about how much fun that talk was. That became a whole tour (where I stayed in my office the whole time). Now, I have a small modification to that area of the code in Ruby itself.

What's something neat you've been playing with? What's the smallest step in your mind you could take to comfortably share that - even with one person? Years later, who knows what that might lead to.

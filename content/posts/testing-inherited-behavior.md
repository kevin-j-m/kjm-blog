+++
title = "Testing Inherited Behavior"
date = 2023-07-14T19:00:24-04:00
tags = ["ruby", "testing", "rspec"]
summary = "Inheriting the wisdom of crowds"
description = "This post describes different ways of testing inherited behavior, including duplicating the tests, not testing the inherited behavior, and using a shared example."
+++

## Getting It On Record

Recently I asked on [Mastodon](https://ruby.social/@kevin_j_m/110668925217643586):

> How do you prefer to see methods that are inherited from another class unit tested?
>
> I'm applying this to #ruby code, but this question is meant to be language-agnostic to cover how you test inherited behavior in OO languages.

This is something that I've waffled on, and changed my opinion on, seemingly each time I have to deal with this. For me, a lot of the tension comes from choosing why I'm testing. Am I testing to verify the correctness of the code, or am I testing to document expected behavior?

Let's say we're managing a record collection.

```ruby
class VinylAlbum
  def shape
    :circle
  end

  def audio_technology
    :analog
  end

  def max_minutes
    46
  end
end
```

This all works until we get our first [extended play](https://en.wikipedia.org/wiki/Extended_play) (EP) record. To model that, we inherit from our `VinylAlbum` and override `max_minutes`.

```ruby
class Ep < VinylAlbum
  def max_minutes
    30
  end
end
```

We want to test our new `Ep` class. What do we do? Here's what respondents to my poll said.

{{< figure src="/img/mastodon_test_inherited_behavior_poll.png" class="mid" alt="A poll on Mastodon asking how people unit test inherited behavior. 67% say don't test it. 21% say something fancy (like a shared example). 13% say duplicate the test. 24 responses" >}}

## Duplicate The Test

This was the least popular response (holding aside "Other", which no one answered with). In this case, you can consider the inheritance to be an implementation detail. `Ep` responds to three methods, so I'm going to test how it behaves to receiving those three messages.

```ruby
RSpec.describe Ep do
  describe "#shape" do
    it "is a circle" do
      album = Ep.new

      expect(album.shape).to eq :circle
    end
  end

  describe "audio_technology" do
    it "is analog" do
      album = Ep.new

      expect(album.audio_technology).to eq :analog
    end
  end

  describe "#max_minutes" do
    it "is 30 minutes" do
      album = Ep.new

      expect(album.max_minutes).to eq 30
    end
  end
end
```

This is documenting the public API surface of the unit under test. I'm signaling when reading or running these tests that `Ep` has these methods and responds in this way to them.

The cost here is in maintaining these tests. Any changes to `VinylRecord` will mean changing not only its tests, but also `Ep`'s tests. These tests also run in the regression suite forever. That contributes time to run tests that you might consider to be testing the same thing twice.

### Testing Inherited Behavior Creates An Integration Test

I also received [feedback](https://ruby.social/@acute_distress/110672196493618303) to consider testing in this way an integration test. And my question was specifically about *unit testing*.

> only test what YOUR UNIT does and nothing else...Otherwise you are writing integration tests. It’s a grey area especially if you have added a private method to the child. Then you’ve got to ask yourself is your design correct.

I don't think this opinion is wrong. It may be a little at odds with how free I *tend* to see people be with the term "unit test". Particularly in the Ruby, and especially Rails, community. I've seen more than my share of what we call unit tests of models in a Rails app that hit the database. That test is not __technically__ a unit test. It's not only collaborating with another class. It's working with a whole other *external dependency*. Testing the behavior we're inheriting doesn't seem that out of line in comparison.

### Where's The Line?

Let's take this to the extreme though and look at a class that has no behavior defined in Ruby.

```ruby
irb(main):001:0> class Foo; end
irb(main):002:0> foo = Foo.new
irb(main):003:0> foo.public_methods.count
=> 64
```

Are you writing tests for those 64 methods you get from `BasicObject` and wherever else? I know I'm not.

Maybe instead we say we'll only test behavior that we explicitly inherit.

```ruby
irb(main):001:0> class Foo < ApplicationRecord; end
irb(main):002:0> foo = Foo.new
irb(main):003:0> foo.public_methods.count
=> 466
```

I'm not writing tests for all the behavior I get from Active Record either.

So why duplicate the tests? It seems here that the decision is to test behavior inherited from code that I own, or that's defined in my codebase. I'm trusting that Active Record is testing its behavior. I don't trust *my* behavior I inherit from `VinylRecord`.

## Don't Test

This was the most popular option. We trust Ruby to test the methods it defines. And we trust Active Record to test the methods it defines. As opposed to the duplication section, here we *also* trust `VinylAlbum` to test the methods it defines.

```ruby
RSpec.describe Ep do
  describe "#max_minutes" do
    it "is 30 minutes" do
      album = Ep.new

      expect(album.max_minutes).to eq 30
    end
  end
end
```

The `VinylAlbum` tests are already testing `shape` and `audio_technology`. `max_minutes` is different, and defined in the implementation of the `Ep` class, so we only test that.

## Something Fancy (Shared Examples)

This was the second most popular response. I suggested a [shared example](https://rspec.info/features/3-12/rspec-core/example-groups/shared-examples/) as an implementation of something fancy. I received no responses explaining another fancy method. So I'm assuming that all the respondents meant a shared example.

For this, we document the behavior that makes up a vinyl album.

```ruby
RSpec.shared_examples "an album" do
  describe "#shape" do
    it "is a circle" do
      expect(album.shape).to eq :circle
    end
  end

  describe "audio_technology" do
    it "is analog" do
      expect(album.audio_technology).to eq :analog
    end
  end

  describe "#max_minutes" do
    it "is 30 minutes" do
      expect(album.max_minutes).to eq 30
    end
  end
end
```

And then each of the classes that have this behavior include this shared example in their tests.

```ruby
RSpec.describe VinylAlbum do
  it_behaves_like "an album"
end

RSpec.describe Ep do
  it_behaves_like "an album"
end
```

This tests the full API surface of the classes inheriting this behavior. That may satisfy a need you feel where you'd otherwise duplicate the test. All that without actually having to write the tests over again.

There's just one problem: the `Ep` tests don't pass. That's because of the `max_minutes` method, which is overridden. An EP can't be 46 minutes; it can be no more than 30.

Now, we can get around that a couple of ways.

### Passing Parameters To The Shared Example

We can pass a parameter to the shared example.

```ruby
RSpec.shared_examples "an album" do |max_minutes|
  describe "#max_minutes" do
    it "is #{max_minutes} minutes" do
      expect(album.max_minutes).to eq max_minutes
    end
  end
end
```

And then specify that in each of our usages of the shared example.

```ruby
RSpec.describe VinylAlbum do
  it_behaves_like "an album", 46
end

RSpec.describe Ep do
  it_behaves_like "an album", 30
end
```

The tests now pass. However, I'd argue this is a bit confusing. Read the invocation of the shared example in a vacuum. You have no idea what that number has to do with the shared example. It causes the test to pass, but it's lacking explanatory information. You need to read the shared example to see what 46 or 30 refers to.

### Passing Context To The Shared Example

We can provide it a name by instead passing the context of that number to the shared example. The shared example will implicitly use an `expected_max_minutes` reference.

```ruby
RSpec.shared_examples "an album" do
  describe "#max_minutes" do
    it "has a maximum number of minutes" do
      expect(album.max_minutes).to eq expected_max_minutes
    end
  end
end
```

Each of the tests will define `expected_max_minutes` in a block.

```ruby
RSpec.describe VinylAlbum do
  it_behaves_like "an album" do
    let(:expected_max_minutes) { 46 }
  end
end

RSpec.describe Ep do
  it_behaves_like "an album" do
    let(:expected_max_minutes) { 30 }
  end
end
```

This adds clarity about what the number refers to when we call the shared example. However, the test description had to become more vague. We can't use it to say how many minutes it should be. Instead, we just say it has a maximum number of minutes, whatever that may be.

### Customizing The Shared Example

In either case, we needed to take some action to get our tests to pass. That's because behavior was overridden - the `max_minutes` method.

That can get unwieldy over time if you have a large inheritance tree. Whether that be with long branches or a large number of leaves on one branch. Any time there's a customization of the default behavior, we need to account for it. The base shared example needs to change and every use of the shared example needs to handle it.

We could use the shared example to test only the behavior that's consistent in all classes. Here that would mean `max_minutes` isn't in the shared example. Instead we'd test that within the implementations of both the classes. That can become difficult as more classes inherit and customize behavior. We'll continue pulling items from the shared example. We need to have the discipline to remember to test them in all the implementing classes.

We could also build multiple shared examples. There could be one that tests all the behavior that is common. Anything overridden could then move into a separate shared example. All the existing inheriting classes would use that new shared example. The new class that overrides it would test its customization itself. That again requires discipline to manage as the inheritance structure grows and changes.

## Setting The Record Straight

I'll admit - I'm drawn to duplicating the tests. I like using my tests as documentation. I like showing what to expect it to respond to, and how it will react.

I also don't think that stands up to logical scrutiny when compared to a lot of __other__ values I hold dear. That's where the tension comes from that led me to make this poll.

I don't test code that I don't own or control. As an aside, there's always an exception. But, in general, as already mentioned, I'm not testing the methods that come along "for free" on any Ruby object. Or testing the full API surface of a class inheriting from `ApplicationRecord`. For some reason, I'm called to test inherited behavior that's defined in the codebase.

I care very deeply about maintenance. I care about minimizing the negative impact that code today will have on my team tomorrow and next year. Those duplicated tests have a big cost the team needs to manage over the life of that code.

In that light, a shared example may feel like a welcome compromise. With a one liner, I've tested all those methods and no one's the wiser. Except - we are. We *still* need to maintain them. And those tests are still running, testing the same behavior, multiple times. That makes the feedback loop from our test suite slower.

Not adding the tests sounds like the choice I should make, in light of what I value. However, my interest in using tests to document behavior tells me otherwise. Perhaps thinking of those tests as integration tests will help tamp down the feeling. I'll keep it in mind going forward.

Thanks to everyone who responded to my poll. Are there more topics you'd like to see a poll about and have me write up my thoughts on? [Let me know](https://ruby.social/@kevin_j_m).

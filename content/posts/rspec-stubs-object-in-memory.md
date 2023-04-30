+++
title = "RSpec Stubs The Object In Memory"
date = 2024-09-13T15:00:10-04:00
tags = ["ruby", "rspec", "testing"]
summary = "Objects in memory may be farther than they appear"
description = "This article demonstrates a short-coming with using RSpec mocks in code which accesses objects out of a database."
+++

## Publish or Perish

Let's say we've been sitting on a handful of blog posts that are ready to publish, but we haven't released yet. This is a fictional story that in no way mirrors reality of any particular writer. Certainly not the author writing this post now. In a spirit of inspiration, we decide to write a small class to publish all these posts that exist in our system.

```ruby
class DraftPost
  def self.publish_all
    Post.draft.map(&:publish)
  end
end
```

Rather than actually publish them, we've successfully procrastinated. We have a *way* to publish them, eventually. While we're busy not actually publishing them, let's test this method works.

## Prolonged Publication Procrastination

We want to verify that we send the `publish` message to each of these posts. We don't actually want to execute the `publish` method. Perhaps that integrates with a third-party API. We don't want to manage [testing that interaction]({{< ref "testing-dependencies" >}}) here.

Instead, we'll stub out the response using [RSpec mocks](https://rspec.info/features/3-12/rspec-mocks/).

With that goal, we write our test.

```ruby
it "publishes all draft posts" do
  draft = Post.create(draft: true)
  allow(draft).to receive(:publish)

  DraftPost.publish_all

  expect(draft).to have_received(:publish)
end
```

This test *fails*. We put a breakpoint in our test. We confirm that a post with the same database ID as `draft` __does__ get the `publish` method called on it.

Even though these objects pass the equality check, the issue is that they are *not* the same object in memory. Compare the `object_id` of the `draft` object to the object pulled out of the database. They're different. The post published in `DraftPost.publish_all` does not have the stub applied to it.

The stub operates on that `draft` object in memory, and only that object. Even though the other object is equal to the `draft` object, it is not the *same* as the `draft` object. Because of that, our assertion does not pass.

## Any Instance Of

At this point, we may try to instead test what's returned from `publish_all` while still not having the `publish` method called on the posts.

```ruby
it "publishes all draft posts" do
  draft = Post.create(draft: true)

  allow_any_instance_of(Post).to receive(:publish).and_return("called publish")

  expect(DraftPost.publish_all).to eq ["called publish"]
end
```

Our test now passes. However, we have some drawbacks. First off, [RSpec itself](https://rspec.info/features/3-12/rspec-mocks/working-with-legacy-code/any-instance/) considers using `allow_any_instance_of` to be suspect.

> This feature is sometimes useful when working with legacy code, though in general we discourage its use for a number of reasons

Now, this doesn't mean it's *wrong*. But, I'd argue to only use it after carefully considering other options.

The other thing we changed here is our testing philosophy. Now we're testing what's returned by the method, rather than what the method does. And we did this by stubbing out what any Post should return when we call the `publish` method on it.

That again may be fine, but it may not test that we publish all the draft posts, which is our goal. Instead, it confirms what the `publish_all` method returns.

This may be hyperbolic, but if we changed the implementation to look like this:

```ruby
class DraftPost
  def self.publish_all
    ["called publish"]
  end
end
```

Now our test that asserts this method is calling `publish` still passes, but we're definitely not publishing any posts.

## Change the Implementation

Let's change `publish_all` so we pass all the posts to publish into it.

```ruby
class DraftPost
  def self.publish_all(posts)
    posts.map(&:publish)
  end
end
```

Now we can use the RSpec mock as we want.

```ruby
it "publishes all draft posts" do
  draft = Post.new(draft: true)
  allow(draft).to receive(:publish)

  DraftPost.publish_all([draft])

  expect(draft).to have_received(:publish)
end
```

This test passes, and as a bonus, we don't need to access the database in our test anymore. What a speed improvement!

Once again, this choice comes with some careful consideration. First off, modifying the implementation to satisfy the way we want to test it may be undesirable. I do want to point out that I don't think this is true generally. I do believe in using your tests as your first consumer of your code. If something is tough to test, it may be hard to use or confusing to understand. There's value in receiving that signal and deciding to change the code's design or structure in that case.

Stepping back here, passing in the posts makes the name of this class pretty meaningless. This should only work on draft posts. However, we can pass any kind of post (really, anything that responds to `publish`) and this will still work.

Maybe we want something that'll generically publish any kind of post. If so, we probably shouldn't have that inside a `DraftPost` class. If this should only operate on draft posts, then this may not be the choice we want to make.

Passing in the posts allows our stub to work. Passing in the posts may allow us to achieve some purity of using dependency injection. Passing in the posts may make this method less usable. Enough to the point where perhaps it shouldn't exist at all.

## Stub the DB Interaction

We still want to use the mock. We don't want to use RSpec features that the core team themselves recommend against. We don't want to change the implementation. It turns out we like the benefit of not needing to access the database. We can achieve all those goals by *also* stubbing out the database call.

```ruby
it "publishes all draft posts" do
 draft = Post.new(draft: true)
 allow(Post).to receive(:draft).and_return([draft])
 allow(draft).to receive(:publish)

 DraftPost.publish_all

 expect(draft).to have_received(:publish)
end
```

## Finally Publishing The Post

We confirm that we send the right message (`publish`) without invoking its implementation. We do give up some confidence in that we don't know for sure that `Post.draft` works how we expect. We can regain that confidence by having other tests specific for that method.

One could argue this gives us more flexibility. If the implementation of `Post.draft` changes, this test will continue to work. It can get posts from a database or third-party API. Our method doesn't care.

On the flip side, one could also argue that this test is tying us to a particular implementation. If `DraftPost.publish_all` changed to instead use `Post.where(draft: true)`, then this test will no longer work. Even though using the scope or using where will retrieve the same records from the database.

Unfortunately, there is no *right* answer. We must consider the trade-offs we're willing to make to achieve our goal. What ✨magic✨ you're interested in maintaining is up to you.

Keep in mind that RSpec mocks only work on that particular object in memory. If the implementation pulls the same record fresh out of the database, the mock will not apply. When using RSpec mocks in your tests, objects in memory may be farther than they appear.

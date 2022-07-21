+++
title = "Symmetric Spies: RSpec Test Organization"
date = 2022-09-17T08:53:21-04:00
tags = ["testing", "ruby", "rspec"]
summary = "Secret agents must keep followers off their trail. Your tests don't have to."
description = "There are many options to arrange test doubles with RSpec. This article proposes doing so in a way that provides a consistent test organization."
+++

## Special Operation Briefing

We're managing a network of secret agents. We need to send sensitive information to our [mole](https://en.wikipedia.org/wiki/Mole_(espionage)) who has infiltrated their target.

To do so, our agents will only send a scrambled version of the message. The mole will have to convert that into meaningful text.

```ruby
class Agent
  def pass_secret(message, to:)
    to.accept_secret(Cipher.encode(message))
  end
end
```

Our mole can receive this random-seeming string of characters. After deciphering the real meaning, they hold on to it for later.

```ruby
class Mole
  def accept_secret(message)
    messages << Cipher.decode(message)
  end
end
```

We want to make sure our message-passing scheme works. But we don't want to go through the laborious process of encoding and decoding these messages. It's computationally intensive and not the focus of our test. The `Cipher` is a collaborator in our class, but it's not the star of the show. We can test that implementation, and bear those costs, in the unit tests of the `Cipher` class. We're testing the `Agent` class today, not `Cipher`.

## Efficient Espionage Expectations

We'll avoid encoding the message, but make sure it happens, by asking RSpec to [expect the message](https://relishapp.com/rspec/rspec-mocks/docs/basics/expecting-messages).

```ruby
it "encodes the message" do
  agent = Agent.new
  mole = Mole.new

  expect(Cipher).to receive(:encode).with("hello")
  agent.pass_secret("hello", to: mole)
end
```

Our test passes. We've verified `pass_secret` encodes the message provided to it. We can move on; however, I'd prefer a different test construction.

## Organization Infiltration

The domain we're testing is about obfuscation, but I want as few surprises as possible in my tests. I like to follow a familiar pattern of how to construct my tests, so they read from top to bottom in a consistent way.

Some people refer to this as [four-phase tests](http://xunitpatterns.com/Four%20Phase%20Test.html) (setup, exercise, verify, teardown). Others call it the [AAA pattern](https://wiki.c2.com/?ArrangeActAssert) (arrange, act, assert).

Name aside, we want a convention that we can rely on. That makes it easier to scan a test and understand where we are in the process as a reader. The difficulty with our existing test is it breaks that convention. We state what we want to verify, or assert, before exercising, or acting on, the unit under test.

```ruby
it "encodes the message" do
  # setup
  agent = Agent.new
  mole = Mole.new

  # verification
  expect(Cipher).to receive(:encode).with("hello")

  # exercise
  agent.pass_secret("hello", to: mole)
end
```

## Double Agent

We can change our test to follow this pattern by using [spies](https://relishapp.com/rspec/rspec-mocks/docs/basics/spies). Our `Cipher` will be a [partial double](https://relishapp.com/rspec/rspec-mocks/docs/basics/partial-test-doubles), and we'll assert we call the [method we're spying on](https://relishapp.com/rspec/rspec-mocks/docs/basics/spies#spy-on-a-method-on-a-partial-double).

```ruby
it "encodes the message" do
  # arrange
  allow(Cipher).to receive(:encode)

  agent = Agent.new
  mole = Mole.new

  # act
  agent.pass_secret("hello", to: mole)

  # assert
  expect(Cipher).to have_received(:encode).with("hello")
end
```

In our `allow` statement, we have no constraints on what we pass to `encode`. That's fine in this case - we tighten up that restriction in the assertion. There we make sure not only that we call the `encode` method on `Cipher`, but also with the right input ("hello").

I call this symmetry, or symmetric spies. We start our test by `allow`ing the message to be received. We end our test by resolving to `expect` that the message has been received.

## Phase 2 of the Operation

As of now, we've verified that our agent encodes the message. We also want to make sure the mole receives the message. The __encoded__ message. And we want to verify that without actually running our message through the encoder.

Luckily, RSpec can help us out here too. We can use `with` and `and_return` on our test double. With those two methods, the double will provide the specified output, given that input.

```ruby
allow(Cipher).to receive(:encode).with("hello").and_return("mystery")
```

`Ciper.encode` will only return `"mystery"` if we call it with `"hello"` as an argument.

Let's use this to write a test that makes sure the mole gets our encoded message.

```ruby
it "sends the encoded message to the receiver" do
  agent = Agent.new
  mole = Mole.new

  allow(Cipher).to receive(:encode).with("hello").and_return("mystery")
  allow(mole).to receive(:accept_secret)

  agent.pass_secret("hello", to: mole)

  expect(mole).to have_received(:accept_secret).with("mystery")
```

## Breaking the Code, or Symmetry

You may have noticed something in that last test that feels like it betrays the intent of symmetry. We are spying (with partial doubles) on two methods. We only expect to receive one. That's not symmetrical at all!

We already tested that `pass_secret` will call `encode`. We have existing coverage verifying that. The focus of __this test__ is ensuring the encoded message gets to the mole. We're using our partial double on `Cipher` to avoid a method call that takes a long time. Another use case may be to avoid a side-effect that we don't want to incur in our test. We're not using it to verify system behavior.

We *are* setting up the mole as a partial double to verify behavior. We want to make sure the agent interacts with the mole as expected. Because of that, we verify that we call the right method, with the right input, to the mole. We don't verify how we encode the message.

There would be nothing *wrong* with asserting that we call `encode` on `Cipher`. It's at best duplicative, given our prior test. Yet, it does take away focus on the intent of our test, which is verifying communication with the mole.

The goals of the test drive what expectations we make in that test. The goal of this test isn't about encoding; it's about communicating with the mole. Seeking symmetry isn't having the same number of doubles or spies and expectations. It's organizing the test in a manner consistent with the four-phase test or AAA pattern.

## Debriefing

We may not *need* the symmetry within an individual test. Keeping it, and our test organization scheme, in mind helps us write readable, predictable tests. The element of surprise may be critical to a spy, agent, or mole in the field. It's less of an asset in your test suite.

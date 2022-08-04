+++
title = "Fake It While You Make It - The Proposal"
date = 2022-08-03T21:25:22-04:00
tags = ["conference-proposal", "conference", "railsconf"]
summary = "Presented at RailsConf 2020"
description = "The proposal for a talk I gave at RailsConf 2020."
+++

## Abstract

We all write code to interface with external systems, like a web service or a message queue. Can you confidently write tests without requiring the system as a dependency? How can you shield users of your code from the inner workings of the interface? Explore one attempt to answer these questions.

There's no shortage of tools at your disposal to solve these problems. This talk will introduce some available options, provide guidance on when one approach may be more appropriate than another, and discuss how to use these tools together to ease the testing process.

## Details

This talk will follow one effort to test interactions with an external dependency that evolved over time, as the nature of the interaction became clearer, and as the complexity of the work around the dependency increased.

### Testing Strategies

In the process of testing against the dependency, a variety of tools were used, each of which will be presented with a sample implementation, after which I will discuss their benefits and challenges.

#### Direct Interaction with Dependency

##### Benefits

* Confidence that your code will work the same in your test as in production.
* Lowest barrier to entry.

##### Challenges

* May negatively affect the performance of the test suite.
* Can be non-deterministic, depending on the nature of the dependency.
* Subject to any constraints the dependency may impose upon your system, such as an HTTP API that rate limits you after a certain number of requests.

##### When To Use

* When unfamiliar with the dependency.
* When exploring new features or functionality.

#### Stubbing Responses

##### Benefits

* Promotes deterministic behavior in the response of the dependency for your test.
* Allows for the test to succeed without the need for the dependency at all, limiting the performance overhead of the tests on the test suite.

##### Challenges

* Must know the response structure.
* Must ensure that the structure and data returned continues to mirror the reality of what the actual dependency provides.

##### When To Use

* When you have a stable interface.
* When the response surface itself is small, to mitigate verbosity in your test itself.

#### Building a Fake

##### Benefits

* Provides a full-stack, complete test interaction.
* Limited noise in the test itself, in terms of setup verbosity.
* Allows you to build in as much or as little complexity as you need to serve the needs of your testing scenarios in an isolated, reusable location.

##### Challenges

* Must ensure that the structure and data returned continues to mirror the reality of what the actual dependency provides.
* Must consider how to test and verify that the implementation of your fake is performing as expected.

##### When To Use

* When you need confidence in the communication mechanisms or protocols themselves.
* When your test requires a multi-step interaction with the dependency, particularly where the dependency may need to store state, such as testing an OAuth handshake.

#### Fixture Data

##### Benefits

* Provides a truthful representation of an interaction at a moment in time.
* Provides a complete picture of a full response.
* Requires the dependency only long enough to capture the fixture data.

##### Challenges

* Can act as a mystery guest, where it’s not clear where the data came from or why the test is responding in that way, given that the data is in a different location and not obvious within the test itself.
* Given that the fixture is a representation at a particular moment in time, it’s not guaranteed that the dependency continues to react with the same request in the same way.
* Requires periodic access to the system to continually refresh these snapshots.

##### When To Use

* When you need a complete response.
* When your dependency is accessible for the generation and refreshing of this data.
* When you must limit your impact on the dependency itself, such as not continually creating a new instance of a resource via a POST action to an HTTP API on an actual system every time the test is run.

### Architecture Learnings from this Investigation

Isolating the interaction with the dependency will be a main takeaway from this exploration. The focus will be the benefits that this provides from a testing perspective, as well as general extensibility towards the overall architecture of your application.

Finally, once the full public API of the system is defined and it's ready to be integrated as part of a larger system or shared with the world as a gem, we will discuss creating a "test mode" for your component. A great example of this is Sidekiq's [testing mode](https://github.com/mperham/sidekiq/wiki/Testing). This will cover tips for how to build this functionality, how to expose it to consumers of your component, and the challenges associated with creating and maintaining this feature.

## Pitch

Interfacing with external systems has always been considered a challenging problem. It's also becoming a more common task, as we continue to interconnect with different systems. Microservice architectures make this even more pronounced, as we orchestrate many small, separate systems together to serve as the backbone of our overall technology solution.

Much of my career has been spent identifying, implementing, and maintaining interfaces. My goal is always to make it easy for other developers to interact with the system, and provide them with the information they need to complete their task without needing to be overwhelmed with technical details. I also developed, and continue to maintain, a gem that provides functionality to test interactions with another gem without the need for mocking or stubbing. I've used that experience to inform my choices to ease testing of other interfaces I continue to create. The goal of this talk will be to share how a variety of testing practices can be used in concert when working with external dependencies.

## Bio

Kevin spent more time investigating, refactoring, and rewriting tests for the project that prompted this talk than he did writing the actual implementation.

Kevin lives near Boston, where he is a Software Developer at The Gnar Company.

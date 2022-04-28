+++
title = "Testing Dependencies: Fake It While You Make It"
date = 2022-06-05T21:15:06-04:00
tags = ["testing", "ruby"]
summary = "Surveying options for testing third-party dependencies"
description = "There are many ways to test usage of third-party dependencies. This article explores four approaches: direct interaction, stubbing, faking, and fixtures."
+++

## Sweathr

Is it feeling chilly outside where you live? Should you put a sweater on before you head outside? These questions used to be hard to answer. No more! Today we're going to be talking about Sweathr. This new, totally not fake, website exists to tell you if it's sweater weather where you live.

I debuted this revolutionary product at [RailsConf 2020](https://youtu.be/iEfpAp2sqiw). It communicates with a weather HTTP API to receive temperature information.

Here is the core business logic of our application. We define sweater weather to be from 55-65 degrees Fahrenheit.

```ruby
class Location
  def sweater_weather?
    uri = URI(conditions_url_for_zip_code)
    result = Net::HTTP.get(uri)
    response = JSON.parse(result)

    feels_like = response.dig("current_observation", "feelslike_f")
    feels_like.to_f.between?(55, 65)
  end
end
```

We'd like to test the `Location#sweater_weather?` method. Today, we're going to investigate four options to do so with the API call in the method.

* [Direct Interaction](#direct-interaction)
* [Stub](#stub)
* [Fake](#fake)
* [Fixture](#fixture)

### Direct Interaction

As a starting point, we can write some tests that use the API directly.

```ruby
it "is time to break out the sweater" do
  location = Sweathr::Location.new(zip_code: "02108")

  expect(location.sweater_weather?).to be true
end
```

This will give us confidence that we are using the dependency correctly. Our test has full production parity. It will work the same in our test suite as it will deployed for our end users.

It comes with significant downsides though. The most glaring is that it's non-deterministic. This test succeeding __literally depends__ on the weather in that location. It's likely to fail often - and not for actionable reasons. It will fail because it happens to be too hot or too cold outside at the given moment this test runs.

Tying this test to our dependency may be slow. We need to wait for the HTTP request and response to resolve before our test passes. We're also subject to the constraints of our dependency. With a HTTP API, we may have rate limits. That can become problematic when this test runs over and over again, locally and on CI. Our massive development team at Sweathr need to run the test suite often, and we may get rate limited.

You may think that this is a foolish approach. I'll agree, it's unlikely
the best choice in this example. Don't dismiss this strategy entirely though. You're probably using it in your own test suite! I'm yet to see a test suite for a Rails app with persistence that doesn't interact with the backing store. We even write things we call *unit* tests which call out to the database (gasp). This is directly interacting with our dependency. And we've largely decided, as a community, that we accept this. We are directly interacting with our dependency in our test.

### Stub

We can avoid managing HTTP communication with our dependency. Instead, we'll stub out how we expect it to respond. In this case, we'll use [webmock](https://github.com/bblimke/webmock) to stub out a response at the HTTP layer.

```ruby
it "is time to break out the sweater" do
  api_response = {
    current_observation: {
      feelslike_f: "55.0"
    }
  }.to_json

  stub_request(:get, url).to_return(status: 200, body: api_response)

  location = Sweathr::Location.new(zip_code: "02108")

  expect(location.sweater_weather?).to be true
end
```

This solves our determinism problem as well. In the world of this test, the weather is always 55 degrees at this location. It's faster than communicating over HTTP - it doesn't need HTTP at all. We can run this test in the middle of the woods with no internet connection and it'll pass without a problem.

There are still trade-offs we're making with this approach. First of all, we must know the response structure. We may need to [directly interact](#direct-interaction) with the dependency first to confirm the structure.

Just as important, we need to keep that structure up to date. If the weather API starts sending data in a different format, this test is going to continue to pass. We'll only find out there's an issue on production. That's less than great - I want my test to tell me there's a problem.

Stubbing out a response from your dependency in tests can be a great choice if you have a stable interface. You're avoiding using the dependency entirely. As such, you're taking on the responsibility to make sure it still reflects reality. That's easier to manage if the interface and data structure doesn't change.

### Fake

Stubbing out the response may be too magical for you. A more tangible approach may be to build our own weather API. For test purposes we'll communicate with that API, rather than the real one.

Let's build our own weather API quick:

```ruby
class FakeWeather < Sinatra::Base
  get "/api/:key/conditions/q/:zip_code.json" do
    json current_observation: { feelslike_f: "56.0" }
  end
end
```

We can use the `FakeWeather` API rather than the real one using a tool like [capybara_discoball](https://github.com/thoughtbot/capybara_discoball).

```ruby
it "is time to break out the sweater" do
  location = Sweathr::Location.new(zip_code: "02108")

  Capybara::Discoball.spin(FakeWeather) do |s|
    location.endpoint_url = s.url

    expect(location.sweater_weather?).to be true
  end
end
```

Within the `Capybara::Discoball.spin` block, we're communicating with `FakeWeather`. We're not issuing a request to the actual weather API. Because that fake always says it's 56 degrees, we still have a deterministic test. We can control how much flexibility we have in our fake. We can update it so that different locations get different temperatures if we'd like. That complexity lives in the `FakeWeather` class. It's not contained within our individual tests that communicate with it.

Using the fake puts us back to communicating over HTTP. It happens to be communicating with a local HTTP server, but it's still using the protocol.

We still have the same consistency issues we had with stubbing. We need to make sure the responses our fake is providing mimic the responses of the real API and keep that up to date.

Using a fake may be beneficial when we need to test the communication mechanism, or protocol. It can also be useful when testing a multi-step interaction, like an OAuth flow. Rather than stubbing out each API call in that flow in a test, we can build a fake that has endpoints to respond to each of those requests.

### Fixture

We have another option to capture a real interaction with the dependency. We won't need to build out a fake version of our dependency. We won't stub out a made-up response. We can store that interplay and reload it from that file for future runs. We call that file a fixture.

We'll use a tool called [vcr](https://github.com/vcr/vcr) to capture real HTTP requests and responses, storing and using them from then on. VCR's language and terminology leans heavily into its real-world analogue. If you don't know what a VCR is, it's a box we used to hook up to TVs to watch movies before DVD players. If you don't know what a DVD player is, it's a box we used to hook up to TVs to watch movies before streaming.

```ruby
it "is time to break out the sweater" do
  location = Sweathr::Location.new(zip_code: "02108")

  VCR.use_cassette("temp_needs_sweater") do
    expect(location.sweater_weather?).to be true
  end
end
```

The first time this test runs, it makes an actual HTTP request to the weather API. VCR saves the request and response information in a file called "temp_needs_sweater.yml".

The next time this test runs, VCR uses the request and response from that YAML file. No HTTP requests.

This fixture is an absolutely true representation of an interaction with the dependency. It's true as of the moment in time it generates the fixture. Generating the fixture in a test and running the code in production should react the same.

Unfortunately, the time we generated that fixture quickly turns into the past. To ensure that correctness, we must update the fixture. Recording new fixtures lets us validate the continued correctness of our test. This is in contrast with the stub or the fake. Even though we *can* update them, we still need to take the action to refresh these fixtures.

## Depending on Tests

We may be writing a test to serve a variety of needs. The dependency testing strategies listed above serve different needs. 

When we're using the dependency for the first time, we want to make sure the code we're writing *works*. We're not familiar enough with it to guess how it will respond. We need the actual dependency. Interacting with it directly is the best approach to give us experience with it. It may prove to be good enough for a long time, like in the case of calling out to a database.

The overhead of accessing the dependency may prove to be too onerous. It may make the test unreliable. It may make the test too slow. We can stub out the response. This depends on how confidently we understand how the dependency will respond. It's helpful when we have some way to alert us that the assumption we've made in our stub is now invalid.

A fake version of the dependency adds back some complexity. We need to generate the fake and communicate over the original protocol. The fake tests our usage of the dependency at a higher level. We isolate the complexity of the interactions inside the fake.

A fixture captures a real interaction with the dependency. We must refresh that interaction so it reflects the dependency's current reality.

There is no "right" answer to how to test against third-party dependencies. It's all [trade-offs]({{< ref "/to-change-or-not-to-change" >}}), and it's up to us to determine which risks we're willing to accept for the greatest benefit.

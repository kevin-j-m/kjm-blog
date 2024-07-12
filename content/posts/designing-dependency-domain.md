+++
title = "Designing A Dependency's Domain"
date = 2022-06-19T20:00:00-04:00
tags = ["ruby", "software-design"]
summary = "Turning one five line method into two classes and a module"
description = "This article decomposes an interaction with an external dependency into its component parts."
+++

## Sweathr

In our [last post]({{< ref "testing-dependencies" >}}) I introduced you to Sweathr. We've revolutionized the sweater weather prediction landscape forever. We evaluated many ways to test the core business logic, which called out to a HTTP weather API.

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

Testing this code involved a fair amount of work. Any of the options had significant trade-offs we needed to weigh. Part of the reason testing this method was so much work is because it involves a dependency, the weather API. Part of the reason is, even though it's a small method, it's doing a lot of things.

In this post, we're going to decompose this method into three different units. Along the way, we'll talk about *why* we're doing this.

### API Client

Let's start at the top. We're going to have a separate class that will handle HTTP communication with the weather API. We know we're going to have other code related to this API, so we'll put it in its own module.

```ruby
module Sweathr
  module Weather
    class Api
      def current_conditions(zip:)
        uri = URI("#{auth_uri}/conditions/q/#{zip}.json")

        JSON.parse(Net::HTTP.get(uri))
      end

      private

      def auth_uri
        "https://real-weather-api.gov"
      end
    end
  end
end
```

This class is solely focused on communicating with the dependency. There's subtle complexity here though. This class needs to determine how it's going to return the response. Right now, this is converting the response body JSON and returning the hash. This means that it's choosing to swallow the status code, headers, or other data the API is telling us. That's a choice, and it's isolated in this class. If we need access to other information, we know where we need to go to get it. Documenting, or iterating on, these types of decisions has a well-defined home.

#### API Client Testing

To test this, we can use any of the methods from our [last post]({{< ref "testing-dependencies" >}}). However, I would advocate for one that, at least at some point, issues a real request to the API. That is the focus of this class, and as such, we want to make sure it's performing as expected. Using a fake or a stub here wouldn't give us a lot of confidence in testing our implementation.

Making a direct API call or using a fixture that's updated has an issue: determinism. Remember, this is a weather API - and the weather changes. We can't assert that it's sunny and 70 degrees and have it pass all the time.

Luckily, we don't have to. Instead, what we can focus on is the *structure* of the data, rather than the *contents*.

```ruby
it "retrieves the current conditions" do
  result = api.current_conditions(zip: "02108")

  expect(result).to include(
    "current_observation" => a_hash_including(
      "feelslike_f" => a_kind_of(String)
    )
  )
end
```

It feels weird asserting we get a String in a language like Ruby sometimes, I'll grant you that. But, it doesn't matter to us what the temperature is. We want to validate that we can get back a temperature. We can do that by making sure it's nested inside the current observation with the key we expect.

This test is flexible where it can be - what the temperature is. It's also rigid where it has to be - we get a response and the data is in a format we expect.

This can serve as an early-warning system for upcoming components in our design. If this test fails, we'll need to change other areas of our domain.

Running this test as written [directly interacts]({{< ref "testing-dependencies#direct-interaction" >}}) with the dependency. This test will fail without an internet connection, or when the API is down. We could instead use a [fixture]({{< ref "testing-dependencies#fixture" >}}) that's regularly updated to mitigate that concern.

### Data Representation

The next piece we're going to build will take in that response body and expose the data of interest to us. First things first, we need to give it a name. This data is telling us what the weather is in a place. We'll call it the current weather conditions.

```ruby
module Sweathr
  module Weather
    class CurrentConditions
      def initialize(results)
        @results = results
      end

      def feels_like_f
        @results.dig("current_observation", "feelslike_f").to_f
      end
    end
  end
end
```

All we care about is knowing how hot or cold it feels outside. So, that's the one public method we'll expose. The API provides the number as a string, and we'll use this method to convert it to a float, so it's in a format we expect. This class provides us a location to expose more data that comes back should we need it in the future.

#### Data Representation Testing

We can unit test this using conventional and familiar means. There's no need to interface with the API at all.

```ruby
describe "#feels_like_f" do
  it "gives the feels like temp in Fahrenheit" do
    conditions = Sweathr::Weather::CurrentConditions.new(json)

    expect(conditions.feels_like_f).to eq 55.5
  end
end
```

Because this is so targeted, we can also dig into very specific edge cases. We can explore how we handle data we might receive from the API without needing to interact with the API. How will this respond to receiving a string that can't convert to a float? What if there is no current observation key in the hash? We can provide a hash that meets those criteria and test that with minimal overhead.

### Domain Module

We're going to tie these two components together in a module. This module will be our external API by which we consume this work.

```ruby
module Sweathr
  module Weather
    def self.client
      @client ||= Api.new
    end

    def self.current_conditions(zip_code:)
      Sweathr::Weather::CurrentConditions.new(
        client.current_conditions(zip: zip_code)
      )
    end
  end
end
```

As a consumer of this, I don't need to worry about knowing all the underlying classes involved. Let's see how we can use this in our existing `Location` class.

## Usage

Interfacing with our weather API is now abstracted into a separate module. We can re-write our `Location#sweater_weather?` method to use it.

```ruby
class Location
  def sweater_weather?
    current = Sweathr::Weather.current_conditions(zip_code: @zip_code)

    current.feels_like_f.between?(55, 65)
  end
end
```

More of the focus in this method is now on the business logic. This method retrieves the current weather conditions, and determines if it feels like sweater weather. Accessing that information from the weather API is still important. But we don't need to understand that complexity when we're reading this method.

This separation gives us a new option for testing our interactions with the weather API. We'll explore that next.

## Test Mode

### Fake Client

Let's borrow a strategy from our prior post. We're going to build a [fake]({{< ref "testing-dependencies#fake" >}}). But, we don't need to fake out the HTTP traffic. We're going to build a class that responds to the same methods as `Sweathr::Weather::Api`, but doesn't make a HTTP call.

```ruby
class FakeWeatherClient
  def current_conditions(zip:)
    {
      "current_observation" => {
        "feelslike_f" => @results[zip_code]
      }
    }
  end
end
```

This client needs to responds to the same methods as the real client, so it can stand in for the original. Helpfully, it can also have extra behavior. We're going to add a method that allows us to specify the weather in a location.

```ruby
class FakeWeatherClient
  def add_condition(zip_code:, temp_f:)
    @results[zip_code] = temp_f
  end
end
```

Now, we can control the weather anywhere in the world!

### Test Mode

We need a way to specify that we want to use this fake client. We'll do so by making one change to our domain. We're going to add the ability to set the client we want to use to consume the weather API.

```ruby
module Sweathr
  module Weather
    def self.client
      @client ||= Api.new
    end

    def self.client=(client)
      @client = client
    end
  end
end
```

We don't want to have to set this client in all our tests - and we certainly don't want to forget to set it back. We'll instead build a module that will take care of this for us.

```ruby
module Sweathr
  module Weather
    module Testing
      def self.enable!
        Sweathr::Weather.client = FakeWeatherClient.new
        yield
        Sweathr::Weather.client = Sweathr::Weather::Api.new
      end
    end
  end
end
```

This method expects a block, and for the [duration of the block]({{< ref "temporary-state-in-tests" >}}), our fake client is the client it uses. At the conclusion of the block, it returns the client to one that will execute API requests.

Now in testing our `Location` class, we can define the state of the world (or at least the weather) in the setup of our test.

```ruby
it "is time to break out the sweater" do
  Sweathr::Weather::Testing.enable! do
    Sweathr::Weather.client.add_condition(
      zip_code: "02108",
      temp_f: "56.0"
    )

    location = Sweathr::Location.new(zip_code: "02108")

    expect(location.sweater_weather?).to be true
  end
end
```

We're able to succinctly state that the temperature in zip code 02108 is 56 degrees at the top of the block. Then we exercise how our method under test responds to that scenario.

## Domain Development

We needed to undertake a lot of work to allow for that test to exist. We decomposed a five line method into two classes and a module. We added in *another* module to test our weather API. And we drive those tests by a fake version of one of the new classes we wrote.

In doing so, we gained the ability to test our interaction with the dependency in greater detail. We have logical extension points for building in new functionality. We know where to add a method to consume new API endpoints. Or where to expose more data from the current weather conditions. We removed the need to handle HTTP traffic to test our application's business logic.

Was it worth it? For this contrived example, maybe not. As your usage of a dependency expands, having this separation has more merit. Defining a separate domain for the dependency clears up where the responsibility of the dependency ends and your usage of it begins. It makes it easier to reuse common interactions with the dependency. It provides clear seams to add in future functionality. It may be worth considering as you organize your next interaction with a new dependency.

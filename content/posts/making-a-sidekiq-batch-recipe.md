+++
title = "Making a (Sidekiq) Batch Recipe"
date = 2024-04-13T18:00:22-04:00
tags = ["ruby"]
summary = "Don't stew over this too much"
description = "This post provides an introduction to using Sidekiq batches, which is available in Sidekiq Pro."
+++

## The Right Number of Cooks in the Kitchen

Today we're going to make a stew. The recipe has three steps that can all run independently. But when they're done, their output needs to come together to finish the stew.

We'll set each step up as a separate Sidekiq job. The details of each step aren't important for this demonstration.

```ruby
class GetRawVeggiesWorker
  include Sidekiq::Worker

  def perform; end
end

class GetBaconWorker
  include Sidekiq::Worker

  def perform; end
end

class GetCupOfSoupWorker
  include Sidekiq::Worker

  def perform; end
end
```

We can enqueue these to run by themselves no problem. However, we need to know when they're all done so we can finish our recipe. We can group these together using a [Sidekiq Pro](https://sidekiq.org/products/pro.html) feature: batches.

We'll write a series of RSpec tests to explore how to use batches to make our recipe.

## Tracking Kitchen Progress

We'll start by creating a batch, and adding our recipe steps to it as jobs. Just like a Sidekiq job has a `jid` (job ID), a batch has a `bid` (batch ID). We can use that `bid` to check on the batch's status thanks to the aptly-named `Batch::Status` class.

```ruby
it "adds jobs to a batch" do
  recipe = Sidekiq::Batch.new

  recipe.jobs do
    GetRawVeggiesWorker.perform_async
    GetBaconWorker.perform_async
    GetCupOfSoupWorker.perform_async
  end

  batch_status = Sidekiq::Batch::Status.new(recipe.bid)

  expect(batch_status).to have_attributes(
    total: 3,
    pending: 3,
    complete?: false,
  )
end
```

After making our batch and checking on the status, we see there are three jobs, but none of them ran. That is because this is in a test, and we're using the [Sidekiq fake adapter](https://github.com/sidekiq/sidekiq/wiki/Testing#testing-worker-queueing-fake) by default.

Let's update our test to run the batch jobs [inline](https://github.com/sidekiq/sidekiq/wiki/Testing#testing-workers-inline):

```ruby
it "runs the workers in the batch in inline mode" do
  recipe = Sidekiq::Batch.new

  Sidekiq::Testing.inline! do
    recipe.jobs do
      GetRawVeggiesWorker.perform_async
      GetBaconWorker.perform_async
      GetCupOfSoupWorker.perform_async
    end
  end

  batch_status = Sidekiq::Batch::Status.new(recipe.bid)

  expect(batch_status).to have_attributes(
    pending: 0,
    complete?: true,
    total: 5,
  )
end
```

The jobs executed and the batch is complete. Note that the total number of jobs is five, even though we enqueued three jobs. Interesting! Let's leave that aside as we explore what to do now that we have a batch that completes.

## Calling (back) next steps

The reason we created a batch was so we could do something with the results of our jobs when they finished. We wanted them to run independently, so we can take advantage of parallel execution. But we need to have the system take action once they're all done.

Sidekiq batches respond to [callbacks](https://github.com/sidekiq/sidekiq/wiki/Batches#callbacks). We'll focus on two of the three available callbacks: complete and success.

```ruby
class ActingLessonCallback
  def on_complete(status, options)
    puts "#{options['name']} went to Craft Services"
  end

  def on_success(status, options)
    puts "Baby, you've got a stew goin'"
  end
end
```

Each callback method accepts two arguments. One for the status, and another for a set of options. We're using those options in the `on_complete` callback to pass a name to the status message.

Also, I'm sorry I misled you earlier. This isn't really about a recipe. It's an [acting lesson](https://www.youtube.com/watch?v=oDOffqDsV5Q).

{{< rawhtml class="float" >}}
<iframe width="560" height="315" src="https://www.youtube.com/embed/oDOffqDsV5Q?si=dHN-tl96__zJpc_0" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>
{{< /rawhtml >}}

## Getting an audition callback

Now we know how to use a callback and we created a class to house our callbacks. Let's put it to use by telling our batch about it, and seeing how they get used.

```ruby
it "runs a callback" do
  acting_lesson = Sidekiq::Batch.new
  acting_lesson.on(:complete, ActingLessonCallback, "name" => "Kevin")
  acting_lesson.on(:success, ActingLessonCallback)

  Sidekiq::Testing.inline! do
    expect do
      acting_lesson.jobs do
        GetRawVeggiesWorker.perform_async
        GetBaconWorker.perform_async
        GetCupOfSoupWorker.perform_async
      end
    end
    .to output("Kevin went to Craft Services\n"\
               "Baby, you've got a stew goin'\n").to_stdout
  end
end
```

We registered our callbacks with the batch using the `on` method.

Remember those two additional jobs in the prior example? Where we enqueued three jobs, but the total count was five? Those extra jobs were these callbacks firing. Even though we didn't register any callbacks, the events still fired.

In this test our batch executed, triggering both callback events. As expected, the callbacks output their message. Completion! Success!

## A complete definition of success

While success and complete may sound similar, they have specific and different meanings. The success callback is perhaps the more obvious one. It triggers when the jobs in the batch have completed successfully.

That means that a batch can complete and not be successful - leading us to the `on_complete` callback. That fires when all the jobs have executed. Some of the jobs could have failed. Some may be in the retry queue. But they have run at least once.

To show this, let's create a few more Sidekiq jobs. We'll join the steps of our acting lesson together in one job. And we'll create one more that always fails.

```ruby
class ActingLessonWorker
  include Sidekiq::Job

  def perform; end
end

class HugeMistakeWorker
  include Sidekiq::Job
  sidekiq_options retry: false

  def perform
    raise "I think I'd like my money back"
  end
end
```

Now we'll run another test, creating a batch with these two jobs. At the end, we'll check the status of the batch.

```ruby
it "completes once each job has run once, regardless of success" do
  drama_coach = Sidekiq::Batch.new

  Sidekiq::Testing.inline! do
    expect do
      drama_coach.jobs do
        ActingLessonWorker.perform_async
        HugeMistakeWorker.perform_async
      end
    end.to raise_error "I think I'd like my money back"
  end

  batch_status = Sidekiq::Batch::Status.new(drama_coach.bid)

  expect(batch_status).to have_attributes(
    complete?: true,
    total: 4,
    pending: 1,
    failures: 1,
    success_pct: 75.0,
  )
end
```

Even though not all the jobs were successful, the batch still reports itself as complete. We can see the failure, and the job that failed as pending. We can also look at the success percentage of the batch to understand that not all the jobs succeeded.

## Dress rehearsal implications

Something to be mindful of when running your batches in tests is when the callback will fire. To really have some fun, let's do something I don't reach for often: use a global variable.

```ruby
$global = 0
```

We'll create a job that increments the global.

```ruby
class CounterWorker
  include Sidekiq::Job

  def perform
    $global += 1
  end
end
```

We'll have a callback after the batch succeeds. It outputs how many times the jobs incremented the counter.

```ruby
class BatchCallback
  def on_success(status, options)
    puts "Jobs run: #{$global}"
  end
end
```

In our test, we have a batch that fires the callback and enqueues the counter job twice.

```ruby
 it "runs the success callback after the first job is run with inline test mode" do
  batch = Sidekiq::Batch.new
  batch.on(:success, BatchCallback)

  Sidekiq::Testing.inline! do
    expect do
      batch.jobs do
        CounterWorker.perform_async
        CounterWorker.perform_async
      end
    end.to output("Jobs run: 1\n").to_stdout
  end

  expect($global).to eq 2
end
```

Our global says both jobs ran; however, the output from our callback says only one ran. Both are correct - at different points in time! Remember, these jobs in this *test* are performed inline. The first counter job was enqueued and run, incrementing the counter. Then Sidekiq checked to see if any other jobs were in the batch. At this point there aren't, so it triggers the callbacks. The success callback outputs that one job ran.

Then, the second job in the batch was enqueued and run, incrementing the counter. The end of our test verifies that the global variable is currently set to two. The callback is not executed again because it already ran.

If you're running batches in tests, want to test the callback of a batch, and the callback depends on the state or results of all the jobs in the batch, you may end up with a surprising result.

## Bulking up mid-rehearsal

If you really need to test this, you *could* work around this by changing how you enqueue the jobs in your batch. Using [bulk queueing](https://github.com/sidekiq/sidekiq/wiki/Bulk-Queueing) will get you the result you expect.

```ruby
it "runs the success callback after all jobs run when pushed in bulk with inline test mode" do
  batch = Sidekiq::Batch.new
  batch.on(:success, BatchCallback)

  Sidekiq::Testing.inline! do
    expect do
      batch.jobs { CounterWorker.perform_bulk([[], []]) }
    end.to output("Jobs run: 2\n").to_stdout
  end

  expect($global).to eq 2
end
```

This also enqueues two jobs. When run with the inline test adapter, they both run before the batch callbacks fire. So, both the callback and the global are consistent.

I point this out to show you can do this, and also to perhaps introduce bulk queueing to you. However, if your tests rely on this, I would advocate to reconsider your testing strategy. Test the callback in isolation to verify that part of your code. Trust Sidekiq to manage the orchestration of firing the callback for the batch.

## A draining performance

You may recall that our first test didn't run the jobs because we were using the fake test adapter. I have one other word of caution to point out that I noticed using the fake adapter. Draining the queue that the jobs are enqueued in for a batch later in the test also does not trigger the callbacks.

```ruby
it "doesn't run the success callback of a batch when draining the queue in fake test mode" do
  batch = Sidekiq::Batch.new
  batch.on(:success, BatchCallback)

  batch.jobs do
    CounterWorker.perform_async
    CounterWorker.perform_async
  end

  expect { CounterWorker.drain }.not_to output("Jobs run: 2\n").to_stdout

  expect($global).to eq 2
end
```

Lastly on testing, to have these callbacks fire in tests, you must [add middleware](https://github.com/sidekiq/sidekiq/wiki/Batches#testing) to your tests.

```ruby
around(:example) do |example|
  Sidekiq::Testing.server_middleware do |chain|
    chain.add Sidekiq::Batch::Server
  end

  example.run

  Sidekiq::Testing.server_middleware do |chain|
    chain.remove Sidekiq::Batch::Server
  end
end
```

## Curtain call

This has been an introduction to Sidekiq's batch functionality. It *also* provides a valuable life lesson on acting. Batches can be a great option to parallelize work and report the result of, or combine, those pieces. You can also use it to build complex workflows. Dig into batches in more detail on [Sidekiq's wiki](https://github.com/sidekiq/sidekiq/wiki/Batches).

+++
title = "Announcing simplecov-compare"
date = 2026-02-20T12:23:22-04:00
tags = ["ruby"]
summary = "Comparing coverage across time and space"
description = "SimpleCov::Compare is a gem to find the lines coverage difference between two SimpleCov runs."
+++

SimpleCov's coverage report will tell the coverage of code at a moment of time. You may want to track differences over time though. That's where the [simplecov-compare](https://github.com/kevin-j-m/simplecov-compare) gem comes in.

## Backstory

I have a project at work where I want to evaluate the impact of changes on the application's test coverage. I'm using [SimpleCov](https://github.com/simplecov-ruby/simplecov) to measure and report the coverage.

There are services that help with this. There's a [GitHub action](https://github.com/kzkn/simplecov-resultset-diff-action) I could reach for, if we did use GitHub Actions.

But what I had were the results of two test runs. I had the JSON output from the two runs. And I wanted to compare what was different between those two files. No pipelines or external services.

So I somewhat begrudgingly built one. And now [SimpleCov::Compare](https://github.com/kevin-j-m/simplecov-compare) exists.

## Exploring SimpleCov

My initial hope was to use SimpleCov itself to do this as much as possible. For that, I started by looking at the [collation task](https://github.com/simplecov-ruby/simplecov/blob/522dc7d3aee12084a80680dcb014580ed156e988/lib/simplecov.rb#L86), because I knew that already takes multiple JSON files and stitches them together. I'm not combining them together, more the opposite. But it was my entrypoint to learn how SimpleCov is operating.

The `ResultMerger` [class](https://github.com/simplecov-ruby/simplecov/blob/522dc7d3aee12084a80680dcb014580ed156e988/lib/simplecov/result_merger.rb) is where the majority of the logic lives. The results get [combined](https://github.com/simplecov-ruby/simplecov/blob/522dc7d3aee12084a80680dcb014580ed156e988/lib/simplecov/result_merger.rb#L106). [File by file](https://github.com/simplecov-ruby/simplecov/blob/522dc7d3aee12084a80680dcb014580ed156e988/lib/simplecov/combine/files_combiner.rb), [line by line](https://github.com/simplecov-ruby/simplecov/blob/522dc7d3aee12084a80680dcb014580ed156e988/lib/simplecov/combine/lines_combiner.rb), and [branch by branch](https://github.com/simplecov-ruby/simplecov/blob/522dc7d3aee12084a80680dcb014580ed156e988/lib/simplecov/combine/branches_combiner.rb).

That helped give me an initial understanding on the process, but again, I want to do the opposite. That said, SimpleCov has the ability to represent the data stored in the JSON files as ruby objects. So I pivoted instead to leverage those classes.

The key class (and spoiler alert, ultimate blocker) is the `SourceFile` [class](https://github.com/simplecov-ruby/simplecov/blob/522dc7d3aee12084a80680dcb014580ed156e988/lib/simplecov/source_file.rb). Given a filename and coverage information, you can instantiate one of these objects. It'll then tell you the coverage information I need, like how many lines are covered or missing. Awesome!

Except it's not, for me at least. To know the [covered lines](https://github.com/simplecov-ruby/simplecov/blob/522dc7d3aee12084a80680dcb014580ed156e988/lib/simplecov/source_file.rb#L48), we need to have the lines themselves built up. That seems fair. Building the lines needs access to the source ([src](https://github.com/simplecov-ruby/simplecov/blob/522dc7d3aee12084a80680dcb014580ed156e988/lib/simplecov/source_file.rb#L227)). *That* requires [opening the file](https://github.com/simplecov-ruby/simplecov/blob/522dc7d3aee12084a80680dcb014580ed156e988/lib/simplecov/source_file.rb#L181) by the filename. At which point, I cannot use it for my purposes. Because the filename stored in the JSON resultset is machine-specific.

I ran the coverage on a different machine from the one I want to analyze it on. And even though I have those files on this other computer, they're not in the same directory structure. I'm unable to use that because `SourceFile` cannot open the file with the filename provided.

So I set out on my own.

## Development

While I built from the ground-up, I didn't do so from first principles. I used the structure from SimpleCov to inform the organization of the gem. Where relevant, I comment in the code which SimpleCov class the SimpleCov::Compare class correlates to.

### Representing Data

I was only concerned with lines coverage. I won't explain in this post how ruby reports coverage results. [I've](https://github.com/ruby/ruby/commit/0026f644d739efed0d69911b434a1012ad55c393) done that both [here]({{< ref "rubys-got-you-covered/#lines-coverage" >}}) and [elsewhere](https://docs.ruby-lang.org/en/master/Coverage.html) before.

I started from the bottom and worked my way up. I provided a way to represent an [individual line](https://github.com/kevin-j-m/simplecov-compare/blob/b8b2a2afd3b663dde64f4bb8eecb69cf8cdd544e/lib/simplecov/compare/file_line.rb) in a file. The line can tell you if it's been covered or not. I then combined those lines for an entire [file](https://github.com/kevin-j-m/simplecov-compare/blob/b8b2a2afd3b663dde64f4bb8eecb69cf8cdd544e/lib/simplecov/compare/file_result.rb), which I called a `FileResult`. Here is where I can calculate the lines coverage for a file. Ruby does not provide that information for you.

I built on that again to accept the [overall coverage output](https://github.com/kevin-j-m/simplecov-compare/blob/b8b2a2afd3b663dde64f4bb8eecb69cf8cdd544e/lib/simplecov/compare/result_set.rb) of an individual JSON file, called a `ResultSet`. This keeps a record of the files that the coverage report documents after [parsing the JSON](https://github.com/kevin-j-m/simplecov-compare/blob/b8b2a2afd3b663dde64f4bb8eecb69cf8cdd544e/lib/simplecov/compare/result_file.rb).

### Comparing Results

Now that I had the structure of the data, I could do what I came to do: take two result set files and compare them. I started by comparing [one file to another](https://github.com/kevin-j-m/simplecov-compare/blob/b8b2a2afd3b663dde64f4bb8eecb69cf8cdd544e/lib/simplecov/compare/file_comparison.rb). That comparison needs to know if it's comparing the same file (by filename). It reports on the difference in lines coverage between them.

Comparing [two result sets](https://github.com/kevin-j-m/simplecov-compare/blob/b8b2a2afd3b663dde64f4bb8eecb69cf8cdd544e/lib/simplecov/compare/result_set_comparison.rb) builds on that. That iterates through every file in the [starting result set](https://github.com/kevin-j-m/simplecov-compare/blob/b8b2a2afd3b663dde64f4bb8eecb69cf8cdd544e/lib/simplecov/compare/result_set_comparison.rb#L57) and finds the matching file from the other result set, if there is one. If there is a difference in lines coverage, the class stores that difference.

The comparison sweeps again, this time retrieving the files in the [second result set](https://github.com/kevin-j-m/simplecov-compare/blob/b8b2a2afd3b663dde64f4bb8eecb69cf8cdd544e/lib/simplecov/compare/result_set_comparison.rb#L71) that didn't match. These would be new files, or ones that don't exist in the original result set. Those are also added as differences.

Now I know everything that's different between the two coverage runs.

### Displaying Differences

Having all that information is great. It's less helpful with no way to see it though. For this I have two different concepts.

A formatter accepts a comparison of result sets. It organizing that data in some readable format. It's expected to respond to a `format` method which returns a string. SimpleCov::Compare ships with one formatter, providing a [markdown](https://github.com/kevin-j-m/simplecov-compare/blob/b8b2a2afd3b663dde64f4bb8eecb69cf8cdd544e/lib/simplecov/compare/formatter/markdown_formatter.rb) summation of the results.

A reporter takes in output to provide. Responding to the `report` method, it...reports that output in some format. SimpleCov::Compare has two reporters. One for [STDOUT](https://github.com/kevin-j-m/simplecov-compare/blob/b8b2a2afd3b663dde64f4bb8eecb69cf8cdd544e/lib/simplecov/compare/reporter/simple_reporter.rb) and a more ✨[glamorous](https://github.com/kevin-j-m/simplecov-compare/blob/b8b2a2afd3b663dde64f4bb8eecb69cf8cdd544e/lib/simplecov/compare/reporter/glamour_reporter.rb)✨ option.

{{< figure src="/img/simplecov-compare-terminal.png" class="mid" alt="The output from simplecov-compare as seen in the terminal" >}}

### Using Tooling

To access any of this, there is [one method](https://github.com/kevin-j-m/simplecov-compare/blob/b8b2a2afd3b663dde64f4bb8eecb69cf8cdd544e/lib/simplecov/compare.rb#L22) in the top-level API. It accepts file paths to two coverage reports. You can optionally specify the formatter and reporter.

```ruby
SimpleCov::Compare.report(
  base_path: "starting.json",
  to_path: "ending.json",
)
```

A [script](https://github.com/kevin-j-m/simplecov-compare/blob/b8b2a2afd3b663dde64f4bb8eecb69cf8cdd544e/bin/simplecov-compare) also exists to allow you to call this directly from the terminal.

```sh
simplecov-compare starting.json ending.json
```

## Dependencies

Perhaps ironically, the gem doesn't need SimpleCov directly. The only dependency needed to run the library code is [Glamour](https://github.com/marcoroth/glamour-ruby) from [Marco Roth](https://marcoroth.dev/). That gives me the pretty output for my Markdown in the terminal.

The gem uses [minitest](https://github.com/minitest/minitest) as a test runner with [mocktail](https://github.com/testdouble/mocktail) for mocking. No disrespect to [RSpec](https://rspec.info/). I used building this gem as an opportunity to revisit a different set of tools from that which I use day to day at work.

## The Future

Honestly? I don't know. As-is, this solves my problem.

Given that it does not rely on SimpleCov, I thought about making this tool-agnostic. I can extract the core that compares coverage results. Each coverage tool would require an adapter. That layer would translate the coverage report into a `ResultSet`. SimpleCov would be the first tool supported. And I'd need a different name. If there's interest, [let me know](https://github.com/kevin-j-m/simplecov-compare/issues) along with what types of tools you'd like to use this with.

As it stands, the gem is modular enough to support people providing their own [formatter](https://github.com/kevin-j-m/simplecov-compare/tree/main/lib/simplecov/compare/formatter) to organize the data. Same for writing their own [reporter](https://github.com/kevin-j-m/simplecov-compare/tree/main/lib/simplecov/compare/reporter) to output the results. The [CLI](https://github.com/kevin-j-m/simplecov-compare/blob/main/bin/simplecov-compare) will need some work to support that, but it's possible to [pass in your own](https://github.com/kevin-j-m/simplecov-compare/blob/2402324905668a732302172ab519866121c8a6f6/lib/simplecov/compare.rb#L22) through ruby code now.

I'll field any bug reports. I'm not sure about feature enhancements. If you give it a try, I'd like to hear about it. Enjoying comparing your coverage over time!

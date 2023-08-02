+++
title = "Formatting SQL with anbt-sql-formatter"
date = 2025-09-14T10:23:22-04:00
tags = ["sql", "ruby"]
summary = "A glossary of configuration settings"
description = "This post documents the different configuration settings available in the anbt-sql-formatter gem."
+++

## Seeing SQL

Have you ever needed to display the SQL query used to generate the data in the report you're also showing? Me neither.

UNTIL I DID.

And when I did, I wanted it to look nice. Readable. Formatted.

Something that didn't look like this:

```ruby
irb(main):001:0> puts "SELECT * from FOOS join bars on foos.id = bars.foo_id WHERE A = true AND B = 'test' OR coalesce(status, 'unknown') <> 'unknown'"
SELECT * from FOOS join bars on foos.id = bars.foo_id WHERE A = true AND B = 'test' OR coalesce(status, 'unknown') <> 'unknown'
```

Reading that, I felt broken. And not in the [best possible way](https://thebloggess.com/broken/). I didn't feel any positive vibes. Still don't.

I started thinking about needing to ante up for the [standard documentation](https://blog.ansi.org/sql-standard-iso-iec-9075-2023-ansi-x3-135/) and do some light reading of the nine parts. And then I'd [parse](https://en.wikipedia.org/wiki/Parsing) the statement, using a [lexer](https://en.wikipedia.org/wiki/Lexical_analysis) to generate tokens to determine how to format this string.

Then I realized what time it was. I forgot to mention this, but it was 4pm on Friday. Not exactly the best time to delve into this intellectual pursuit when I had a weekend of not parsing SQL planned.

So instead I did the next best thing - I searched for the result of someone else's intellectual pursuit of the same problem to see if I could apply it. And that's how I found the [anbt-sql-formatter](https://github.com/sonota88/anbt-sql-formatter) gem.

## Initial Formatting

Very generally, anbt-sql-formatter works by setting up the set of rules you want it to use to format the SQL, supplying that to a formatter, and then asking the formatter to apply those rules to the SQL.

You're not on your own to define these rules. With no modifications, it'll apply a default set of rules.

```ruby
irb(main):001:0> require "anbt-sql-formatter/formatter"
irb(main):002:0> rule = AnbtSql::Rule.new
irb(main):003:0> formatter = AnbtSql::Formatter.new(rule)
irb(main):004:0> puts formatter.format("SELECT * from FOOS WHERE A = true AND B = 'test'")
SELECT
        *
    FROM
        FOOS
    WHERE
        A = TRUE
        AND B = 'test'
```

Now, that may not be to your taste, but that's ok - as I mentioned, we can tweak the rules. The rest of this post will describe the different options we can use to customize these rules.

As a starting point, we'll build a class to encapsulate this formatting:

```ruby
require "anbt-sql-formatter/formatter"

class SqlFormatter
  def self.format(sql)
    new.format(sql)
  end

  def initialize
    @formatter = AnbtSql::Formatter.new(rule)
  end

  def format(sql)
    @formatter.format(sql)
  end

  private

  def rule
    AnbtSql::Rule.new.tap do |rule|
      rules(rule)
    end
  end

  def rules(rule)
  end
end
```

The currently empty `rules` method is what we'll modify to demonstrate each option.

## Rule Summary

The rest of this post serves as a reference to describe the different rules you can apply. If you don't want to read this from top to bottom, you may use this summary to guide what sections to dig into.

* [keyword](#keyword): What casing to apply to all keywords.
* [indent_string](#indent_string): The string used to show indentation.
* [function_names](#function_names): The list of known function names.
* [space_after_comma](#space_after_comma): Whether to add a space after a comma.
* [kw_plus1_indent_x_nl](#kw_plus1_indent_x_nl): The list of keywords to add a newline after and indent what follows one level deeper.
* [kw_minus1_indent_nl_x_plus1_indent](#kw_minus1_indent_nl_x_plus1_indent): The list of keywords to add a newline before, one level of indentation shallower, and then indenting what follows one level deeper.
* [kw_nl_x](#kw_nl_x): The list of keywords to add a newline before.
* [kw_nl_x_plus1_indent](#kw_nl_x_plus1_indent): The list of keywords to add a newline before, indented one level deeper.
* [kw_multi_words](#kw_multi_words): The list of multi-word keywords.
* [in_values_num](#in_values_num): The number of entries in an `IN` statement to display on one line.

## keyword

I tend to use a lot of UPPERCASE when I write SQL. I don't know why. Maybe you do too. And maybe you'd wish it displayed lowercase. Or you want to write it in whatever casing you'd like, but have the keywords show up in uppercase consistently. That's what this first rule can dictate for us, with the help of some constants.

### none

With this option, no modifications are made to the casing of keywords.

__Setting the rule__

```ruby
  def rules(rule)
    rule.keyword = AnbtSql::Rule::KEYWORD_NONE
  end
```

__Using the rule__

```ruby
irb(main):001:0> puts SqlFormatter.format("SELECT * from FOOS")
SELECT
        *
    from
        FOOS
```

Here, the `SELECT` keyword is in uppercase characters, as it was in the original string. The `from` keyword is in lowercase, also matching the input string. No changes were made to either keyword.

### uppercase

We can also force all keywords to be in uppercase.

__Setting the rule__

```ruby
  def rules(rule)
    rule.keyword = AnbtSql::Rule::KEYWORD_UPPER_CASE
  end
```

__Using the rule__

```ruby
irb(main):001:0> puts SqlFormatter.format("SELECT * from FOOS")
SELECT
        *
    FROM
        FOOS
```

### lowercase

Or all keywords can be transformed to lowercase.

__Setting the rule__

```ruby
  def rules(rule)
    rule.keyword = AnbtSql::Rule::KEYWORD_LOWER_CASE
  end
```

__Using the rule__

```ruby
irb(main):001:0> puts SqlFormatter.format("SELECT * from FOOS")
select
        *
    from
        FOOS
```

## indent_string

Sure, a certain number of spaces seems like a reasonable [default](https://github.com/sonota88/anbt-sql-formatter/blob/97827ccc8519d19b93e613901bec434eb43c210e/lib/anbt-sql-formatter/rule.rb#L60). But, with this rule, we can use anything we want.

__Setting the rule__

```ruby
  def rules(rule)
    rule.indent_string = "<->"
  end
```

__Using the rule__

```ruby
irb(main):001:0> puts SqlFormatter.format("SELECT * from FOOS")
SELECT
<-><->*
<->FROM
<-><->FOOS
```

Now, this may seem ridiculous. And that's because this example is. However, it became incredibly useful for how I needed to apply this. I needed to show the SQL query on a website using Rails. I used the `simple_format` [method](https://api.rubyonrails.org/classes/ActionView/Helpers/TextHelper.html#method-i-simple_format) to format the text. However, one thing it didn't do is apply the spaces for indentation.

But, by using a [special character](https://en.wikipedia.org/wiki/Sentence_spacing_in_digital_media#Character_encodings) for indentation, the `simple_format` helper method now indents my text.

```ruby
    rule.indent_string = "&emsp;"
```

## function_names

By [default](https://github.com/sonota88/anbt-sql-formatter/blob/97827ccc8519d19b93e613901bec434eb43c210e/lib/anbt-sql-formatter/rule.rb#L87-L115), the gem recognizes a set of functions as...functions. And it formats functions by placing the entirety of the function on one line. However, there may be other functions you want formatted as a function, which will put it on its own line.

Consider the default formatting of our use of `COALESCE`, which is a function the gem does not recognize by default.

```ruby
irb(main):001:0> puts SqlFormatter.format("SELECT COALESCE(status, 'unknown') from FOOS")
SELECT
         COALESCE (
             status
             ,'unknown'
         )
     FROM
         FOOS
```

__Setting the rule__

```ruby
  def rules(rule)
    rule.function_names << "COALESCE"
  end
```

__Using the rule__

```ruby
irb(main):001:0> puts SqlFormatter.format("SELECT COALESCE(status, 'unknown') from FOOS")
SELECT
        COALESCE( status ,'unknown' )
    FROM
        FOOS
```

## space_after_comma

The specifies the highly-important standard of whether a comma in the SQL statement should be followed by a space. By [default](https://github.com/sonota88/anbt-sql-formatter/blob/97827ccc8519d19b93e613901bec434eb43c210e/lib/anbt-sql-formatter/rule.rb#L62), there is no space after the comma.

__Setting the rule__

```ruby
  def rules(rule)
    rule.function_names << "COALESCE" # See function_names section for details
    rule.space_after_comma = true
  end
```

__Using the rule__

```ruby
irb(main):001:0> puts SqlFormatter.format("SELECT coalesce(status, 'unknown') from FOOS")
SELECT
        COALESCE( status , 'unknown' )
    FROM
        FOOS
```

## kw_plus1_indent_x_nl

This is specifying how particular keywords (kw) should be formatted. The "x" refers to the keyword, "nl" is for a newline. The name of the setting is meant to demonstrate how this will format the keyword. This is showing that the keyword will be followed by a newline that's indented one level in from the prior line.

There are a set of keywords that have this formatting applied by [default](https://github.com/sonota88/anbt-sql-formatter/blob/97827ccc8519d19b93e613901bec434eb43c210e/lib/anbt-sql-formatter/rule.rb#L66).

With no change in default formatting, here is how the gem formats the `JOIN` keyword:

```ruby
irb(main):001:0> puts SqlFormatter.format("select * from foos join bars on foos.id = bars.foo_id")
SELECT
        *
    FROM
        foos JOIN bars
            ON foos.id = bars.foo_id
```

__Setting the rule__

```ruby
  def rules(rule)
    rule.kw_plus1_indent_x_nl << "JOIN"
  end
```

__Using the rule__

```ruby
irb(main):001:0> puts SqlFormatter.format("select * from foos join bars on foos.id = bars.foo_id")
SELECT
        *
    FROM
        foos JOIN
            bars
                ON foos.id = bars.foo_id
```

## kw_minus1_indent_nl_x_plus1_indent

This is specifying how particular keywords (kw) should be formatted. The "x" refers to the keyword, "nl" is for a newline. The name of the setting is meant to demonstrate how this will format the keyword. This is showing that the keyword will be put on a newline that's indented one level out from the prior line. The line following the keyword will be indented one level in from the line with the keyword. From what I've seen, the next word following the keyword is immediately placed on the next line with the indenting, though the setting name does not have a "nl" after the keyword "x".

There are a set of keywords that have this formatting applied by [default](https://github.com/sonota88/anbt-sql-formatter/blob/97827ccc8519d19b93e613901bec434eb43c210e/lib/anbt-sql-formatter/rule.rb#L71-L72).

With no change in default formatting, here is how the gem formats the `JOIN` keyword:

```ruby
irb(main):001:0> puts SqlFormatter.format("select * from foos join bars on foos.id = bars.foo_id")
SELECT
        *
    FROM
        foos JOIN bars
            ON foos.id = bars.foo_id
```

__Setting the rule__

```ruby
  def rules(rule)
    rule.kw_minus1_indent_nl_x_plus1_indent << "JOIN"
  end
```

__Using the rule__

```ruby
irb(main):001:0> puts SqlFormatter.format("select * from foos join bars on foos.id = bars.foo_id")
SELECT
        *
    FROM
        foos
    JOIN
        bars
            ON foos.id = bars.foo_id
```

## kw_nl_x

This is specifying how particular keywords (kw) should be formatted. The "x" refers to the keyword, "nl" is for a newline. The name of the setting is meant to demonstrate how this will format the keyword. This is showing that the keyword will be put on a newline at the same level of indentation as the prior line.

There are a set of keywords that have this formatting applied by [default](https://github.com/sonota88/anbt-sql-formatter/blob/97827ccc8519d19b93e613901bec434eb43c210e/lib/anbt-sql-formatter/rule.rb#L80).

With no change in default formatting, here is how the gem formats the `JOIN` keyword:

```ruby
irb(main):001:0> puts SqlFormatter.format("select * from foos join bars on foos.id = bars.foo_id")
SELECT
        *
    FROM
        foos JOIN bars
            ON foos.id = bars.foo_id
```

__Setting the rule__

```ruby
  def rules(rule)
    rule.kw_nl_x << "JOIN"
  end
```

__Using the rule__

```ruby
irb(main):001:0> puts SqlFormatter.format("select * from foos join bars on foos.id = bars.foo_id")
SELECT
        *
    FROM
        foos
        JOIN bars
            ON foos.id = bars.foo_id
```

## kw_nl_x_plus1_indent

This is specifying how particular keywords (kw) should be formatted. The "x" refers to the keyword, "nl" is for a newline. The name of the setting is meant to demonstrate how this will format the keyword. This is showing that the keyword will be put on a newline that's indented one level in from the prior line.

There are a set of keywords that have this formatting applied by [default](https://github.com/sonota88/anbt-sql-formatter/blob/97827ccc8519d19b93e613901bec434eb43c210e/lib/anbt-sql-formatter/rule.rb#L76).

With no change in default formatting, here is how the gem formats the `JOIN` keyword:

```ruby
irb(main):001:0> puts SqlFormatter.format("select * from foos join bars on foos.id = bars.foo_id")
SELECT
        *
    FROM
        foos JOIN bars
            ON foos.id = bars.foo_id
```

__Setting the rule__

```ruby
  def rules(rule)
    rule.kw_nl_x_plus1_indent << "JOIN"
  end
```

__Using the rule__

```ruby
irb(main):001:0> puts SqlFormatter.format("select * from foos join bars on foos.id = bars.foo_id")
SELECT
        *
    FROM
        foos
            JOIN bars
            ON foos.id = bars.foo_id
```

## kw_multi_words

Some keywords that you want to apply formatting to are actually more than one word in SQL. This allows you to specify the multi-word combinations that should be treated as a keyword. There is a [default](https://github.com/sonota88/anbt-sql-formatter/blob/97827ccc8519d19b93e613901bec434eb43c210e/lib/anbt-sql-formatter/rule.rb#L83) set of multi-word keywords the gem uses.

__Setting the rule__

Here we'll use this to specify `INNER JOIN` as a multi-word keyword and also apply how we want that keyword to be [formatted](#kw_nl_x).

```ruby
  def rules(rule)
    rule.kw_multi_words << "INNER JOIN"
    rule.kw_nl_x << "INNER JOIN"
  end
```

__Using the rule__

```ruby
irb(main):001:0> puts SqlFormatter.format("select * from foos inner join bars on foos.id = bars.foo_id")
SELECT
        *
    FROM
        foos
        INNER JOIN bars
            ON foos.id = bars.foo_id
```

## in_values_num

This is used to specify how many items inside of an `IN` statement will appear on a single line.

### Default

With no modifications, each element inside the `IN` statement will appear on its own line.

```ruby
irb(main):002:0> puts SqlFormatter.format("select * from foos where baz in ('a', 'b', 'c', 'd')")
SELECT
        *
    FROM
        foos
    WHERE
        baz IN (
            'a'
            ,'b'
            ,'c'
            ,'d'
        )
```

### One Line

You can force all the elements to appear on a single line using a special constant.

__Setting the rule__

```ruby
  def rules(rule)
    rule.in_values_num = AnbtSql::Rule::ONELINE_IN_VALUES_NUM
  end
```

__Using the rule__

```ruby
irb(main):001:0> puts SqlFormatter.format("select * from foos where baz in ('a', 'b', 'c', 'd')")
SELECT
        *
    FROM
        foos
    WHERE
        baz IN (
            'a' ,'b' ,'c' ,'d'
        )
```

### Specifying the number

You can also define an explicit number of elements to appear on a single line.

__Setting the rule__

```ruby
  def rules(rule)
    rule.in_values_num = 2
  end
```

__Using the rule__

```ruby
irb(main):001:0> puts SqlFormatter.format("select * from foos where baz in ('a', 'b', 'c', 'd')")
SELECT
        *
    FROM
        foos
    WHERE
        baz IN (
            'a' ,'b'
            ,'c' ,'d'
        )
```

## Conclusion

The [anbt-sql-formatter](https://github.com/sonota88/anbt-sql-formatter) gem is a great starting point for formatting SQL statements and includes many customizations to modify the formatting to your liking.

+++
title = "Validate CSV Input With ActiveModel::Validations"
date = 2021-07-19T08:08:53-04:00
tags = ["ruby", "rails", "data"]
summary = "BOOK IT! towards using existing framework functionality"
description = "Data input processes may be subject to different business rules than the data it ultimately correlates to in your system. You can still use familiar tools to apply those constraints and validations that only apply in this one case. Learn how to improve the robustness of CSV file ingestion using ActiveModel::Validations in this article."
canonicalUrl = "https://www.thegnar.com/blog/active-model-validations-csv"
+++

## Standard Library Parsing

We're maintaining a system that tracks information about books, including their
publication dates. On occasion, publishers will send us CSVs with updated
publication dates, and we need to update our Rails application to have those
dates.

We want a repeatable process, so rather than updating these by hand, let's use
Ruby's [CSV class](https://ruby-doc.org/stdlib-3.0.1/libdoc/csv/rdoc/CSV.html)
to parse this data.

```ruby
CSV.parse(csv_text, headers: true).map do |row|
  book = Book.find_by(isbn: row["ISBN"])
  book.update(publication_date: row["Pub Date"])
end
```

Four lines and we have a functionally complete parser that updates our system
how we expect. That all seems great. Until, that is, we actually run it.

## Book Checked Out

We execute our parser on the first data file we receive, and it quickly stops
with the following error:

```
NoMethodError: undefined method `update' for nil:NilClass
```

On inspecting the state of our database, we see that the first three books in
the CSV file had their publication dates updated, but the rest didn't. Looking
at the fourth row in the CSV, we discover that the ISBN for that row isn't in
our database. In that case `find_by` returns `nil`, and calling `update` on
`nil` is exactly our problem. An exception is raised, and further rows of the
CSV aren't parsed.

We can fix that! If we don't find the book, let's log the error and move on to
the next row, without calling `update`.

```ruby
CSV.parse(csv_text, headers: true).map do |row|
  book = Book.find_by(isbn: row["ISBN"])

  if book.blank?
    Rails.logger.error("Could not find book")
    next
  end

  book.update(publication_date: row["Pub Date"])
end
```

Now the entire CSV processes, our books are updated, and everyone's happy.
Right?

## Blank Pages

Days later, we discover that a book which previously had a publication date
doesn't anymore. It's not unusual for a book to not have a publication date - we
have records of books that haven't been published yet. However, books shouldn't
_lose_ an existing publication date.

We see that the book in question was in the CSV, and the Pub Date column was
empty for that row. Turns out, that was an error from the publisher in preparing
the CSV. Any book in that file should always have a publication date - the
purpose of this process is to provide publication dates.

We can prevent this from happening in the future by ensuring that a row has a
publication date before attempting to process it:

```ruby
CSV.parse(csv_text, headers: true).map do |row|
  if row["Pub Date"].blank?
    Rails.logger.error("Skipped updating book with no publication date")
    next
  end

  book = Book.find_by(isbn: row["ISBN"])

  if book.blank?
    Rails.logger.error("Could not find book")
    next
  end

  book.update(publication_date: row["Pub Date"])
end
```

## Losing The Plot

Our "simple" parser is now a lot more complicated. Business rules about the
shape, structure, and expectations of the data are now littered along
with actions consuming the data to do things like find the book and update it
with the appropriate publication date. It's harder to identify what the core
responsibility of this snippet of code is.

## Adding Chapters

Let's try to increase the clarity of our code by separating out how to process
an individual row of the CSV.

### First Act: Establishing A New Character (Class)

We'll start by making a class that takes in the needed data from the CSV row and
finds the book associated with the ISBN.

```ruby
class BookPublicationDateImportRow
  attr_reader :isbn,
    :publication_date

  def initialize(isbn:, publication_date:)
    @isbn = isbn
    @publication_date = publication_date
  end

  def book
    @book ||= Book.find_by(isbn: isbn)
  end
end
```

Perhaps you've heard of a [form object](https://thoughtbot.com/blog/activemodel-form-objects)
to represent data associated with a particular form on your web application. You
can consider that's what we're doing here, except our input isn't from a form on
a web page - it's a row from a CSV file.

### Second Act: Rising Validations

Now, rather than rewriting validation logic, as we had in our procedural code
initially, we can use ActiveModel's [Validations](https://api.rubyonrails.org/v6.1.3.1/classes/ActiveModel/Validations.html)
module. That'll allow us access to the validation [helpers](https://guides.rubyonrails.org/active_record_validations.html#validation-helpers)
we're used to using with other Rails ActiveRecord models.

```ruby
class BookPublicationDateImportRow
  include ActiveModel::Validations

  validates :book, presence: true
  validates :publication_date, presence: true
  ...
end
```

We would have caught our last problem of losing publication dates if that
validation were on the Book model itself - and we may be tempted to look to add
it now. However, remember - having a book with no publication date is totally
normal for our application. It's only in **this** instance of receiving a
publication date update from a publisher with no publication date where it's
unacceptable to have a value for that attribute.

### Third Act: Update Resolution

We'll also mirror ActiveRecord's API by adding in a `save` method that makes
sure our instance is passing its own validations before updating the book:

```ruby
class BookPublicationDateImportRow
  ...
  def save
    return false unless valid?

    book.update(publication_date: @publication_date)
  end
end
```

## Rewriting The Intro

Now that we have something that's responsible for managing an individual row, we
can update our parsing code to be responsible for iterating through that CSV and
pass off the details of how to manage that data to our new class.

```ruby
CSV.parse(csv_text, headers: true).map do |row|
  book_import = BookPublicationDateImportRow.new(
    isbn: row["ISBN"],
    publication_date: row["Pub Date"],
  )

  if !book_import.save
    Rails.logger.error(book_import.errors.full_messages.join(", "))
  end
end
```

## Epilogue

By adding this new class, we've given ourselves an extension point for
additional logic on the row. Any data transformation, like converting the
publication date string to a date object, can be handled here (however, for CSV
parsing, do take a look at the standard library's [converters](https://ruby-doc.org/stdlib-3.0.1/libdoc/csv/rdoc/CSV.html#class-CSV-label-Built-In+Field+Converters)
as well!).

Any further validations that need to be exercised on the data can take place in
this separate class. Moreover, we can use framework features and concepts that
we're already familiar with, rather than rewriting our own business logic for
validation.

> This post originally published on [The Gnar Company blog](https://blog.thegnar.co/active-model-validations-csv).

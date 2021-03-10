+++
title = "Querying PaperTrail Object Changes in JSON"
date = 2021-02-15T16:47:03-05:00
tags = ["ruby", "papertrail", "postgres"]
summary = "Reliving the past has never felt so futuristic"
+++

[PaperTrail](https://github.com/paper-trail-gem/paper_trail) may be a fit for
your rails app if you've ever wanted help answering the question, "what
series of events conspired to put my database in this state?" It acts as a time
capsule, storing each change made to instances of your models that you have
PaperTrail turned on for.

Let's create a user. Unfortunately, we can't make up our mind about which
email to use, so we change that a few times.

```ruby
> user = User.create(email: "kevin@example.com", first_name: "Kevin", last_name: "Murphy")
> user.update(email: "km@example.com")
> user.update(email: "kevinfinal@example.com")
```

We have PaperTrail set to [track changes](https://github.com/paper-trail-gem/paper_trail#only)
made to a user's email. Later on, we may need to do some archaeology to figure
out why this user's email address isn't "km\@example.com". Let's investigate how
PaperTrail can help with that.

## Using PaperTrail's API

PaperTrail provides a few options to [navigate](https://github.com/paper-trail-gem/paper_trail#3b-navigating-versions)
changes made over the history of our object. We're going to use
`where_object_changes` to figure out what happened with "km\@example.com".
`where_object_changes` will find any time the provided attributes changed to
**or** from the values provided.

Let's first find out if this user's email ever was "km\@example.com".

```ruby
> user.versions.where_object_changes(email: "km@example.com").count
=> 2
```

Now we know that at some point in time, this user's email was stored as
"km\@example.com". We can hone in on the changes, and specifically only look at
the email address changes.

```ruby
> user.versions
    .where_object_changes(email: "km@example.com")
    .pluck(:object_changes)
    .map { |c| c.slice("email") }
=> [{"email"=>["kevin@example.com", "km@example.com"]},
   {"email"=>["km@example.com", "kevinfinal@example.com"]}]
```

Any PaperTrail Version has an `object_changes` attribute. The value of that
attribute is a hash. The keys of that hash are the attributes that changed, and
the value is a tuple (as an array) that shows first the value that attribute
changed from, and the value it changed to.

Here we're finding all versions that have to do with the email address
"km\@example.com" and only showing the changes to the email address.

This first version shows that the email address changed from "kevin\@example.com"
to "km\@example.com". Later on, the second version shows the email address
changing from "km\@example.com" to "kevinfinal\@example.com".

## Enhanced JSON Querying

If [PaperTrail versions](https://github.com/paper-trail-gem/paper_trail#postgresql-json-column-type-support)
are stored as JSON or JSONB in Postgres, we have some more detailed queries we
can write by digging into Postgres' [JSON functions](https://www.postgresql.org/docs/9.4/functions-json.html).

### Changed

Sometimes we may not have an attribute value that we want to search for. We may
need to first know when an attribute changed to anything at all. Let's look for
any time that this user's email attribute changed.

```ruby
> user.versions.where("object_changes -> 'email' IS NOT NULL").pluck(:object_changes).map { |c| c.slice("email") }
=> [{"email"=>["", "kevin@example.com"]},
 {"email"=>["kevin@example.com", "km@example.com"]},
 {"email"=>["km@example.com", "kevinfinal@example.com"]}]
```

The first version is logged when you initially create the row in your database,
which is why it changes from the empty string to "kevin\@example.com". From
there, we see the additional updates to email, as all versions have changed the
email address.

### Changing From

While the prior query gives us _less_ specificity than PaperTrail provides with
`where_object_changes`, we may also benefit from _more_ specificity. For
example, we may want to know when the user's email changed from
"km\@example.com", and not care about when it changed to "km\@example.com".

We **can** use `where_object_changes`, but to find out which changes are only
changing the value from "km\@example.com", we'll need to scan the results
ourselves or in memory, rather than relying on the database.

However, we can construct a query ourselves to tell us only when an attribute
changed from a value.

```ruby
> user.versions.where("object_changes ->>'email' ILIKE '[\"km@example.com\",%'").pluck(:object_changes).map { |c| c.slice("email") }
=> [{"email"=>["km@example.com", "kevinfinal@example.com"]}]
```

Remember that the object changes of an attribute are stored as a tuple in an
array. The first value in the array is the value the attribute changed from, and
the second value is the value the attribute changed to.

This query looks within the `object_changes` attribute, and checks to see if the
value of the `email` field within that attribute has the email address we're
looking for as the first element in the tuple. The opening bracket before the
email address in escaped quotes is the start of the array/tuple.

### Changing To

We can also construct a query to look for versions on the other side of the
equation. We can find all instances where the attribute changed _to_ a
particular value.

```ruby
> user.versions.where("object_changes ->>'email' ILIKE '[%,\"km@example.com\"]'").pluck(:object_changes).map { |c| c.slice("email") }
=> [{"email"=>["kevin@example.com", "km@example.com"]}]
```

When looking for what the value changed to, we want to look in our tuple to find
the email address in question as the last element, which will be preceded by a
comma and have an ending bracket after it.

## Trailing Off

Thanks to storing our PaperTrail versions in a JSON format, we were able to
sneak in an applied lesson on Postgres' powerful JSON functions and
operators while also reviewing the structure of data that's used by PaperTrail
to track any versions of our database records.

> This post originally published on [The Gnar Company blog](https://blog.thegnar.co/querying-papertrail-object-changes-json).

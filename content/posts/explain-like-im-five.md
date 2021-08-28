+++
title = "Explain Like I'm Five"
date = 2020-11-11T15:38:10-05:00
tags = ["rails", "activerecord"]
summary = "Five ways to run EXPLAIN on your Rails database queries"
description = "Five ways to run EXPLAIN on your Rails database queries."
+++

## `EXPLAIN`ing Myself

Running `EXPLAIN` to show the execution plan for different database queries can
help you understand why the performance of a particular database interaction
is the way it is, and how you might be able to improve it. The purpose of this
post is not to interpret and understand the results of an execution plan.
Rather, we'll learn five different ways you can get this information from your
Rails app.

Some of these examples are PostgreSQL-specific.

## 1. ActiveRecord's `explain` Method

Rails already has the `explain` method [built into](https://apidock.com/rails/ActiveRecord/Relation/explain) ActiveRecord for you to use.
You can add `explain` to any ActiveRecord relation, and you'll receive the
execution plan.

```ruby
> User.where(email: "test@example.com").explain
=> EXPLAIN for: SELECT "users".* FROM "users" WHERE "users"."email" = $1 [["email", "test@example.com"]]
                      QUERY PLAN
-------------------------------------------------------
 Seq Scan on users  (cost=0.00..2.71 rows=1 width=340)
   Filter: ((email)::text = 'test@example.com'::text)
(2 rows)
```

This gives us a great starting point, and works across various databases.
However, if you want some additional features, like [running](https://www.postgresql.org/docs/current/sql-explain.html) `EXPLAIN ANALYZE`,
you'll need to look elsewhere.

## 2. Interpolating a Query in an ActiveRecord Connection

You can fall back to creating your own SQL statement and passing that into
ActiveRecord's `execute` method. However, you probably don't want to go through
the error-prone and arduous effort of hand-writing the SQL query you just wrote
using ActiveRecord's syntax.

Luckily, you don't have to! You can convert your ActiveRecord query to a string with
`.to_sql`, and add that into a string you provide to `execute`:

```ruby
> ActiveRecord::Base.connection.execute("EXPLAIN #{User.where(email: "test@example.com").to_sql}").values
=> [["Seq Scan on users  (cost=0.00..2.71 rows=1 width=340)"], ["  Filter: ((email)::text = 'test@example.com'::text)"]]
```

This itself isn't much of a win at all over using ActiveRecord's `explain`
method. It's longer, you've got to remember to grab the `values` from the
`execute` results, and the output isn't as nicely formatted. However, because
this is "just SQL" that you're running in `execute`, you can use any features
your database engine of choice provides, like `EXPLAIN ANALYZE`:

```ruby
> ActiveRecord::Base.connection.execute("EXPLAIN ANALYZE #{User.where(email: "test@example.com").to_sql}").values
=> [["Seq Scan on users  (cost=0.00..2.71 rows=1 width=340) (actual time=0.184..0.233 rows=0 loops=1)"],
 ["  Filter: ((email)::text = 'test@example.com'::text)"],
 ["  Rows Removed by Filter: 57"],
 ["Planning time: 0.185 ms"],
 ["Execution time: 0.472 ms"]]
```

Thanks to [Mark Lodato](https://github.com/mlodato517) for this recommendation.

## 3. The activerecord-explain-analyze Gem

If you're willing to take on a dependency to get some additional explanatory power,
are using ActiveRecord 4 through 6, and use PostgresSQL, then
you can reach for the [activerecord-explain-analyze](https://github.com/6/activerecord-explain-analyze) gem.

Now you can specify the output formatting of your `EXPLAIN` results, and call
`EXPLAIN ANALYZE`:

```ruby
> User.where(email: "test@example.com").explain(analyze: true)
=> EXPLAIN for: SELECT "users".* FROM "users" WHERE "users"."email" = $1
Seq Scan on public.users  (cost=0.00..2.71 rows=1 width=340) (actual time=0.120..0.128 rows=0 loops=1)
  Output: id, email, sign_in_count, current_sign_in_at, last_sign_in_at, current_sign_in_ip, last_sign_in_ip, created_at, updated_at, time_zone, first_name, last_name, role, applicant_id, centrify_uuid, display_name, uuid, login_authorized, invite_id, legacy_identifier, disabled_at, invite_sent_at, password_last_changed_at, deprovisioning_reason
  Filter: ((users.email)::text = 'test@example.com'::text)
  Rows Removed by Filter: 57
  Buffers: shared hit=2
Planning time: 0.277 ms
Execution time: 0.183 ms
```

## 4. The pg-eyeballs Gem

[pg-eyeballs](https://github.com/bradurani/pg-eyeballs) is another gem that's PostgreSQL-specific, and provides additional
functionality that ActiveRecord's `explain` method does not currently.

Our sought-after `EXPLAIN ANALYZE` is one of many options you can request:

```ruby
> User.where(email: "test@example.com").eyeballs.explain(options: [:analyze])
=> ["Seq Scan on users  (cost=0.00..2.71 rows=1 width=340) (actual time=0.028..0.036 rows=0 loops=1)\n  Filter: ((email)::text = 'test@example.com'::text)\n  Rows Removed by Filter: 57\nPlanning time: 0.087 ms\nExecution time: 0.084 ms"]
```

## 5. Terminal CLI of Your Database

All of these prior examples have been run from within a Rails process such as the
Rails console. However, we can skip Rails entirely and use our database
directly. In our case with PostgreSQL, we can use `psql`.

```bash
# psql -U postgres
```

After connecting, we can list which databases exist with `\l`.

```bash
postgres=# \l
                                              List of databases
                Name                 |  Owner   | Encoding |  Collate   |   Ctype    |   Access privileges
-------------------------------------+----------+----------+------------+------------+-----------------------
 example_app_development             | postgres | UTF8     | en_US.utf8 | en_US.utf8 |

```

After finding the correct database, we can connect to it with `\c`.

```bash
postgres=# \c example_app_development
```

And then we can run any query we would like, including EXPLAIN:

```bash
example_app_development=# EXPLAIN SELECT * FROM USERS WHERE EMAIL = 'test@example.com';
                      QUERY PLAN
-------------------------------------------------------
 Seq Scan on users  (cost=0.00..2.71 rows=1 width=340)
   Filter: ((email)::text = 'test@example.com'::text)
(2 rows)
```

Again, we have all the features available to us that our database engine
supports, so we can use `EXPLAIN ANALYZE` or any other functionality, without
needing it to be built into Rails. This gives us all the power our database
provides, but we lose the expressiveness of ActiveRecord's query API - or
rather, we need to find the `.to_sql` representation of the query we're
interested in prior to using this.

## `EXPLAIN`ing Which to Use

If you're interested in quickly getting an execution plan of an existing
ActiveRecord query, start with using ActiveRecord's `explain` method.

Should you need more functionality that your database engine provides, you can
`execute` any query you would like to your database through ActiveRecord.

If you need that additional functionality, such as `EXPLAIN ANALYZE`, on a
regular basis, consider taking on an additional dependency that'll provide that
for you, such as `activerecord-explain-analyze` or `pg-eyeballs`.

Don't forget you can go directly to your database without using Rails as an
intermediary.

I hope this has `EXPLAIN`ed a thing or two (or five) about ways to gather performance
information for your queries. Once you decide which method is right for you,
good luck optimizing!

> This post originally published on [The Gnar Company blog](https://blog.thegnar.co/explain-like-im-five).

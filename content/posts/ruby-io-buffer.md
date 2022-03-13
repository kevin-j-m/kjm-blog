+++
title = "Buffered IO Streams In Ruby"
date = 2021-09-20T09:00:07-04:00
tags = ["ruby"]
summary = "The buffer ate my homework"
description = "Ruby IO may not immediately write where you are asking it to. Here we explore where writing to a stream goes and when it may or may not be committed where you would expect it to."
canonicalUrl = "https://www.thegnar.com/blog/ruby-io-buffer"
+++

## Object Permanence

We have some really important information in our console.

```ruby
> data = "some really important information"
=> "some really important information"
```

We want to store that information to disk, but only temporarily. We'll do so
using a `Tempfile`, which is built in to Ruby, but must be required to be used.

```ruby
> require "tempfile"
> file = Tempfile.new
```

This creates a new temporary file on disk, and as it's new, it is currently
empty, which we'll check two different ways.

```ruby
> File.size(file.path)
=> 0
> file.size
=> 0
```

**Finally** we can write some really important information to disk, after which
we'll check the size again.

```ruby
> file.write(data)
=> 33
> File.size(file.path)
=> 0
> file.size
=> 33
```

When we [write](https://ruby-doc.org/core-3.0.1/IO.html#method-i-write) to the
IO stream (in this case a file), we get the number of bytes written, 33,
returned. After writing to the file, we asked for the size of the file with
`File.size` and got 0. Then, we asked the file for its size and got 33. What
happened?

## Where's that string?

Maybe the string was written to the file instance in memory before being
committed to disk. Let's look at the [size](https://ruby-doc.org/stdlib-3.0.1/libdoc/objspace/rdoc/ObjectSpace.html#method-c-memsize_of) of our objects.

```ruby
> require "objspace"
> ObjectSpace.memsize_of(data)
=> 40
> ObjectSpace.memsize_of(file)
=> 80
```

Now, `memsize_of` is a hint/guess - and the docs are clear about that:

> Note that the return size is incomplete. You need to deal with this information as only a HINT.

That'll work for us; we'll just use it to see if it changed at all.

Let's try to write again, now that we've seen the file is currently 80 bytes in
memory.

```ruby
> file.write(data)
=> 33
> ObjectSpace.memsize_of(file)
=> 80
```

The size of the `file` object itself didn't change, so I guess it's not hiding
in there.

As we saw previously, passing the path to `File.size` doesn't show the
newly-written bytes being written to the file, but asking the `file` instance
itself for its `size` does.

Also, after asking for `file.size`, `File.size(file.path)` **does** have the
size including the newly-written bytes. So they do _eventually_ agree on the
file's size.

```ruby
> File.size(file.path)
=> 33
> file.size
=> 66
> File.size(file.path)
=> 66
```

## Sizing Up The Difference

Calling `size` on the file instance has a [documented](https://ruby-doc.org/stdlib-3.0.1/libdoc/tempfile/rdoc/Tempfile.html#method-i-size) side effect.

> As a side effect, the IO buffer is flushed before determining the size.

That explains where our string went after writing it! It was stored in Ruby's
IO buffer. Flushing the buffer [pushes](https://ruby-doc.org/core-3.0.1/IO.html#method-i-flush)
its contents to the operating system.

Let's observe that by checking the size of the file, writing more bytes to it,
checking the size of the file doesn't change, and explicitly flushing the buffer.

After flushing the buffer, the size of the file _does_ change by the number of
bytes written.

```ruby
> File.size(file.path)
=> 66
> file.write(data)
=> 33
> File.size(file.path)
=> 66
> file.flush
> File.size(file.path)
=> 99
```

[Rewinding](https://ruby-doc.org/core-3.0.1/IO.html#method-i-rewind) the file
after writing it also appears to flush the buffer.

```ruby
> File.size(file.path)
=> 99
> file.write(data)
=> 33
> File.size(file.path)
=> 99
> file.rewind
=> 0
> File.size(file.path)
=> 132
```

## No Buffering

We can bypass Ruby's IO buffer by setting the stream's [sync mode](https://ruby-doc.org/core-3.0.1/IO.html#method-i-sync).
By default, this is set to buffer; however, setting it to true will immediately
flush the stream contents to the operating system.

```ruby
> new_file = Tempfile.new
> new_file.sync
=> false
> new_file.sync = true
> streaming = "no buffering"
> File.size(new_file.path)
=> 0
> new_file.write(streaming)
=> 12
> File.size(new_file.path)
=> 12
```

`File.size` is recognizing the bytes in the file without needing to flush the
buffer, either directly or via a method that does so as a side effect. The sync
mode is pushing whatever we write directly to disk (at least, through the
operating system).

## Closing Our Stream

Ruby will buffer writes in an IO stream, such as a file, and you need to be
mindful of when or if that buffer is flushed should you then immediately check
the impact that writing to a stream had on the item being written to.

> This post originally published on [The Gnar Company blog](https://blog.thegnar.co/ruby-io-buffer).

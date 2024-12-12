---
title: "Don't Forget to Cast Your Boolean ENV Variables"
tags: [ruby]
summary: >
  When ENV variable values are being used in conditionals, they can do the exact opposite of what you
  intended them to do if they aren't cast to booleans first
date: '2024-12-12'
draft: false
---

## TL;DR

```ruby
# bad
if ENV.fetch('MY_VAR', false)

# good
if ActiveModel::Type::Boolean.new.cast(ENV.fetch('MY_VAR', false))
```

Environment variables are always strings. When using ENV variables to represent booleans, they should always be parsed as such to prevent bugs.
Otherwise, you might end up accidentally writing `if "false"`, which is truthy.

---

The other day, I was working on a client's project to repair records that were missing some of their file attachments. While looking through
the code, I came across this conditional:

```ruby
if ENV.fetch('IMPORT_SKIP_EXISTING', false)
  # Skip processing this record's files
else
  # Process this record completely, including files
end
```

*In case you're unfamiliar with Ruby, `ENV` is a hash table of key-value pairs. The `#fetch` method will attempt to fetch the value at the
specified key, i.e. the first argument (`"IMPORT_SKIP_EXISTING"`). If a value can't be found, it falls back on the second argument (`false`)*

One way to repair the broken records is to reimport them into the database, effectively syncing them with the data's source of truth. I chose this as
my path forward and checked the value of the variable in the `.env` file:

```.env
IMPORT_SKIP_EXISTING=true
```

The next step was simple: change the ENV variable's value to `false` to allow for each record's files to be brought in when reimporting them.

Except... when I made the change and started reimporting, none of the records were getting any files.

> What did I miss?

I was digging through the code for several minutes before I remembered:

> Oh yeah, ENV variable values are strings

If we strip away the ENV fetching syntax, this is what I *thought* I was doing:

```ruby
if false # falsy
```

But in reality, what I was doing was:

```ruby
if "false" # truthy
```

The insidious part about this is that, before I changed the value, it was "working" just fine; `if true` and `if "true"` are both truthy.

The solution I chose was to cast the value to a boolean:

```ruby
if ActiveModel::Type::Boolean.new.cast(ENV.fetch('IMPORT_SKIP_EXISTING', false))
```

Alternatively, I could have used string comparison (`if "true" == "true"`). However, I prefer casting to a boolean in this case since
the intended purpose is an on/off switch, a.k.a. a boolean's sole purpose.

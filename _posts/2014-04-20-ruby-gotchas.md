---
layout: post
title:  "ruby gotchas"
date:   2014-04-20 12:00:00
tags:
- coding
- ruby
- gotchas
---

I just want to start by saying I've been doing ruby development
professionally for about 2 years now and like any ruby programmer,
I discovered quite a bit of things that didn't quite work as expected.

A few weeks ago, I found an interesting post that had a collection of these 'gotchas'
from El Passion - [Ruby Gotchas](http://blog.elpassion.com/ruby-gotchas/).
I'd like to expand on this post but I highly recommend taking a look at the original article.

## and vs && with render/redirect_to

To avoid muliple `render`s or `redirect_to`s, you'll want to
`return` to stop the rest of the code from running.


```ruby
class DashboardController < ApplicationController
  def index
    render 'template' && return #=> index.html.erb
  end
end
```

> Make sure to use `and return` instead of `&& return` because `&& return`
  will not work due to the operator precedence in the Ruby Language.

```ruby
class DashboardController < ApplicationController
  def index
    render 'template' and return #=> template.html.erb
  end
end
```

### single quote vs double quote 

The main difference between single quotes and double quotes is that
single quotes are for literals and auto-escapes escape characters in strings.
Because of this, you can't use string interpolation with single quotes.

```ruby
> date = Time.now
=> 2014-04-20 20:27:09 -0400

> 'The current time is #{date}'
=> "The current time is \#{date}"

> "The current time is #{date}"
=> "The current time is 2014-04-20 20:27:09 -0400"
```

### Best practices
This just depends on your personal style, I prefer double-quotes unless your
string literal contains `"` or escape characters you want to suppress.

### Performance
There used to be a performance difference between `'` vs `"` but if you
are using `>= 1.9.3` it [doesn't matter](http://stackoverflow.com/questions/1836467/is-there-a-performance-gain-in-using-single-quotes-vs-double-quotes-in-ruby/1836838#1836838).

## `:+` vs `&:+`
I wouldn't really call this a gotcha but just something you might want to know.
In `inject`/`reduce` you don't need to turn your symbol into a `proc`.
I think this looks a little better but it also gives you a little performance boost.

```ruby
> [1, 2, 3, 4].reduce(:+)
=> 10

> [1, 2, 3, 4].reduce(&:+)
=> 10
```

### Performance
```ruby
     real
&:+ (  0.000075)
:+  (  0.000046)
```

### Note
This seems to only work for `inject`/`reduce`

```ruby
> ('a'..'z').map(:upcase)
ArgumentError: wrong number of arguments(1 for 0)
```

---
Hopefully there will be more 'gotchas' to come shortly.
---
layout: post
title:  "service objects (http)"
date:   2015-03-30 12:00:00
tags:
- coding
- ruby
- services
---

There's been a big move to service oriented architecture (SOA) in the past few years
and if you're like me, you probably work with quite a few services on a regular basis.
Hell, you might even have a few projects that don't even have a database and rely solely on external services/APIs.

I'm not really going to go into any of the pros and cons in this kind of design, but I do want
to show you a pretty simple approach I came up with to consume these services.
For now, I'm going to focus on consuming HTTP services but that doesn't mean this approach couldn't work for other protocols.

## Why?
I had a pretty simple job to do, all I had to do was build some XML and post it to an external
"RESTful API". While looking through the project, however, I noticed four different http libraries were
being used to consume different services, even though they were all basically doing the same thing.
This threw up a red flag to me, so I decided to do something about it.

## What?
For the basics, we need to be able to:

- perform different HTTP methods and recieve a request object (Request)
- make sense of the data (abstraction) and hopefully keep business logic contained (Service)

Since the url is really the only thing that changes for most http request, lets create a base http class.

```ruby
class HttpRequestBase
  HTTP_VERBS = [:get, :post, :delete, :put].freeze

  attr_reader :response, :timeout

  def initialize(args={})
    @timeout = args[:timeout] || default_timeout
  end

  # override in child
  def base_url
  end

  def default_timeout
    10
  end

  def request(method, path, params)
    unless HTTP_VERBS.include?(method)
      warn("HTTP method not supported") and return nil
    end

    url = "#{base_url}#{path}"
    @response = Curl.send(method, url, params)
  ensure
    log(method, url, params)
  end

  class << self
    HTTP_VERBS.each do |action|
      define_method(action) do |path, params={}|
        new.tap do |obj|
          obj.request(action, path, params)
        end
      end
    end
  end

  private
  # Curb doesn't return a response on certain edge cases
  def log(method, url, params)
    status = response ? response.status : "FAILED"
    logger.info("#{status} method=#{method.upcase} #{url} #{params}")
  end
end
```

This might be fine for you but lets think about how other devs might use this class.
Instead of just having comments like `# override in child` that can easily be overlooked
lets raise a `NotImplementedError`.

It's probably also a good idea to give children classes a way to modify the initializer and the http request
for more control. We'll accomplished this by providing a `post_intialize` and `modify_request` hook 
so no one forgets a `super`.

```ruby
def initialize(args={})
  @timeout = args[:timeout] || default_timeout
  post_initialize(args)
end

# post initialize hook
def post_initialize(args)
  nil
end

# override to modify http object (add headers, etc)
def modify_request(http)
  http
end

# ...

def request(method, path, params)
  #...
  @response = Curl.send(method, url, params) do |http|
    http.timeout = timeout
    modify_request(http)
  end
end
```

Cool, so now we really just have a nicer wrapper around Curb, but that lets us do something like this:

```ruby
class GithubRequest < HttpRequestBase
  def base_url
    'https://api.github.com'
  end

  def modify_request(http)
    http.headers['User-Agent'] = 'Valid Agent'
  end

  # this could definitely be put into an included module/concern
  def json_body(default="{}")
    raw = response ? response.body : default
    JSON.parse(raw, symbolize_names: true)
  end
end

http = GithubRequest.get('/users/ebtoulson')
# => #<GithubRequest:0x007fa4420c27e8
#  @response=#<Curl::Easy https://api.github.com/users/ebtoulson>,
#  @timeout=10>

http.json_body
# => {:login=>"Ebtoulson",
#     :id=>1562555,
#     ... }

```

[Source code](https://gist.github.com/Ebtoulson/be69142c53ac3de32d1b)

## What's next?
I think the next thing I'm going to look into is something a more efficient.
Right now, every request object performs their http request synchronously,
so if you needed data from three different services for a page to load, it would be the sum of the response times.
Setting up an async que, similar to [typhoeus](https://github.com/typhoeus/typhoeus),
could allow our wait time to be the max of the response times.

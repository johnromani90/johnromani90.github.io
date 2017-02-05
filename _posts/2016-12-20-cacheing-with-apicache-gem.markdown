---
layout: post
title: Cacheing Ruby With APICache gem
date: '2016-12-20T05:51:00.001-08:00'
author: John Romani
tags:
- api cache gem
- ruby
- cache
- api
- ruby on rails
modified_time: '2016-12-20T11:57:29.460-08:00'
blogger_id: tag:blogger.com,1999:blog-5959763481811732424.post-2587736556669433082
blogger_orig_url: http://johnromani90.blogspot.com/2016/12/cacheing-with-apicache-gem.html
---

If you're running a rails app then chances are you're playing with multiple apis. This week I found myself working with the robust JIRA api so that I could return worklogs and do something meaningful (and more user friendly) for a client.

You could easily write your own cacheing class using timestamps and urls as keys for the queries, but I really like this gem [APICache by Martyn Loughran](https://github.com/mloughran/api_cache) because it has a couple nice features out of the box like :period, :valid, and :cache timeout options. The gem is also simple; __it consists of two main classes, Api and Cache, which are both under 100 lines of code__. The other few classes are wrappers that help you configure your 'mem-cache' store.

First download the api_cache gem Then add 'config/initializers/api_cache.rb' And add this code 

```ruby
#congig/initializers/api_cache.rb

if Rails.env.production?
  APICache.store = APICache::DalliStore.new
end
```

This will set up a dalli store (which is a pure Ruby client for accessing memcached servers) when you are in production and will use memory to cache data when running locally. This allows you to boot up your app and play around without having to run a second server every time. When you get ready to deploy to production read the Dalli docs to learn more about how the process works.

In my example we are retrieving large data sets. Returning the JSON and refreshing the page took around 15 seconds, no bueno. In order to cache all our request to the Jira server we used our own wrapper class that defined a 'get' method. This class inherited from our jira::client so that way all gets would go through our logic. If we want to implement a new http request (like put, patch, post) we can follow the same pattern. We then wrapped our get method with APIcache logic 

```ruby
def get( url )
  APICache.get(url, :cache => cache_timeout, :timeout => 15) do
    super(url)
  end
end

def cache_timeout
  skip_cache ? 1 : 3000
end
```

The skip_cache is an attr_accessor which we pass in when we instantiate this class on specific situations. Besides that we use our default cacheing timeout setting which is 50 mins. 

```ruby
The Api Cache gem also allows these configurations 
{
  :cache => 600,    # 10 minutes  After this time fetch new data
  :valid => 86400,  # 1 day       Maximum time to use old data
                    #             :forever is a valid option
  :period => 60,    # 1 minute    Maximum frequency to call API
  :timeout => 5     # 5 seconds   API response timeout
  :fail =>          # Value returned instead of exception on failure
}
```

Check out the [docs](https://github.com/mloughran/api_cache) for more info.
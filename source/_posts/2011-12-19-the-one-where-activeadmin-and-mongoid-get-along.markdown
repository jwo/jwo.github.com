---
layout: post
title: "The One Where ActiveAdmin and Mongoid Get Along"
date: 2011-12-19 09:18
comments: true
categories: mongoid, activeadmin
---

So you like ActiveAdmin, but are using Mongoid. And so far, ActiveAdmin hates on Mongoid because it uses ActiveRecord instead of ActiveModel. What's a Mongoid to do? Develop your own admin system? As If!
<!-- more -->

First, Watch this Issue
-----------------------

There's already a [github issue](https://github.com/gregbell/active_admin/issues/26) about adding support for other ORMs. Maybe voice your support there.

Then, Monkey Patch!
-------------------

Add this code to your config/initializers/active_active_for_mongoid.rb

{% gist 1497688 %}

It's easy to tell what's going on with the code -- ActiveAdmin needs a way to tell what the name of the collection is (otherwise you get the quoted_table_name problem). Mongoid also has some custom ways of sorting. For example, 

~~~
#ActiveRecord
Product.order("published_at DESC").all
#Mongoid
Product.all(sort: [[ :published_at, :desc ]])
~~~
{:lang="ruby"}

The mongo queries that get executed look like:

~~~
MONGODB yourapp_development['users'].find({}).limit(30).sort([["email", :desc]])
MONGODB yourapp_development['users'].find({}).limit(30).sort([["email", :asc]])
~~~
{:lang="ruby"}

Props to [https://github.com/ebeigarts](https://github.com/ebeigarts) for the patch. It gets you to at least run with the ball for now.

Downsides
---------

* Filters are disabled
* Comments are disabled (I have never used this with ActiveAdmin though)
* You'll probably need to disable this once ActiveAdmin closes issue [#26](https://github.com/gregbell/active_admin/issues/26) (which you are now watching, right?)

######Version Disclosure: This post was valid with ActiveAdmin 0.3.4, Mongoid 2.3.4 and Rails 3.0/3.1

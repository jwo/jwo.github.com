---
layout: post
title: "Easy, Light Weight, and Magical Parallel Processing in Ruby"
date: 2013-02-07 11:18
comments: true
categories: ruby
---

Ever had a set of tasks in Ruby take too long?
------------------------------

We've all been there -- you have code that needs to be run, and it's taking forever. You wish there was a way to speed things along, but you can't tweak the algorithm. You read about multi-threading but hear tales of dragons, pirates, and warnings of people who have ventured before you never to return.

Worry Not, Celluloid is here! (And celluloid-pmap is an easy way to get started)
------------------------------
But fear not! [Celluloid](http://celluloid.io) exists, and is awesome. It's an actor based implementation, but all you _need_ to know is that it's awesome. You can process an array of things in parallel, and continue when it's complete.

<!-- more -->

Let's say you started with the task to see which servers are alive and which are not-so-much-alive:

```ruby
servers = Server.all.map do |server|
  server.status = check_status(server) 
end
# do something with the non-responsive servers
```

So this would map over all servers, and make a network call to check if it's alive.

In a normal non-parallel world, if each status call would take 0.1 seconds, then 10 servers would take 1 second, and 10000 servers would take 1000 seconds. Sub-optimal.

If you execute them in parallel, then goodness happens. Under the hood, celluloid-pmap uses Celluloid::Future's for each element in the array. The pmap will wait until the value is complete before returning, and we'll wait for them all to finish before continuing.

That same example in parallel would look:

```ruby
Server.all.pmap do |user|
   #… same code here
end
```

Parallelization cuts processing time to your slowest item
-------------------------

Let's say you need to create a PDF report for a set of users, store them at S3, and download them for you to give to your users when it's complete. This is a great case for parallel processing since you'll be waiting on:

* wkhtmlpdf to convert HTML to PDF
* fog to upload PDF to S3
* curl to download PDF from S3 for you

With the example below, your total processing time gets cut to the slowest report generated/uploaded/downloaded.

```ruby
    ["email1@example.com", "email2@example.com"].pmap do |email|
      user = User.find_by_email!(email)
      CreatesReports.new(user).generate_reports
      user.reports.each{|report| `curl -o #{report.filename} \"#{report.pdf.url}\"`}
      puts "reports ready for #{email}"
    end
    puts "Everybody's done!"
```

Other examples of usages:

* Searching Google and Amazon and What
* Scheduling of Sidekiq jobs in parallel to speed things up
* Deleting of files at once
* Making database calls in parallel

Make sure to obey your database connection maximums
-------------

If you are iterating over a set of documents and calling any resource that has a limited connection set, or a rate limit, you might run into a Connection Pool problem. That is, you might try connect with 20 connections and your default pool size is 5. What can be done?

celluloid-pmap uses a Celluoid Supervisor to set a maximum number of actors working at the same time. So if you can only connect to 5 postgres users at the same time, you can set that like so:

```ruby
users.pmap(5) {|user| user.say_anything! }
```

The (5) argument will say it's OK to use as many as 5 actors at once. By default, celluloid-pmap will default to the number of cores your machine has.

Also check out Celluloid
-----------------------

I started adding code into a Rails initializer from [Celluloid Simple Pmap](https://github.com/celluloid/celluloid/blob/master/examples/simple_pmap.rb) example. It went like this:

config/initializers/celluloid-pmap.rb

```ruby
module Enumerable
  def pmap(&block)
    futures = map { |elem| Celluloid::Future.new(elem, &block) }
    futures.map { |future| future.value }
  end
end
```

This worked very well, but I found I was adding it to every.single.project. So I worked up an example to have a Supervisor to help with connection pooling and rate limiting, and bam… a gem was born.



Installation and configuration is over at [https://github.com/jwo/celluloid-pmap](https://github.com/jwo/celluloid-pmap). But it's as simple as you'd think:

```
gem install celluloid-pmap
```

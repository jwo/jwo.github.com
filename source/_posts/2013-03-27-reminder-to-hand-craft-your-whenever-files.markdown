---
layout: post
title: "Reminder to hand-craft your whenever files"
date: 2013-03-27 14:39
comments: true
categories: 
---

Day 5: crash. Day 6: crash. Each day, we were receiving notifications from
Rackspace that the server had exceeded its memory allocation and was thrashing
too hard; it rebooted our server for us.

My initial reaction: Increase the memories!

I looked deeper into the problem, and along with some colleagues, discovered
that at midnight our server was kicking off 9 different processes at the same 
time. 9x Rails is just about 8x too many.

<!-- more -->

My whenever schedule (names changed to reflect tex-mex dishes)

```ruby
set :output, 'log/cron.log'

job_type :rake, "cd :path && RAILS_ENV=:environment bundle exec rake :task :output"

every 1.day, :at => '12am' do
  rake 'margarita:enchiladas'
  rake 'margarita:fajitas'
end

every 1.minute do
  rake 'guacamole:soft_tacos'
end

every 12.hours do
  rake 'guacamole:tortilla_soup'
  rake 'guacamole:grande_combo_de_tejas'
end
```

Seems pretty standard for how I organize and schedule tasks. But each of these
would be (and were) running at midnight, since they all intersected that particular
intersection of space and time.

Instead, if we spread this around a bit, we could still get the business
requirements accomplished:

1. Order soft tacos pretty often
2. Get the other dishes one or twice a day

The timing, other than that, didn't matter. And in your apps, it probably
doesn't matter that often either.

An updated "Hand Crafted, Artisan Tex Mex Whenever File"

```
set :output, 'log/cron.log'

job_type :rake, "cd :path && RAILS_ENV=:environment bundle exec rake :task :output"

every 1.day, :at => '12am' do
  rake 'margarita:enchiladas'
  rake 'margarita:fajitas'
end

# skip the top of the hour. Every 5 minutes
every '5,10,15,20,25,30,35,40,45,50,55 * * * *' do
  rake 'guacamole:soft_tacos'
end

every 1.day, :at => ['3am', '3pm'] do
  rake 'guacamole:tortilla_soup guacamole:grande_combo_de_tejas'
end
```

What did this gain us?

* At midnight, only 2 tasks will run.
* Instead of `tortilla_soup` and `grande_combo_de_tejas` running at the same time, they'll now run sequentially (saves on the RAM) 
* `soft_tacos` skips the midnight run

Reminder: Hand craft your whenever cron jobs. And to test the output: `whenever`

```
0 0 * * * /bin/bash -l -c 'cd /Users/jwo/Projects/texmex && RAILS_ENV=production bundle exec rake margarita:enchiladas >> log/cron.log 2>&1'

0 0 * * * /bin/bash -l -c 'cd /Users/jwo/Projects/texmex && RAILS_ENV=production bundle exec rake margarita:fajitas >> log/cron.log 2>&1'

5,10,15,20,25,30,35,40,45,50,55 * * * * /bin/bash -l -c 'cd /Users/jwo/Projects/texmex && RAILS_ENV=production bundle exec rake guacamole:start_guacamole >> log/cron.log 2>&1'

0 3,15 * * * /bin/bash -l -c 'cd /Users/jwo/Projects/texmex && RAILS_ENV=production bundle exec rake guacamole:tortilla_soup guacamole:grande_combo_de_tejas >> log/cron.log 2>&1'
```

More on Cron and whenever:

* [Cron](http://en.wikipedia.org/wiki/Cron)
* [Cron in Ruby](http://railscasts.com/episodes/164-cron-in-ruby)
* [Whenever gem](https://github.com/javan/whenever)

{% render_partial _includes/custom/mailchimp.html %}


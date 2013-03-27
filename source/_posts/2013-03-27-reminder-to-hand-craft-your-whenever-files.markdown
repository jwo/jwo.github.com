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

Want more Ruby?
---------------

<!-- Begin MailChimp Signup Form -->
<link href="http://cdn-images.mailchimp.com/embedcode/slim-081711.css" rel="stylesheet" type="text/css">
<style type="text/css">
    #mc_embed_signup{background:#fff; clear:left; font:14px Helvetica,Arial,sans-serif; }
    /* Add your own MailChimp form style overrides in your site stylesheet or in this style block.
       We recommend moving this block and the preceding CSS link to the HEAD of your HTML file. */
</style>
<div id="mc_embed_signup">
<form action="http://rubyoffrails.us6.list-manage.com/subscribe/post?u=aebdb7c9619330f7cc1bfcdf9&amp;id=8310b1a92e" method="post" id="mc-embedded-subscribe-form" name="mc-embedded-subscribe-form" class="validate" target="_blank" novalidate>
    <label for="mce-EMAIL">Subscribe for updates about RubyOffRails</label>
    <input type="email" value="" name="EMAIL" class="email" id="mce-EMAIL" placeholder="email address" required style="padding: 15px; width: 80%">
    <div class="clear"><input type="submit" value="Subscribe" name="subscribe" id="mc-embedded-subscribe" class="button"></div>
</form>
</div>

<!--End mc_embed_signup-->

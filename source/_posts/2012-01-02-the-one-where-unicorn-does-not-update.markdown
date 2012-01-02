---
layout: post
title: "The One Where Unicorn Doesn't Update on Deploy"
date: 2012-01-02 13:27
comments: true
categories: unicorn
---

Things that are true:

* We love Github
* Github uses Unicorns
* Github deploys all.the.time

Therefor: We should be using Unicorn instead of Passenger.

However, my recent switch to unicorn wasn't all Unicorns and Rainbows! (pun definitely intended). For me, here's what I saw:

<!-- more -->

## tl;dr

Make sure you uncomment your unicorn.rb code that's checking for .oldbin. You can use these three configs to get a working capistrano with nginx and unicorn.

* [nginx.conf](https://gist.github.com/1551895)
* [config/unicorn/production.rb (Unicorn config)](https://gist.github.com/1551909)
* [config/deploy (Capistrano deploy file)](https://gist.github.com/1551881)

## Symptoms

* Once I'd deploy, I'd refresh the page and see my new app version
* I'd refresh and see my old version
* Refresh again: new. repeat.

Once I deployed, I'd see something like this:
```
$ pgrep -lf unicorn_rails
5877 unicorn_rails master (old) -c /u/apps/recruiter-blast/current/config/unicorn/production.rb -E production -D
30123 unicorn_rails worker[0] -c /u/apps/recruiter-blast/current/config/unicorn/production.rb -E production -D
30126 unicorn_rails worker[1] -c /u/apps/recruiter-blast/current/config/unicorn/production.rb -E production -D
30129 unicorn_rails worker[2] -c /u/apps/recruiter-blast/current/config/unicorn/production.rb -E production -D
30132 unicorn_rails worker[3] -c /u/apps/recruiter-blast/current/config/unicorn/production.rb -E production -D 
30410 unicorn_rails master -c /u/apps/recruiter-blast/current/config/unicorn/production.rb -E production -D
30429 unicorn_rails worker[0] -c /u/apps/recruiter-blast/current/config/unicorn/production.rb -E production -D
30432 unicorn_rails worker[1] -c /u/apps/recruiter-blast/current/config/unicorn/production.rb -E production -D
30435 unicorn_rails worker[2] -c /u/apps/recruiter-blast/current/config/unicorn/production.rb -E production -D
30438 unicorn_rails worker[3] -c /u/apps/recruiter-blast/current/config/unicorn/production.rb -E production -D 
```

I kept expecting the master (old) to someday go away, but it wouldn't unless I manually killed it. Capistrano was spinning up my new instance, but not shutting it down.

## Solution

When I got started with unicorn, I had grabbed a unicorn config from the github blogpost, and the following was commented out. Uncomment it so that it'll kill old processed where the PID does not match the current PID and BAM, you're golden.

```
  old_pid = "#{server.config[:pid]}.oldbin"
  if old_pid != server.pid
    begin
      sig = (worker.nr + 1) >= server.worker_processes ? :QUIT : :TTOU
      Process.kill(sig, File.read(old_pid).to_i)
    rescue Errno::ENOENT, Errno::ESRCH
    end
  end
```

## The Config Files

* [nginx.conf](https://gist.github.com/1551895)
* [config/unicorn/production.rb (Unicorn config)](https://gist.github.com/1551909)
* [config/deploy (Capistrano deploy file)](https://gist.github.com/1551881)

### Stupidity Disclosure

After re-examining the [github unicorn](https://github.com/blog/517-unicorn) blogpost, I see the .oldbin check was definitely NOT commented out. So don't blame the github, but if you're seeing unicorn not refresh your app, this is probably why.

### Copyright on the Gists

The three config files are not copyrighted in any way (as I mostly copied them and edited them anyway). Use and be happy.

---
layout: post
title: "The One Where You Run Rake Commands with Capistrano"
date: 2012-09-10 13:38
comments: true
categories: rails
---

Simple Enough, right? Run some rake tasks on your servers. You think it'd be built in, but nope. Drink some sake and let capistrano do the work!

<!-- more -->
<div class="tldr">
	<span class="heading">tl;dr</span> 
Use my codes to only precompile assets on cap:deploy when there is a change to assets or gems.
</div>

### First Off, Sake is not `sake`

[Defunkt](https://github.com/defunkt) wrote [sake](http://rubygems.org/gems/sake) in ought-eight (2008) for system wide rake tasks. This isn't it; this is just a file, name it what you want.

### Background

I'll use rake tasks to do data-migrations that I don't want to stick around in db/migrations. Or to do one-off things like re-process images.

But it's not all that awesome to actually run those things in production when you need to. Here's a way!

{% codeblock config/recipes/sake.rb lang:ruby %}
namespace :sake do  
  desc "Run a task on a remote server."  
  # run like: cap staging rake:invoke task=a_certain_task  
  task :invoke do  
    run("cd #{deploy_to}/current && bundle exec rake #{ENV['task']} RAILS_ENV=#{rails_env}")  
  end  
end
{% endcodeblock %}

Then, make sure you require it
{% codeblock config/deploy.rb lang:ruby %}
load "config/recipes/assets"
{% endcodeblock %}

And that's it! Now, when you want to run that code, say to migrate a database:

```ruby
cap sake:invoke task="db:migrate"
```

This is not mind-blowing, but it's not very obvious for new deployers, so I wanted to add it here for the googlers.

## Naming Notes

Don't name your task rake -- newer versions of rake (0.9.2) are not pleased with that. That's how I ended on sake. (rake => sake).

## Credit

This [answer on Stack Overflow](http://stackoverflow.com/questions/312214/how-do-i-run-a-rake-task-from-capistrano) had some good stuffs. I modified from there.

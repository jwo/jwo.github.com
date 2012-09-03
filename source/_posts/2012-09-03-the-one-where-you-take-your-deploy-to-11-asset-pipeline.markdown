---
layout: post
title: "The One Where You Take Your Deploy to 11: Asset Pipeline"
date: 2012-09-03 14:56
comments: true
categories:  rails
---

Things I Love: the asset pipeline in Rails. Things I detest: Long deploys caused by recompiling the asset-pipeline when _I KNOW NOTHING HAS CHANGED_. Makes it hard to deploy constantly if each takes 2 minutes.

<!-- more -->
<div class="tldr">
	<span class="heading">tl;dr</span> 
Use my codes to only precompile assets on cap:deploy when there is a change to assets or gems.
</div>

Background:  
* The asset pipeline needs to be precompiled in order to serve one awesome application.js and application.css file
* Using a CSS library like Zurb or Bootstrap adds about 2 minutes to the asset pipeline compilation time

So... Here's a solution to fast-compile your assets by only checking if the changeset includes changes under:

* app/assets 
* lib/assets 
* vendor/assets 
* Gemfile.lock 
* config/routes.rb

The assets is pretty self-explanatory. We compile if the Gemfile.lock changed to catch if something like twitter-bootstrap-rails was added or updated. At first the config/routes.rb seems out of place, but it's in case you load an engine (or remove one).

## The code!

Make sure you're loading deploy/assets

{% codeblock Capfile lang:ruby %}
load 'deploy/assets'
{% endcodeblock %}

{% codeblock config/recipes/assets.rb lang:ruby %}
# -*- encoding : utf-8 -*-                                                                                                    

set :assets_dependencies, %w(app/assets lib/assets vendor/assets Gemfile.lock config/routes.rb)

namespace :deploy do
  namespace :assets do

    desc <<-DESC
      Run the asset precompilation rake task. You can specify the full path \
      to the rake executable by setting the rake variable. You can also \
      specify additional environment variables to pass to rake via the \
      asset_env variable. The defaults are:

        set :rake,      "rake"
        set :rails_env, "production"
        set :asset_env, "RAILS_GROUPS=assets"
        set :assets_dependencies, fetch(:assets_dependencies) + %w(config/locales/js)
    DESC
    task :precompile, :roles => :web, :except => { :no_release => true } do
      from = source.next_revision(current_revision)
      if capture("cd #{latest_release} && #{source.local.log(from)} #{assets_dependencies.join ' '} | wc -l").to_i > 0 
        run %Q{cd #{latest_release} && #{rake} RAILS_ENV=#{rails_env} #{asset_env} assets:precompile}
      else
        logger.info "Skipping asset pre-compilation because there were no asset changes"
      end 
    end 

  end 
end
{% endcodeblock %}

### config/deploy.rb

{% codeblock config/deploy.rb lang:ruby %}
load "config/recipes/assets"
{% endcodeblock %}


## How does it know?

The regular assets:precompile task is getting overridden by our custom task. We then:

1. get the FROM git revision
2. get the git log of changes from the FROM to the TO that include our paths we care about
3. Pipe that into wc to see if there's anything there

If there is, then we call the rake assets:precompile (like calling `super`)


## Output

```
    triggering after callbacks for `deploy:update_code'
  * executing `deploy:assets:precompile'
  * executing "cat /home/deployer/apps/yourapp/current/REVISION"
    servers: ["yourapp.comalproductions.com"]
    [yourapp.comalproductions.com] executing command
    command finished in 921ms
  * executing "cd /home/deployer/apps/yourapp/releases/20120903203634 && git log 2e03e2b18530c86a69ba8a2c2d75909142767f5b.. app/assets lib/assets vendor/assets Gemfile.lock config/routes.rb | wc -l"
    servers: ["yourapp.comalproductions.com"]
    [yourapp.comalproductions.com] executing command
    command finished in 400ms
 ** Skipping asset pre-compilation because there were no asset changes
```

PROOF: 400ms < 2.5 minutes


## Credit

This [Gist](https://gist.github.com/3072362) by [@xdite](https://github.com/xdite) is the direct source for the code. I've used this in several projects and think it's awesome-sauce.

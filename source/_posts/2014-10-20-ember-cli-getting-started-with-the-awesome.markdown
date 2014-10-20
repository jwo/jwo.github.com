---
layout: post
title: "ember-cli: Getting Started with the Awesome"
date: 2014-10-20 13:40
comments: true
categories: ember
---

ember-cli {% fn_ref 1 %} is a command line application that creates a separate project for your ember project.
It is crafted by the Ember core team and extended by the Ember community {% fn_ref  %}.
So: your backend API will be in 1 app/repo, and ember in a seperate one. This is
a good thing for all sorts of reasons, but mostly: Your Front End App has grown
up.

<blockquote class="twitter-tweet" lang="en"><p>So excited by how Ember CLI is
shaping up. This is the story I&#39;ve been wanting to tell about developing web
apps for the last 3 years.</p>&mdash; Tom Dale (@tomdale) <a
href="https://twitter.com/tomdale/status/523265743610060800">October 18,
2014</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

What does ember-cli give you over other-tool.js?

* ember-cli includes the ability to build SCSS, Coffee-Script, include bower assets, and test your code.
It will generate ember components for you in the same style as "rails generate model Customer"
* ember-cli can run all of your front-end tests. Fastly. Without hair-pulling.
* ember-cli can build the code for you (into a dist) folder, which can be deployed, or copied to a Rails public folder.
* ember-cli can also be deployed directly to Heroku, or used to build Cordova applications on iOS and Android.

** Tl;dr ember-cli is the awesome **

However: There is a confusion generating getting-started period where you might be confused by some of
the conventions. (confession: I had to fiddle around to figure stuff out.
Hopefully, you dear reader won't have to do so).


## ES6 Modules
If you’ve used ember before in Rails or Lineman{% fn_ref 3 %}, moving to ember-cli could confuse you a bit — 
ember-cli uses ES6 modules, which you may have never seen before.

Prior to ember-cli, you may have declared your ember data models in this fashion:

** app/js/models/user.js **

```javascript
App.User = DS.Model.extend({
  first_name: DS.attr('string'),
  last_name: DS.attr('string'),
  email: DS.attr('string'),
  admin: DS.attr('boolean')
});
```

Things to notice: in this file, we assume that Ember and Ember Data have already been loaded, 
and are in the global namespace ready for use. 

ES6, on the other hand, requires you to be explicit about what you want to use — you import namespaces, 
name them what you want, and then use them.

** app/models/recipe.js **

```js
import DS from 'ember-data';

export default DS.Model.extend({
  permalink: DS.attr('string'),
  name: DS.attr('string'),
  ingredients: DS.attr('string'),
  instructions: DS.attr('string'),
  description: DS.attr('string')
});
```

We import the "DS" from ember-data, and it exports the object we define. What we notice:

1.  We never say this is a "Recipe". Instead, the ember-cli "Resolver" knows that this is from recipe.js and 
therefor is a "Recipe".
2.  The "export default" is important. Without it, nothing happens.
3.  It’s extremely important to name files correctly. Extreme Extremeness.


## Getting Sass Working
Coming from Rails, the first thing I want to do is work with Sass. I found a couple of loops you had to 
jump through to get your .scss working again. 

#### Step 1: install brocolli-sass and broccoli-merge-trees

```bash
npm install --save-dev broccoli-sass
npm install --save-dev broccoli-merge-trees
```

If you’re new to npm, the "--save-dev" means "use only in this project".


#### Step 2: edit Brocfile.js

Make it look like this:

```js
/* global require, module */

var EmberApp = require('ember-cli/lib/broccoli/ember-app');

var app = new EmberApp();

var compileSass = require('broccoli-sass');
var mergeTrees = require('broccoli-merge-trees');

var sassSources = [
  'app/styles',
  'vendor/css'
]

var appCss = compileSass( sassSources , 'app.scss', 'assets/app.css');
var appAndCustomDependencies = mergeTrees([app.toTree(),appCss], {
  overwrite: true
});
// EXPORT ALL THE THINGS!
module.exports = appAndCustomDependencies;
```

This will create an asset-pipeline of sorts for you. Broccoli will compile any sass files in app/styles and vendor/css, and compile app.scss into "assets/app.css". 

Generally, this is what you want.

#### Step 3: Customize your sass

Move app/styles/app.css to app/styles/app.scss. You can now use scss all you like:

```scss
body {
  color: lighten(#222222, 20%);
}
```

(We’ll use Bourbon in a future article)


## Creating Routes

```
ember generate route Index
```

This will create app/routes/index.js (and create "routes" directory for you, cool). We’ll change the default to the standard ember starter:

```
import Ember from 'ember';

export default Ember.Route.extend({
  model: function() {
    return ["red", "yellow", "green"];
  }
});
```

And we’ll create a template to show. 

```bash
ember generate template index
```

This created app/templates/index.hbs. We’ll change its contents to:

<script src="https://gist.github.com/jwo/8fdfe5e630d37a07cdce.js"></script>

### Running Locally

```bash
ember server
```

If things are good to go, you’ll see the colors listed out on the screen. WOAH, the power.

### Building for Deploying

```bash
ember build
```

This creates a "dist" directory with "index.html" and other static assets. We can deploy this to S3, or anywhere else. TOTES AWESOME.

### Deploying to Heroku

If we run into CORS problems, or generally want to use Heroku, there’s a build pack for maximum awesome.

```bash
heroku create --buildpack https://github.com/tonycoco/heroku-buildpack-ember-cli.git
git push heroku master
```

At this point, you have an ember app, build with ember-cli, hosted on Heroku. 

We can set an API Proxy to get around CORS problems:

`heroku config:set API_URL=http://api.example.com/`

If you want to checkout a repo where this is hooked up {% fn_ref 3 %} 


## Where to go from here?

*   Check out ember-cli-cordova https://github.com/poetic/ember-cli-cordova 
*   Watch Jake from Poetic Systems’s Houston.JS talk on using this to build mobile apps
* <blockquote class="twitter-tweet" lang="en"><p>My talk on building hybrid apps with ember is up! <a href="http://t.co/iKOB75Dgex">http://t.co/iKOB75Dgex</a> Presented for <a href="https://twitter.com/HoustonJS">@HoustonJS</a> at <a href="https://twitter.com/PoeticSystems">@PoeticSystems</a></p>&mdash; Jake Craige (@jakecraige) <a href="https://twitter.com/jakecraige/status/518037818246189056">October 3, 2014</a></blockquote><script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>
*   Add Ember-Data and use the ember-cli-buildpack’s API proxy {% fn_ref 5 %}
*   Have serious FUN.

### Next Articles in this series
1.  How to Bourbon/Neat/Bitters your ember-cli
1.  How to use environment variables in ember-cli and on heroku
1.  How to customize the JSON you receive from someone’s API and play nicely with ember-data

<script src="https://app.convertkit.com/landing_pages/862.js?orient=horz"></script>

{% footnotes %}
  {% fn %} [ember-cli](http://www.ember-cli.com/)
  {% fn %} (seriously, check out [emberaddons.com](http://www.emberaddons.com/)),
  {% fn %} [jwo/embereno-buildpack-test](https://github.com/jwo/embereno-buildpack-test)
  {% fn %} [LinemanJS.com](http://www.linemanjs.com/)
  {% fn %} [GitHub/Jwo/weeatt-ember-cli](https://github.com/jwo/weeatt-ember-cli)
{% endfootnotes%}


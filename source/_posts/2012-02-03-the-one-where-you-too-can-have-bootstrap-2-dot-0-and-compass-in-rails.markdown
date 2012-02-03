---
layout: post
title: "The One Where You Too Can Have Bootstrap 2.0 and Compass in Rails"
date: 2012-02-03 11:11
comments: true
categories: compass, bootstrap
---

The situation: You've used compass in the past (even when it was hard to setup with rails), and love Bootstrap. You love bootstrap 2.0 and <3 Responsive design but don't use LESS (because obvs). What do you do? Let's make this happen

<!-- more -->

tldr;

* Demo Site: [http://compass-bootstrap2-rails.heroku.com/](http://compass-bootstrap2-rails.heroku.com/)
* Githubs: [https://github.com/jwo/compass-bootstrap2.0-rails](https://github.com/jwo/compass-bootstrap2.0-rails)

Here's what the site looks like:
###### Full Resolution
![Full Resolution](https://img.skitch.com/20120203-8745c3xwt1y7d7ccb761rw5xh8.png)

###### Smaller Screen Size (tablet'ish)
![Tablet Size](https://img.skitch.com/20120203-rjfh38jhj2u582ai32gmxnnp7g.png)

###### iPhone / Mobile size
![iPhone Size](https://img.skitch.com/20120203-kagxp99s948wc4fphbbypdgrer.png)

How to do these things?
----

It's fairly simple to do thanks to the awesomeness that is Chris Epstein. Props also to Thomas McDonald for bootstrap-sass.

In your gem file, do these things:

```
gem "compass-rails", ">= 0.12"
gem "bootstrap-sass", :git => "git://github.com/thomas-mcdonald/bootstrap-sass.git", :branch => "2.0" 
```

In your application.scss import bootstrap using sass, not sprockets. 

```
@import "bootstrap";
@import "bootstrap/responsive";
```

I wish it were more interesting, but Chris Epstein made compass drop dead simple to integrate.

######Version Disclosure: This post worked with compass-rails 0.12, bootstrap css 2.0, rails 3.2, and a brand off bootstrap-sass.

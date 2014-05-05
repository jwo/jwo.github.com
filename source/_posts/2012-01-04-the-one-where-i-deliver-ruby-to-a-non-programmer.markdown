---
layout: post
title: "The One Where I Deliver Ruby to a Non Programmer"
date: 2012-01-04 13:09
comments: true
categories:  ruby
---

The Situation: Write a program that parses XML files into a custom CSV format. So I do this, and write the app using TDD with no user interface.

To test it against live data, I slap a ruby script on it that I can run from the command line, show it to the client and he's ecstatic. Now to deliver the command line app to him.

Except... Client does not program ruby. Client has the standard Mac setup with ruby 1.8.7 installed with no GCC compiler.

So. I created a [wiki cheat sheet](https://github.com/jwo/jwo.github.com/wiki/Ruby-1.9.2-install-for-Clients-on-OSX) for client to view (using this blog Github pages). With no xCode setup!

<!-- more -->

[Ruby OSX Install Cheat Sheet for Non Ruby Developers](https://github.com/jwo/jwo.github.com/wiki/Ruby-1.9.2-install-for-Clients-on-OSX)

Steps:

1. Install Compiler
2. Install RVM
3. Source the profile
4. Install Homebrew
5. Install nokogiri
6. Install bundler
7. run bundler

See anything I left off?

Tangent: You should be using GLI for command line apps. I have slides on [ZOMG Command Line](https://github.com/jwo/Slides/tree/master/ZOMG-command-line), and you should also read davetron5000's [slides on the subject](http://awesome-cli-ruby.heroku.com/). And read his book: [Build Awesome Command-Line Applications in Ruby](http://pragprog.com/book/dccar/build-awesome-command-line-applications-in-ruby).

###### Version Disclosure: This stuff was accurate with OSX Lion, as of January 4, 2012. It installs ruby 1.9.2, homebrew 0.8, nokogiri 1.5.0, and GLI 1.4.0

###### Update: This post originally mentioned Dave Copeland's handle as dave3000, and corrected it to [Davetron5000](https://twitter.com/#!/davetron5000). I regret the missing Tron and 2000.

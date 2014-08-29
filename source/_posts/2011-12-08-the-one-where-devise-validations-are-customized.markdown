---
layout: post
title: "The One Where Devise Validations are Customized"
date: 2011-12-08 20:41
comments: true
categories: Rails, Devise
---

The situation: You use Devise, and want to make emails optional because you login via username and why bother with emails? 

Or maybe you want to be able to re-use email addresses as logins, say across subdomains. Or even multiple email addresses?

Your problem: Devise validatable hates on this and laughs are your attempts.
<!-- more -->

Keep the sweet Devise format validations, and customize devise validations by rolling your own
----------------------------------------------------------------------------------------------
First, remove the :validatable argument from the devise method

~~~
devise :database_authenticatable, :token_authenticatable 
~~~
{:lang="ruby"}

Then, you'll need to implement your own validations. Depending on your intentions, either include or exclude `validates_presence_of   :email`

~~~
  validates_uniqueness_of	:email, 	:case_sensitive => false, :allow_blank => true, :if => :email_changed?
  validates_format_of	:email,	:with  => Devise.email_regexp, :allow_blank => true, :if => :email_changed?
  validates_presence_of	:password, :on=>:create
  validates_confirmation_of	:password, :on=>:create
  validates_length_of	:password, :within => Devise.password_length, :allow_blank => true
~~~
{:lang="ruby"}

I know the above looks vaguely familiar... say: what you used to write all the time before Devise kicked in. But [look at the devise validatable source](https://github.com/plataformatec/devise/blob/master/lib/devise/models/validatable.rb) -- this is all it's doing anyway.

To allow email re-use across accounts, add `:scope=>:accont_id` on the uniqueness validation.

Also cool: use the `Devise.email_regexp` and `Devise.password_length` to get good email validations.

######Version Disclosure: This post was valid with Devise 1.4.6 and Rails 2.3/3.0/3.1.

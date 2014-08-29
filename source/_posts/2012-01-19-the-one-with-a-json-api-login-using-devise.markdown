---
layout: post
title: "The One With a JSON API Login Using Devise"
date: 2012-01-19 07:28
comments: true
categories: devise api ruby
---

The situation: You need to add an iOS app to your Rails application. Users can login to both locations, and you're using Devise for authentication.

The problem: How do you authenticate users on the iPhone using an email/password created on the website? And how do you tell Devise not to redirect to a login page when you're using a JSON API?

<!-- more -->
<div class="tldr">
	<span class="heading">tl;dr</span> Use this gist for an <a href="https://gist.github.com/1255275">API JSON Devise Sessions Controller</a> that does not redirect. 
</div>

## Background
Devise provides an awesome :token_authenticatable setup where you can login a user using an "auth_token" query_string. 

~~~
/api/recipes?qs=sweet&auth_token=[@user.auth_token]
~~~
{:lang="ruby"}

Then, in your Api::RecipesController, you'd do something like:

~~~
class Api::RecipesController < Api::BaseApiController
  before_filter :authenticate_user!

  respond_to :json
  def index
    @recipes = Recipe.search(params.fetch(:qs, ""))
  end
end
~~~
{:lang="ruby"}

The above will authenticate the user by cookies/session/token, and return recipes in JSON.

But, _How do you get the token?_

## Authentication and Login

We'll let the user enter their email and password on the iPhone, and have the endpoint return the token in JSON if valid. We'll want an HTTP Status code of 401 if it's invalid.

We'll have a Base API Controller from which we inherit

~~~
class Api::BaseApiController < ApplicationController
  respond_to :json
end
~~~
{:lang="ruby"}

First, you'll create an api endpoint for your session:

~~~
class Api::SessionsController < Api::BaseController
  prepend_before_filter :require_no_authentication, :only => [:create ]
  include Devise::Controllers::InternalHelpers

  def create
    build_resource
    resource = User.find_for_database_authentication(:login=>params[:user_login][:login])
    if resource.nil?
      render :json=> {:success=>false, :message=>"Error with your login or password"}, :status=>401
    end

    if resource.valid_password?(params[:user_login][:password])
      sign_in("user", resource)
      render :json=> {:success=>true, :auth_token=>resource.authentication_token, :login=>resource.login, :email=>resource.email}
    else
      render :json=> {:success=>false, :message=>"Error with your login or password"}, :status=>401
  end
~~~
{:lang="ruby"}

The problem is that you don't get the 401 Status code. You get a 302 redirect to the user login page. And that dog won't hunt on an iPhone app.

## WTH Devise?

Devise is relying on Warden, and warden is doing what it's told. If not a valid login, it'll redirect away. So, we need to tell Warden we're going to have a custom failure like so:

~~~
warden.custom_failure!
~~~
{:lang="ruby"}

Refactored down slightly, we get:

## The Solution

{% gist 1255275 %}


Your users will receive something this this on success:
~~~
{
    "success": true,
    "auth_token": "dfc7ad3884e7",
    "login": "emailexample",
    "email": "email@example.com"
}
~~~
{:lang=js}

And this on failure (with a status code of 401). 
~~~
{
    "success": false,
    "message": "Error with your login or password"
}
~~~
{:lang=js}

######Version Disclosure: This post was valid with Devise 1.4.6 and Rails 3.0/3.1.


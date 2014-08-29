---
layout: post
title: "The one where we DRY up Mongoid and Rspec using Shared Examples and Modules"
date: 2012-02-29 10:19
comments: true
categories: mongoid rspec
---

The Situation: You're adding Addresses to seemingly every document you have. Doctors, Patients, Insurance Agents. Even the Delivery Man -- everybody has a contact.

The Problem: You're tempted to copy/paste the field and validations, but your spidy-senses tell you not to.

Let's DRY this up using modules and rspec shared examples. We'll end up with a way you _could_ enable fast rails tests on your Mongoid models.

<!-- more -->

<div class="tldr">
	<span class="heading">tl;dr</span> Shared examples describing a module. Include module in a mongoid class: <a href="https://gist.github.com/1942427">The Gist</a>.
</div>

Some Assumptions Before We Begin
--------------------------------

1. You have a project setup with Mongoid (2.2+, Rspec 2.8+, and rspec-mongoid)
2. You are awesome

OK, so we would normally have a model for the Doctor like so:


~~~
class Doctor
	include Mongoid::Document
	include Mongoid::Timestamps

	field :first_name, type: String
	field :last_name, type: String
	field :phone_number, type: String
	field :address_line_1, type: String
	field :city, type: String
	field :state, type: String
	field :postal_code, type: string
	field :email, type :String
	validates_presence_of :last_name
	validates_presence_of :first_name
end
~~~
{:lang="ruby"}

### Step 1: Write those tests first

~~~
require 'spec_helper'

describe Doctor do
	it { should have_fields(:first_name, :last_name) }
	it { should have_fields(:phone_number, :email, :address_line_1, :city, :state, :postal_code) }
	it { should validate_presence_of(:first_name) }
	it { should validate_presence_of(:last_name) }
end
~~~
{:lang="ruby"}

Note: If this doesn't pass, you may need to add  configuration.include Mongoid::Matchers in your rspec. See the [mongoid-rspec gem](https://github.com/evansagge/mongoid-rspec) for use.

### Step The Second: Extract the mongoid definitions

~~~
require 'spec_helper'

module Contactable

  def self.included(receiver) 
    receiver.class_eval do
      field :first_name, type: String
      field :last_name, type: String
      field :phone_number, type: String
      field :address_line_1, type: String
      field :city, type: String
      field :state, type: String
      field :postal_code, type: String
      field :email, type: String
      validates_presence_of :first_name
      validates_presence_of :last_name
    end
  end
end

class Doctor
  include Mongoid::Document
  include Mongoid::Timestamps
  include Contactable
end
~~~
{:lang="ruby"}

And run those tests!

~~~
....

Finished in 0.11461 seconds
4 examples, 0 failures
~~~

### Step 3: Extract the Tests!

We'll extract the tests out into a shared_examples file in the support folder, and then tell the Doctor class that it should behave like a Contact. 

~~~
#spec/support/shared_examples.rb
shared_examples_for Contactable do
	it { should have_fields(:first_name, :last_name) }
	it { should have_fields(:phone_number, :email, :address_line_1, :city, :state, :postal_code) }
	it { should validate_presence_of(:first_name) }
	it { should validate_presence_of(:last_name) }
end

#spec/models/doctor_spec.rb

require 'spec_helper'
describe Doctor do
	it_behaves_like Contactable
end
~~~
{:lang="ruby"}

Run those tests!  Could be faster

~~~
....

Finished in 0.13347 seconds
4 examples, 0 failures
~~~

All is going awesome! So we can now easily add contact information (and validations), to say users:

~~~
class User
  include Mongoid::Document
  include Mongoid::Timestamps
  include Contactable
end

describe User do
  it_behaves_like Contactable
end
~~~
{:lang="ruby"}

Now your Users have first_name, last_name, address, and more!

### Let's Take this to 11

Now we can very, very easily isolate the Doctor out for very fast tests.

~~~
module Mongoid
	module Document; end
  module Timestamps; end
end
class Pharmacy; end

require_relative "../../app/models/doctor"
require_relative "support/shared_examples"

describe Doctor do
  it_behaves_like Contactable

	it "will tell the pharmacy to release the rx's" do
		Pharmacy.should_receive(:release_the_rx!)
		subject.gimme!
	end
end

~~~
{:lang="ruby"}

The above would fail since #gimme! doesn't exist, but it would be:

* In total isolation from Rails
* Total isolation from Mongo DB
* fast!

I created the following to easily add re-usable data-bags to documents in Mongoid, rather than copying and pasting. The same shared-example technique works well in ActiveRecord.

###### Version Disclosure: written against mongoid-rspec 1.4, mongoid 2.4, rspec 2.8.


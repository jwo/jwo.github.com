---
layout: post
title: "Getting Websockets, CarrierWave, and Fog to play nicely together"
date: 2015-03-05 11:30
comments: true
categories: 
---

Getting Websockets, CarrierWave, and Fog to play nicely together

Alternate title: "Dancing with Dragons, or, how I got screwed by ActiveRecord Callbacks yet again."

I’ve been working on the following scenario:

* 2 different people have an ember app open
* One of them adds a new record to the Rails app
* I want other-person’s ember to refresh with that new record instantaniously-ish

### The setup

```ruby
class Photo < ActiveRecord::Base
  mount_uploader :image, ImageUploader

  after_create do
    Pusher.notify("new_photo", PhotoSerializer.new(self).attributes)
  end
end
```

Fairly simple, but here’s what’s going on:  

* When our photo is created, carrier wave will resize the image, upload it to S3
* We will send the JSON through Pusher to any clients subscribed to the `new_photo` channel. The JSON we’re sending is the same JSON that ActiveModelSerializer will use in the API. WIN.


## But then, melancholy.

The JSON that comes through has something rather odd for `large_image_url` - it’s a local path to the /tmp/upload directory, which absolutely does not work. :/

```json
{
    "id": "399ca80efb2bd65e7254f299adfe278a",
    "name": "thumbythumbythumby",
    "large_image_url": "/uploads/tmp/1425583084-37430-8620/large_thumbnail.jpg",
    "thumb_photo_url": "/uploads/tmp/1425583084-37430-8620/thumb_thumbnail.jpg"
}
```

Basically, when Pusher is sending PhotoSerializer.new(self).attributes, the image has not been stored up to S3. We want to do Pusher later, after fog does it’s thing.

Callbacks are run in this order:


```ruby
before_validation
after_validation
before_save
around_save
before_create
around_create
after_create
after_save
after_commit/after_rollback
```

So, `after_create` is occurring before `after_save`. Let’s switch that up and our problems should be solved:

```ruby
class Photo < ActiveRecord::Base
  mount_uploader :image, ImageUploader

  after_save on: :create do
    Pusher.notify("new_photo", PhotoSerializer.new(self).attributes)
  end
end
```

Result: Same thing. huh?

To see the order in which callbacks are run, we can do this:

```ruby
Photo._save_callbacks.map(&:filter)
```

That will show the name of the callbacks being called, and will look something like:

```ruby
[70220584822820, :remove_previously_stored_photo, :store_photo!, :write_photo_identifier]
```

That 70220584822820 is our block callback. And it’s first. Why is it first? Can we get it to go last?  Callbacks are run in a last-first order. Further, changing it to something like the following also doesn’t work.

```ruby
class Photo < ActiveRecord::Base
  mount_uploader :image, ImageUploader

  after_save on: :create do
    Pusher.notify("new_photo", PhotoSerializer.new(self).attributes)
  end
end
```

### So then what the hell works?

We need to store the image before we get the attributes. So, let’s add that `store_image!` callback we saw in the list of callbacks. This will result in `store_image!` being called twice, but it won’t actually store the image twice — the second call is basically a no-op.

```ruby
class Photo < ActiveRecord::Base
  mount_uploader :image, ImageUploader

  after_save on: :create do
    store_image! ## It’s this. this is carrierwave saving the image.
    Pusher.notify("new_photo", PhotoSerializer.new(self).attributes)
  end
end
```

```json
{
    "id": "399ca80efb2bd65e7254f299adfe278a",
    "name": "thumbythumbythumby",
    "large_image_url": "https://sevensevenseven-dev.s3.amazonaws.com/uploads/photo/photo/32/large_thumbnail.jpg",
    "thumb_photo_url": "https://sevensevenseven-dev.s3.amazonaws.com/uploads/photo/photo/32/thumb_thumbnail.jpg"
}
```

### IT WORKS. REJOICE. Then: Anger.

CALLBACKS!!! 

![khallbacks](/images/khallbacks.jpg)

It reminds me of a [story about a younger Rails developer warning other Rails devs about the dangers of Callbacks, and how they tend to screw you over](http://jessewolgamott.com/blog/2012/06/06/railsconf-2012-ignite-talk-activerecord-and-velveeta/).

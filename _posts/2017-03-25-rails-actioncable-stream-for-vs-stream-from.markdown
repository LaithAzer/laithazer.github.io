---
layout: post
title: Rails ActionCable&#58 stream_for vs stream_from
date: 2017-03-25 14:00
comments: true
external-url:
category: 'Ruby on Rails'
---

ActionCable is the new WebSockets integration feature in Ruby on Rails 5+. It has several great features, one of which is the ease of creating new channels.

When creating a new ActionCable channel class, you have two possible methods for creating the new channel name:

* `stream_from`: expects a string
* `stream_for`: expects an object

The difference is not entirely clear when following the ActionCable documentation, so I will show two scenarios where it could be useful:

### stream_from

The more straightforward of the two options. It just expects any string. For example, if I had different chat rooms that users could join, my class would look something like this:

{% highlight ruby %}
class ChatChannel < ApplicationCable::Channel
  def subscribed
    stream_from "chat_channel_#{params[:id]}"
  end
end
{% endhighlight %}

This way, we could use the same class to create multiple chat channels based on the `id` param.

On the client, we would create the new subscription as follows:

{% highlight javascript %}
App.chat_1 = App.cable.subscriptions.create({channel: "ChatChannel", id: 1});
{% endhighlight %}
On the server, we would get this in the logs:

{% highlight shell %}
ChatChannel is transmitting the subscription confirmation
ChatChannel is streaming from chat_channel_1
{% endhighlight %}

So now we can just handle subscribing to different chat rooms by passing in an id parameter.

When we want to publish something to a specific chat channel, we would use the following ActionCable method:

{% highlight ruby %}
ActionCable.server.broadcast("chat_channel_1", "Hello World")
{% endhighlight %}

The server logs would look like this:

{% highlight shell %}
[ActionCable] Broadcasting to chat_channel_1: "Hello World"
ChatChannel transmitting "Hello World" (via streamed from chat_channel_1)
{% endhighlight %}

So now, any user suscribed to `chat_channel_1` will receive that message.

### stream_for

This is the less straightforward option in my opinion. The reason is mainly that you now have to pass in an object (an instance of any model) instead of a string. A useful scenario could be if we want to subscribe each logged in user to their own channel for receiving notifications. The channel code would look like this:

{% highlight ruby %}
class UserChannel < ApplicationCable::Channel
  def subscribed
    stream_for current_user
  end
end
{% endhighlight %}

This is assuming you have set up a connection identifier called `current_user` in your Connection class.

On the server logs, a subscription would look like this:

{% highlight shell %}
UserChannel is transmitting the subscription confirmation
UserChannel is streaming from user:Z2lkOi8vYWMtbG5sL1VzZXIvMQ
{% endhighlight %}

So now instead of a string, we get an identifier based on the user object we passed in to the `stream_for` method.

On the client the subscription creation would be just like the previous example:

{% highlight javascript %}
App.user = App.cable.subscriptions.create("UserChannel")
{% endhighlight %}

For broadcasting to this channel, we would use the built in class method `broadcast_to` (instead of just `broadcast` as in the previous example):

{% highlight ruby %}
user = User.find(data['user_id'])
UserChannel.broadcast_to(user, "hi")
{% endhighlight %}

In the above example, we found the user we wanted to broadcast to (what you use to find the user doesn't matter, you just need to load the correct user object). Once we find the user, we pass the actual user object into the method, and it will broadcast to that user.

The server logs will now look like this:
{% highlight shell %}
[ActionCable] Broadcasting to user:Z2lkOi8vYWMtbG5sL1VzZXIvMQ: "hi"
UserChannel transmitting "hi" (via streamed from user:Z2lkOi8vYWMtbG5sL1VzZXIvMQ)
{% endhighlight %}

So now only that one unique user subscribed to that channel will receive the message

## Summary

Both ways of creating channels behave the same: you can subscribe to a channel and publish to a channel. The main difference to keep in mind is your use case: if you want to bind a channel to a model in your app (a User for live notifications, an Article for live comments, an Order for live updates, etc.), then use `stream_for`, otherwise you can keep it generic and use `stream_from`.

{% if page.comments %}

{% endif %}
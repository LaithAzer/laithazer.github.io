---
layout: post
title: How to Verify Users in ActionCable With Rails API Mode
date: 2018-02-10 00:00
comments: true
external-url:
category: 'Ruby on Rails'
---

Authenticating your users when connecting to your websocket is very important, as you don't (usually) want just anyone to be able to send and receive on your websocket. The out-of-the-box solution with verifying users in ActionCable is highly dependent on your app being monolithic and using cookies/sessions.

More and more people are using Rails as just the API layer. This usually involves some form of token-based authentication system that is sent in the request header. I could not find a way online of how to authenticate your users with this system, so I have come up with this:

## Front-end

In your front-end app, let's say React in this case, use the ActionCable npm package to connect to your websocket as usual:

{% highlight javascript %}
const cableUrl = 'ws://localhost:3000/cable';
const consumer = ActionCable.createConsumer(`${cableUrl}`);
{% endhighlight %}

Now in order to use the token to verify a user's identity, we can simply append it as a parameter (let's say my component has a prop of currentUser):

{% highlight javascript %}
const cableUrl = 'ws://localhost:3000/cable';
const { accessToken } = this.props.currentUser;
const consumer = ActionCable.createConsumer(`${cableUrl}?token=${accessToken}`);
{% endhighlight %}

## Back-end

The default way to verify a user in ActionCable looks something like this:

{% highlight ruby %}
module ApplicationCable
  class Connection < ActionCable::Connection::Base
    identified_by :current_user
 
    def connect
      self.current_user = find_verified_user
    end
 
    private
      def find_verified_user
        if current_user = User.find_by(id: cookies.signed[:user_id])
          current_user
        else
          reject_unauthorized_connection
        end
      end
  end
end
{% endhighlight %}

However, we do not have cookies since our Rails app is just an API layer. So we have to use our token that we passed in as a parameter to verify the user. Generally there are 3 steps to verify the token:

1. Ensure the token exists
2. Ensure the token is not expired
3. Ensure the token is not revoked

After we verfify that the token sent is valid, we can use it to load the `current_user`. This is a simplified version but shows the flow necessary:

{% highlight ruby %}
module ApplicationCable
  class Connection < ActionCable::Connection::Base
    identified_by :current_user
 
    def connect
      self.current_user = find_verified_user
    end
 
    private
      def find_verified_user
        token = AccessToken.where(token: request.params[:token]).first
        if verify_token(token)
          current_user = token.user
        else
          reject_unauthorized_connection
        end
      end

      def verify_token(token)
        token && !token.expired? && !token.revoked?
      end
  end
end
{% endhighlight %}

After that, we can now use the `current_user` object as normal in our channels.

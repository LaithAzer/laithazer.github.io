---
layout: post
title: Rails Interactors - Skinny Models AND Skinny Controllers!
date: 2017-04-16 21:00
comments: true
external-url:
category: 'Ruby on Rails'
---

A common practice in Rails apps is "Skinny Controller, Fat Model". This is a good practice, but eventually most apps will start to have a few models that are way too fat and become [God Objects](https://en.wikipedia.org/wiki/God_object). But what if there was a way to have both skinny models AND skinny controllers?! Good news, there is! Let me tell you about the Interactor pattern.

It's important to understand this pattern does not improve your application performance or user experience, it is just a layer of abstraction. The main objective is to separate business logic. You will actually be writing more code, but as your application grows it will be worth it as you can avoid God Objects, fat controllers/models, huge test files and developer satisfaction will be higher.

The best way to show how an Interactor works is through an example:

## Example

Almost all Rails applications will have some sort of User model. It's probably also one of the first classes created in the app and will inevitably have a lot of people working on it as the app grows. After all, our apps are built for users to interact with!

Let's assume we are building a social app that allows users to join groups, similar to Facebook. The user/groups associations might look something like this:

{% highlight ruby %}
class User < ApplicationRecord
  has_many :user_groups
  has_many :groups, through: :user_groups
end

class Group < ApplicationRecord
  has_many :user_groups
  has_many :users, through: :user_groups
end

class UserGroup < ApplicationRecord
  belongs_to :user
  belongs_to :group
end
{% endhighlight %}

Very simply associations set up. Now let's create an endpoint in our UserGroups controller for users to join groups. However, before a user successfully joins a group, we want to do the following:

* check if the user is old enough to join
* check if the user hasn't hit the arbitrary limit of 10 groups

Following the fat model, skinny controller advice we might end up doing something like this:

{% highlight ruby %}
def create
  # Assume @group is loaded from a before_action method

  if !current_user.is_old_enough_to_join_group?(@group)
    flash[:alert] = "You are too young to join this group."
  elsif current_user.is_group_limit_reached?
    flash[:alert] = "You have joined too many groups already."
  else
    user_group = UserGroup.new(user_id: current_user.id, group_id: @group.id)
    if user_group.save
      flash[:notice] = "You have successfully joined the #{@group.name} group!"
    else
      flash[:alert] = "Could not join group."
    end
  end

  redirect_to group_path(@group)
end
{% endhighlight %}

This endpoint is not *too* bad. It reads okay and is not doing that much (yet). However, we need to create the two methods in our User model, so let's update that:

{% highlight ruby %}
class User < ApplicationRecord
  MAX_NUMBER_OF_GROUPS = 10

  has_many :user_groups
  has_many :groups, through: :user_groups

  def is_old_enough_to_join_group?(group)
    age > group.minimum_age
  end

  def is_group_limit_reached?
    groups.count >= MAX_NUMBER_OF_GROUPS
  end
end

{% endhighlight %}

So already for just ONE piece of our app's business logic, we created TWO methods in our User model 😱. Hopefully you can see as your app grows in complexity, it will be tempting to just throw in more methods into the User model, and before you know it, you will have a God Object!

## Interactor Refactor

Let's clean up our controller method to use an Interactor that we will soon create:

{% highlight ruby %}
def create
  result = AddUserToGroup.call group: @group, user: current_user
  message = result.message

  if result.success?
    flash[:notice] = message
  else
    flash[:alert] = message
  end

  redirect_to group_path(@group)
end
{% endhighlight %}

Much cleaner (in my opinion at least)! Let's take a look at what happened..

We added an AddUserToGroup Interactor, which is just a Ruby class with a specific method named `call`. As shown above, we pass in to the Interactor whatever it needs to perform the business logic (in this case the `group` to join and `current_user`), and we depend on it to inform us whether it was a successful operation. Based on the success, we either show an alert to the user or a notice. The controller is now much cleaner and doesn't care about executing the business logic, it just cares about showing the user a message and redirecting them.

Let's now write our AddUserToGroup Interactor:

{% highlight ruby %}
#add_user_to_group.rb
class AddUserToGroup
  include Interactor

  TOO_YOUNG_ERROR_MESSAGE = "You are too young to join this group.".freeze
  TOO_MANY_GROUPS_ERROR_MESSAGE = "You have joined too many groups already.".freeze
  COULD_NOT_JOIN_GROUP_ERROR_MESSAGE = "Could not join group.".freeze
  SUCCESSFULLY_JOINED_GROUP = "You have successfully joined the #{context[:group].name} group!".freeze

  def call
    fail! message: TOO_YOUNG_ERROR_MESSAGE unless is_old_enough_to_join_group?
    fail! message: TOO_MANY_GROUPS_ERROR_MESSAGE unless group_limit_reached?

    if create_new_user_group
      context[:message] = SUCCESSFULLY_JOINED_GROUP
    else
      fail! message: COULD_NOT_JOIN_GROUP_ERROR_MESSAGE
    end
  end

  private

  def group_limit_reached?
    context[:user].groups.count >= User::MAX_NUMBER_OF_GROUPS
  end

  def is_old_enough_to_join_group?
    context[:user].age >= context[:group].minimum_age
  end

  def create_new_user_group
    context[:user_group] = UserGroup.new(user_id: context[:user].id, group_id: context[:group].id)
    context[:user_group].save
  end
end
{% endhighlight %}

There are a few key things to notice with using the Interactor:

* The arguments passed into the Interactor `call` method are available inside the Interactor through the `context` Hash
* The context Hash can be modified just like any other Hash
* It will be available in the result that is returned when calling the Interactor from the controller, i.e.:

{% highlight ruby %}
result = AddUserToGroup.call group: @group, user: current_user
puts result.user_group
 => #<UserGroup id: 1, user_id: 1, group_id: 1>
puts result.message
=> You have successfully joined the Example group!
{% endhighlight %}

* An Interactor can be failed if the `fail` or `fail!` methods are called, otherwise it will be successful
* The `fail!` method will stop execution of code

## But.. More Lines of Code?! 😨

Yes, we are definitely writing more lines of code, however the benefits outweight the extra effort:

* Developers can find all the code for a particular piece of business logic more easily (it will be contained in our Interactor file)
* It will be much easier to test our controller as we can easily stub our Interactor (controllers are notoriously painful to test in Rails)
* Our code is more aeshetically pleasing and organized (important for developer satisfaction!)
* Our model now does not contain the very specific business logic methods of checking the user's age and their number of groups

Hopefully this short example shows how Interactors can benefit your Ruby on Rails development experience.
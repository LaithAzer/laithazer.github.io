---
layout: post
title: Two Important Pre-Optimizations in Rails
date: 2019-06-22 00:00
comments: true
external-url:
category: 'Ruby on Rails'
---

Pre-optimizing code is generally frowned upon, and for good reason. It's not worth spending expensive developer time on making a part of the codebase very fast if there is no reason for it to be. You might ship the feature and realize your users don't even want it. Maybe it's used so infrequently that nobody really complains that its not blazing fast.

If we apply the Pareto principle to code, it tells us that 80% of our traffic comes from 20% of the codebase. So that means that instead of spending time making every single line of code the fastest it can possibly be, we should instead monitor our code in production and *measure* where slowness actually occurs, then focus on fixing that.

However, I generally always apply these two "pre-optimizations" in Rails code no matter what. I say "pre-optimizations" in quotes because in reality, without performing these actions, there's a big chance your app will crash or your users will get frustrated using your app. Let's take a look at the two:

### 1. Background Jobs

Background jobs are one of the easiest ways to speed up your rails application. The main idea is that, there is usually some work that doesn't have to happens straight away, so it can be deferred, or done in the "background". A few of these are:

* Sending emails
* Audit logs
* Mobile notifications
* Processing media
* Batch processing jobs

Let's take the classic example of a user registering on your website. Without background jobs, we would create the user record in the database, then send an email to the user confirming their registration, and finally redirect the user to another page.

Using background jobs, we would only schedule sending an email to the user, and immediately redirect them after successful creation in the database. This saves time for the user to see feedback, but also saves resources on your web server as the work will be offloaded to another process running background jobs.

### 2. Finding in Batches

A common piece of rails functionality is iterating over a collection and updating it. For example, maybe you need to resend the terms of consent to your users. This could look like this:

```ruby
User.all.each do |user|
  TermsOfConsentMailer.send_latest_terms(user.id).deliver_later
end
```

This might look pretty good. We are using deliver_later, which we just mentioned above regarding sending emails in the background being good practice. Also we only pass in the id to the mailer params, saving a bit of memory from passing in the whole object.

However, what if we have 5 million users in our database? That's at least 5 million objects loaded into memory at once!

I've seen this happen many times, and it often crashes the application. Even if it doesn't, Ruby won't be able to reclaim the used memory until Garbage Collection kicks in.

The proper way to do this is using [batches](https://api.rubyonrails.org/classes/ActiveRecord/Batches.html). By default, it will only load 1000 records at once in memory, saving you from overloading your application. So our code becomes like this:

```ruby
User.all.find_in_batches(batch_size: 500) do |group|
  group.each do |user|
    TermsOfConsentMailer.send_latest_terms(user.id).deliver_later
  end
end
```

This one is actually hard to capture in profiling tools like New Relic. I've seen it often misreported as a slow query, but my guess is that New Relic cannot differentiate between the app level and database level when such a huge collection is loaded, and it will just bring the application down.

A further optimization is just loading the id, as that's all we need, thus saving even more memory:

```ruby
User.select('id').all.find_in_batches(batch_size: 500) do |group|
  group.each do |user|
    TermsOfConsentMailer.send_latest_terms(user.id).deliver_later
  end
end

```
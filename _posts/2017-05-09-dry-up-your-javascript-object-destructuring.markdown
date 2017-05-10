---
layout: post
title: DRY up your JS - Object Destructuring
date: 2017-05-09 20:00
comments: true
external-url:
category: 'JavaScript'
---

When ES6 was released it introduced a lot of new features to the JS community. Reactions were initially mixed, but as time has passed people are beginning to see the benefits.

One of my favourite features is object destructuring. It seems to be influenced by [pattern matching](https://en.wikipedia.org/wiki/Pattern_matching) from functional programming. If you have not seen pattern matching before, do not worry, as object destructuring is pretty simple and can be demonstrated by a few examples:

## Grabbing Multiple Values

Let's say we have a JSON payload that looks something like this:

{% highlight javascript %}
payload = {
  "user": {
    "firstName": "Laith",
    "lastName": "Azer",
    "occupation": "Software Developer",
    "address": {
      "city": "Toronto",
      "province": "Ontario"
    }
  },
  "status": {
    "numberOfBlogPosts": "4",
    "url": "laithazer.com"
  }
}
{% endhighlight %}

Let's say now we want to grab the first name, city, and url from the payload. What I've seen many people especially those with less experience in JS do is:

{% highlight javascript %}
const firstName = payload.user.firstName;
const city = payload.user.address.city;
const url = payload.status.url;
{% endhighlight %}

This works fine, but if we use destructuring it can all happen on one line:

{% highlight javascript %}
const { user: { firstName, address: { city } }, status: { url } } = payload;
{% endhighlight %}

Woah, there's a lot going on in that one line. You are right. To the untrained eye, it can seem daunting, but once you understand what to look for, it becomes quite elegant to read.

Let's break it down into a few steps:

##### 1. Grab the user object

We first grab the user object from the payload, this should be straightforward:

{% highlight javascript %}
const user = payload.user;
// the same as writing
const { user } = payload;
{% endhighlight %}

##### 2. Grab the firstName

Cool! So we learned how to grab one level deep from the object using that new syntax `const { user }`. This is just telling ES6 to look for that user object inside the payload object. Let's now go one level deeper to grab the firstName from the user:

{% highlight javascript %}
const firstName = payload.user.firstName;
// the same as writing
const { user: { firstName } } = payload;
{% endhighlight %}

Nice, we're getting there. In this step we `pattern matched` with the payload to grab the firstName. Notice in the payload the firstName is nested inside the user object, so we use the same pattern by writing `{ user: { firstName } }`.

##### 3. Grab the city

This one should be straighforward, we are now going 3 levels deep but it's the same concept exactly:

{% highlight javascript %}
const firstName = payload.user.firstName;
const city = payload.user.address.city;
// the same as writing
const { user: { firstName, address: { city } } } = payload;
{% endhighlight %}

##### 4. Grab the url

Again straightforward, we just grab the status object similarly to how we did for the user object:

{% highlight javascript %}
const firstName = payload.user.firstName;
const city = payload.user.address.city;
const url = payload.status.url;
// the same as writing
const { user: { firstName, address: { city } }, status: { url } } = payload;
{% endhighlight %}

That's it! Pretty simple when you break it down. Now let's look at another common use case where destructuring is useful.

## Function Arguments

You can also use destructuring to grab values straight from function arguments. This is also pretty straight forward and best shown through example:

{% highlight javascript %}
function printInfo(payload) {
  const firstName = payload.user.firstName;
  const city = payload.user.address.city;
  const url = payload.status.url;
  console.log(`${firstName} lives in ${city}! Find them at ${url}`);
}

// We can make this a one-liner:

function printInfo({ user: { firstName, address: { city } }, status: { url } }) {
  console.log(`${firstName} lives in ${city}! Find them at ${url}`);
}

// However the function arguments become really big,
// so somewhere in the middle is fine too:

function printInfo({ user, status }) {
  const { firstName, address: { city } } = user;
  const { url } = status;
  console.log(`${firstName} lives in ${city}! Find them at ${url}`);
}
{% endhighlight %}

Hopefully after reading the first section the above code should be intuitive!
---
layout: post
title: How to use jQuery in React (hint - DON'T!)
date: 2017-05-13 12:00
comments: true
external-url:
category: 'React'
---

A very common mistake jQuery developers make when starting to write React is to want to bring in jQuery in React. This is usually the case for DOM manipulation. In this post I will show two ways to perform DOM manipulation in React without bringing in jQuery. Ideally, DOM manipulation should be a last resort when writing React code; leave the DOM manipulation up to the React engine and use state/props whenever possible.

There are a few valid reasons for manual DOM maniuplation according to the [docs](https://reactjs.org/docs/refs-and-the-dom.html#when-to-use-refs):

* Managing focus, text selection, or media playback.
* Triggering imperative animations.
* Integrating with third-party DOM libraries.

## But What Do I Lose By Using JQuery? 🤔

You don't really *lose* anything. However, there are at least 2 setbacks: 
* extra network overhead to load your front-end
* developer frustration

If experienced React developers look at React components that have any jQuery code in them, it's usually a huge code smell.

## DOM Manipulation in React

For our example, we'll build a simple form in our component and attach an `onSubmit` handler to alert the typed value to the user.

Doing it using jQuery might look something like this:


{% highlight javascript %}

import React, { Component } from 'react';
import $ from 'jquery';

class Example extends Component {
  alertUser = (e) => {
    e.preventDefault();
    const value = $('#user-input').val();
    alert(value);
  }

  render() {
    return (
      <form id="example-form" onSubmit={this.alertUser}>
        <input id="user-input" />
        <button type="submit">Alert!</button>
      </form>
    );
  }
}

export default Example;

{% endhighlight %}

However, let's look at three other simple methods to do this.

#### 1. Ideal React™ way:

As previously mentioned, controlling things like the value of an input should be done through state whenever possible, as React has tons of built-in optimizations for this:

{% highlight javascript %}

import React, { Component } from 'react';

class Example extends Component {
  constructor(props) {
    super(props);

    this.state = {
      value: '',
    };
  }

  handleOnChange = (e) => {
    this.setState({
      value: e.target.value,
    });
  }

  alertUser = (e) => {
    e.preventDefault();
    alert(this.state.value);
  }

  render() {
    return (
      <form id="example-form" onSubmit={this.alertUser}>
        <input id="user-input" value={this.state.value} onChange={handleOnChange} />
        <button type="submit">Alert!</button>
      </form>
    );
  }
}

export default Example;

{% endhighlight %}

Pretty straight forward. However, the point of this post is for scenarios when it's not as straight forward as grabbing the value of an input, and what your options are to do manual DOM manipulation, as follows.

#### 2. Vanilla JavaScript

You might have heard of a website called [youmightnotneedjquery.com](http://www.youmightnotneedjquery.com). The premise of this website is that the vast majority of common DOM manipulations can be done in Vanilla JavaScript. Let's take a look at how a React component will look using this method.

{% highlight javascript %}

import React, { Component } from 'react';

class Example extends Component {
  alertUser = (e) => {
    e.preventDefault();
    const value = document.getElementById('user-input').value
    alert(value);
  }

  render() {
    return (
      <form id="example-form" onSubmit={this.alertUser}>
        <input id="user-input" />
        <button type="submit">Alert!</button>
      </form>
    );
  }
}

export default Example;

{% endhighlight %}

So we now use the native JavaScript `document.getElementById` to grab the input instead. Pretty simple!

#### 3. React Refs

React comes built in with a method to *refer* to components called Refs (short for references). It is quite simple to use: attach a `ref` attribute to the component you want to refer to (in this case our input), and you can refer to it within your main component using `this.refs.refName`.

{% highlight javascript %}

import React, { Component } from 'react';

class Example extends Component {
  alertUser = (e) => {
    e.preventDefault();
    const value = this.userInput.value;
    alert(value);
  }

  render() {
    return (
      <form id="example-form" onSubmit={this.alertUser}>
        <input id="user-input" ref={(input) => { this.userInput = input; }} />
        <button type="submit">Alert!</button>
      </form>
    );
  }
}

export default Example;

{% endhighlight %}

## Summary

In this post we went over why we would NOT want to use jQuery in React, and alternatives to doing common jQuery DOM manipulations in React, when you need it. The most common ways are using [Vanilla JavaScript](http://youmightnotneedjquery.com/) and [React Refs](https://facebook.github.io/react/docs/refs-and-the-dom.html).
---
layout: post
title: 3 different ways to load your data in React
date: 2017-05-01 20:00
comments: true
external-url:
category: 'React'
---

React is one of the best front-end JavaScript frameworks in use right now (disclaimer: this is my *opinion*). It has very quickly become the market leader for fast, modern front-end frameworks. This post is not about why React is so great, but it is more to talk about the different ways I have come accross when it comes to doing something extremely common in your front-end code: loading/fetching data.

## Community Divided

There isn't really a "best practice" or "convention" when it comes to loading your data in React. This is actually a good thing, as it maintains the awesome part of React being very un-opinionated and letting you decide how you want to design your app.

However, too much flexibilty can lead to multiple developers on the same team doing the same thing in different ways. There is usually a style guide at most companies or a consensus about how certain actions should be done.

In this post, I will go through 3 different ways I have come accross for loading data in React. You can compare the pros/cons yourself and decide what approach would be best for you to do. After all, they all accomplish the same thing!

1. <a href="#1-container-components">Container Components</a>
2. <a href="#2-redux-async-actions">Redux Async Actions</a>
3. <a href="#3-hybrid-approach">Hybrid Approach</a>

### 1. Container Components

Dan Abramov popularized the container/presentational component pattern back in [March 2015](https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0). With this pattern, we would have container components that usually load data inside the `componentDidMount` method, and once the data is loaded we render the presentatonal components. A simple example is as follows:

Let's say we want to fetch the list of posts from the soccer subreddit and just dislay the title of each post as divs. Our container component would thus be responsible for fetching the data and would be as follows:

{% highlight javascript %}
import React, { Component } from 'react';
import axios from 'axios';
import SubredditPresenter from './SubredditPresenter.js';

export default class SubredditContainer extends Component {
  constructor(props) {
    super(props);

    this.state = {
      isLoading: true,
      posts: []
    };
  }

  componentDidMount() {
    axios.get('https://www.reddit.com/r/soccer.json')
      .then(response => {
        this.setState({
          isLoading: false,
          posts: response.data.data.children.map(post => post.data.title)
        })
      })
      .catch(error => console.log('oops'))
  }

  renderLoadingMessage = () => {
    return <div>Loading..</div>
  }

  renderPosts = () => {
    return this.state.posts.map(post => {
      return <SubredditPresenter post={post} key={post} />
    })
  }

  render() {
    return (
      <div>
        { this.state.isLoading ? this.renderLoadingMessage() : this.renderPosts() }
      </div>
    )
  }
}
{% endhighlight %}

There are a few things to take note of here:

* We have a boolean on the state the determines when loading data is completed
* We are using the Promise based [axios](https://github.com/mzabriskie/axios) package for performing our HTTP request
* We are dependent on the *component* state to store our data

This approach is straight-forward and the code is still kept clean, as our presentational component can now be a simple functional component as follows:

{% highlight javascript %}
import React from 'react';

const SubredditPresenter = ({ post }) => {
  return (
    <div>{post}</div>
  )
}

export default SubredditPresenter;
{% endhighlight %}

##### Pros

* Easy to write/understand
* Very little dependency on other libraries
* Separates concerns (i.e. Container and Presentational Components)

##### Cons

* Not reusable: unfortunately if we want to use the list of posts somewhere else on our app, we either have to load the data again or somehow make the container determine which presentational component to render, adding complexity quickly

Only 1 con in my opinion for this method, but it is pretty major as any even moderately complex application will definitely share data accross different pages/components

### 2. Redux Async Actions

Redux is one of the most popular libraries for handling Application State right now. It uses the Flux implementation of loading data where you dispatch three differnet actions:

1. Request began
2. Request successful
3. (or) Request failed

The documentation on this approach is great, and you should [check it out](http://redux.js.org/docs/advanced/AsyncActions.html) if you are interested in the specifics.

##### Pros

* Excellent separation of different states of your app
* Data is saved in the store so can be used from within any component connected to your store
* A good pattern that will lead to well-organized files

##### Cons

* Lots of boilerplate
* Potentially bloats the store with data that only one component needs
* Not easy to understand the first time doing it

### 3. Hybrid Approach

My preferred way of handling data loading in React apps is to use a hybrid of both approaches. Stick with using the container/presentational pattern first, and only if you see yourself needing a datum in multiple components, then refactor to put it in the store.

I find with this approach you get the best of both worlds. You can write code fast, avoid unneccessary boilerplate, separate your concerns and share your data accross different components if needed.

One major concern I have heard from developers when I use this approach is that they don't like to mix two different patterns. This is valid but my arguments are as follows:

* Both patterns are accepted and battle-tested in the React community
* You do not need to bloat your application state with data that only one component uses; it will become a pain to manage after a while
* This approach allows you to build features fast

## Summary

The React community will likely never reach a consensus on what the *best* way to load data is. I have outlined 3 different appoaches in this post that I believe are all valid, and it is up to the team to decide which works best for them.

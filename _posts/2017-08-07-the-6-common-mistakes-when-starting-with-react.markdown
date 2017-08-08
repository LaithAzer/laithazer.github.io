---
layout: post
title: The 6 Most Common Mistakes When Learning React
date: 2017-08-07 00:00
comments: true
external-url:
category: 'React'
---

I have been mentoring students at Lighthouse Labs, an intense web development bootcamp in Toronto, for almost 2 years now. Throughout those two years I have interacted with probably over 200 students, from debugging their code, teaching them new concepts and advising them on ways to architect a problem. I feel very fortunate to do this as not only is it a lot of fun, but I also get to see people progress from absolute beginner to solid developer, and they are always so happy to reach the milestone.

During this process, I also get to see how different people approach learning React. React was not initially part of the curriculum at Lighthouse, but after it has exploded in popularity, it made sense to include it. An added bonus is that it is also relatively easy to learn, compared to say, Angular v1 😰.

Still though, its patterns are a bit hard to digest in the beginning, and there is so much thrown at you when you first pick it up: "I have to use `state`? How is it different than `props`? What is a container/smart/higher-order/connected component? What is a functional/stateless component? What the hell is Redux?! Is Webpack magic? I think I might go back to jQuery.." (please don't 😬)

From watching lots of students struggle and then start to love React, here are, in my opinion, the five most common mistakes when learning React.

## 1. Trying to do too much, too fast

There is nothing wrong with taking a full week (or more) to get comfortable with a new language. This isn't really talked about in the React community. It's sort of expected that you somehow know about Webpack/Babel/ES6/one-way binding, etc. This can get overwhelming really quickly, and you might lose interest in learning React because of the mess that the JavaScript landscape is currently in.

This is especially the case if your first project includes some sort of state management library like Redux, Flux or Vue.

The best way to remedy this is to simply take time and build out your knowledge in small steps. Learn about props, children and component state before worrying about the difference between functional and smart components. Don't worry about one-way data binding until you have a solid understanding of just passing down props from parent to children. Stick to the basics for the fist few days at least.

## 2. Going straight to Redux/Vue/Flux

State management libraries go hand-in-hand with React because for any moderately complex app, it becomes very difficult to manage state through passing props. However, they deserve their own dedicated time to learn rather than trying to mix it in with React learning.

These days there are more Redux/React tutorials on the web than cat pictures 😺. It's highly advised you use them to build out some very simple apps first, like the classic to-do list. Once you successfully implemented one-way data binding, it'll all make sense and scale to larger apps easily.

## 3. Too much re-rendering

This is mainly a consequence of not adhering to design patterns. I have seen many times were props are set on a parent component, such as `<MainPage />` and then passed all the way down to components such as `<UserListContainer />`, `<UserList />`, `<User />`, `<UserAvatar />`, and such. In this case it is good that such a page is broken down into very specific components, but the problem is if anything changes in any user, it will cause potentially dozens of components to re-render at the same time, and most of them unncessarily.

Ideally, props should only be passed 2 levels deep, from parent -> child. This causes less re-rendering, and thinking in this way leaves your app much more decoupled and easier to scale. At most in my opinion, props should be passed 3 levels deep. After that it becomes a pain to manage/debug code.

## 4. Using large components

In the last point, the example used a component `<UserListContainer />` that rendered several other components. This could have also been accomplished by simply putting in all the logic for user data inside the `render` method in the top-level component `<UserListContainer />`. However, the main philosophy behind React is to break down a complex page into small, re-usable `Components`. If you just take a full page and put it all entirely into one component, it defeats the purpose of React.

## 5. Forgetting to bind `this`

A very common mistake that most React beginners will make is forgetting to bind `this` in functions where the need to access props or state. Probably the first time it happens is during a simple onClick callback:

{% highlight javascript %}
class UserList extends React.Component {
  handleClick(evt) {
    this.setState({ value: evt.target.value })
  }
}
{% endhighlight %}

We get an error:

{% highlight shell %}
TypeError: Cannot read property 'setState' of null
{% endhighlight %}

This happens because `this` is not defined, so we need to bind it to the scope of the class, either through using the new ES6 fat arrow syntax:

{% highlight javascript %}
class UserList extends React.Component {
  handleClick = (evt) => {
    this.setState({ value: evt.target.value })
  }
}
{% endhighlight %}

Or we can bind it in the constructor:

{% highlight javascript %}
class UserList extends React.Component {
  constructor(props) {
    super(props);
    this.handleClick = this.handleClick.bind(this);
  }

  handleClick(evt) {
    this.setState({ value: evt.target.value })
  }
}
{% endhighlight %}

## 6. Extending Hell

I call this mistake Extending Hell, because it occurs when we simply abuse the new "Class" syntax in JavaScript, and we run into the [evils of Extend](http://www.javaworld.com/article/2073649/core-java/why-extends-is-evil.html). Honestly this is not just a beginner "mistake", or even a mistake at all. It's something that is almost unavoidable when using OOP design for very complex projects.

I am working on a project at work right now that suffers from this problem, and oh man do we have a Fragile Base Class 😞. I am mainly typing this to warn React beginners: in React, you probably NEVER need to extend off anything other than `React.Component`. Believe me, `duplication is far cheaper than the wrong abstraction`.


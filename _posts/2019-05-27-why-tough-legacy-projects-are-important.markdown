---
layout: post
title: Why messy legacy projects can be a blessing in disguise
date: 2019-05-27 00:00
comments: true
external-url:
category: 'Misc'
---

If you google `legacy code`, you'll get a lot of articles explaining how to handle it and get through it. It's not an exaggeration to say that working with tough legacy code can be draining. It takes weeks to build small features. It takes days to understand what a few lines of code were meant to do. It takes days to realize that you created a bug that wasn't caught because there weren't enough tests.

However, I've come to realize that legacy code is a blessing in disguise. I also noticed how you approach and your attitude towards working with the code can be a major contributor to your success. Here's what working on legacy code has taught me:

## It will make you a better developer

When people mention legacy code, most people's thoughts are negative. But I think it's important to ask why. Legacy code usually has the reputation of being hard to change, difficult to understand and most people are scared to touch it. But what if inherit a "perfect" codebase? 100% test coverage, fully automated deploys, end-to-end testing, easy to setup locally, etc.. I don't think people would complain then.

The point is, looking at the mistakes made in the legacy codebase can be an extremely valuable lesson. Maybe the code wasn't modular enough, so it made it very hard to understand. Maybe documentation was lacking, so it's very tricky to get it working on local. Maybe variables and methods weren't properly named, so it's impossible to understand what the code is doing. The pain you go through facing these problems will teach you how important small things like variable naming and namespacing can be to make code readable to others.

For example, at my workplace there are THREE projects, separated by domain, that all use the same database. This has caused so much headache for the engineering team, that it will clearly never be allowed to happen again within our team. But we had to go through the pain that comes with such a mistake before realizing it. However, it got us to market quickly and allowed the company to continue growing, so it's not all negative.

## It will teach you patience

The average person probably thinks programming involves writing 100s of lines of code not stop for hours every day. The reality is, most programmers spend a lot more time thinking about design vs writing lines of code. And there is good reason behind it. The first time you program something, it will likely not be great, and you'll find flaws in it quickly. It takes time to think of all the edge cases, modularize your code, organize it, test it, ensure it's readable, document it, etc. This is to ensure it is readable by others, because reading code is much, MUCH harder than writing code.

However, often a feature is just complicated, full-stop. In these cases, there isn't much that can be done to make it easy to understand without the developer having to spend some time on it. This is where patience comes in. Things like debuggers can be a life saver, allowing you to step through code and inspect things.

## It will teach you empathy

I think it's important to never assume bad intentions about other developers. I've come to realize that nobody WANTS to write bad code. There are many reasons why it can happen though. Maybe the developer didn't have much experience with the language and was forced to use it, with no supervision. Maybe there were extremely tight deadlines and the code was commited on very little sleep. Maybe best practices weren't known when it was written or were hard to access.

A brand new rails project doesn't start out bad. Probably within the first year it's super easy to understand for an experienced developer. It's also probably really fast. But as time goes on, deadlines might make the developers choose shortcuts. Maybe a couple of the core team leave and then some new developers need to take over. Who knows, but there are so many things that can happen in software development that cause a project to become "legacy", and it's not usually the fault of one person.

## It will make you realize it's all ups and downs

I don't think anything in life is all happy and positive. If it was, then it would not be positive by definition; it will be your normal.

Being pushed and challenged by working on legacy codebases can sometimes be a down in software development. But I know first hand that performing tough refactors on it can be some of the most rewarding work. Solving massive performance issues can make you a hero to your customers.

But it will also make you really cherish the time you get to spend on greenfield projects, or projects where a lot of care was put in by the developers to ensure high readability and maintainability. Or even working on small side projects.

## The code you write today is legacy code

Lastly, all the code I'm writing today will be someone else's "legacy" codebase in a few years. So to me, it's really important that I learn from the mistakes I've seen working with legacy code. This will hopefully reduce future developers working with my code from doing `git blame` too much 😝

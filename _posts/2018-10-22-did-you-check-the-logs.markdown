---
layout: post
title: Did you check the logs?
date: 2018-10-22 00:00
comments: true
external-url:
category: 'Misc'
---

Over the past few years working as a developer, I've realized it is impossible to ship anything that is completey error and bug free. This is why there are great *tools* out there to help us quickly identify and fix said errors. Lots of solutions out there will alert you when for example a 500 error occurs in production, or let's say a Kubernetes pod crashes.

Several times a week, someone will come up to me and say something like:

* "Deploying to production failed"
* "When I submit x form, I get a 422 error"
* "A customer called in and complained they are unable to do x"
* "I can't run the latest branch on my local"
* "Loading the index page is really slow"

My first response, every single time, is `Did you check the logs?`.

9/10 times, the answer is `No.`.

9/9 of those times, my face is 😐.

There's a limited amount of willpower and energy someone can put into *productive* work a day. Working for 12 hours doesn't mean you are getting more done than working 6 hours. This is why lots of top companies care more about **output** than hours spent at a desk. Being interrupted multiple times in the day makes it impossible to be productive and achieve high output.

This is why, when most things break in software, before going to a developer for answers the first step should always be `Check The Logs™️`. The people who generously write OSS like Ruby on Rails and Kubernetes really put a great emphasis into very detailed logging. On Kubernetes, a simple `kubectl logs <POD_NAME>` with a `grep` for the endpoint or timestamp of the error, has a very high chance of telling you exactly what went wrong. On your local machine, if you are let's say running a Docker container, there is the useful `docker logs` command. A classic Puma or Nginx server running locally can be searched easily as well.

Usually it is very clear when something like a migration blows up, a secret for Kubernetes is missing, or there is an n+1 or very large, slow query making the response take forever.

By checking the logs, this empowers people to:

* Debug issues faster
* Debug issues with less resources
* Learn more about some of the internals of a codebase, especially if the person is not a developer
* Keep MTTR low

While it is not the be-all and end-all solution to anything that goes wrong in software development, it is usually the best starting point to `Check The Logs™️`.
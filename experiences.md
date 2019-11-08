---
layout: page
title: Experiences
subtitle: I've gotten to work on some neat things
---

## Project: FeatureFlag

### An Elixir macro library for altering function behaviors at runtime

[GitHub Repo](https://github.com/MainShayne233/feature_flag)

This came out of solving a problem at work:

- We had some old functionality we wanted to replace with new, better code.
- We wanted to be able to switch back to the old behavior if thinks went sideways.
- We didn't want switching back to require an entire new build and deploy.

`feature_flag` solved this by allowing us to dictate the behavior of a function using a configuration value. Elixir configuration values can be mutated at runtie, so if we needed to switch back to the old behavior, it was as simple as connecting to the app and executing some code to modify that configuration value. Also, `feature_flag` allowed all of this while still have you define your function mostly like a normal function, just with multiple behaviors.


## System Building: Ship

### A multi-service Elixir system for accurate shipping cost estimations

"...FedEx called. They want us to stop DDoS'ing them."

I was working on a data anlysis project that called for us to be able to estimate the full cost of fulfilling thousands of arbitrary orders, including shipping costs. The amount of shipments we needed to simulate meant that your generic shipping cost API wasn't going to cut it; we needed something faster, and that didn't depend on network calls.

I designed and lead the development of a multi-service system that'd accomplish this goal. Part of this system was an Elixir app that regularly polled [FedEx's Rate API](https://www.fedex.com/en-us/developer.html) for changes to regularly-changing values that would affect shipping cost (fuel surcharge, volume discounts, etc) and persisted these values to be used in running later calculations.

The Elixir app was pretty simple. It mostly consisted of a single [Supervisor](https://hexdocs.pm/elixir/Supervisor.html) that managed a handful of [GenServers](https://hexdocs.pm/elixir/GenServer.html) that each periodically fetched some data from the FedEx API, parsed out what we needed, and persisted that juicy data. We needed to make quite a few requests, enough to call for sending concurrent requests, but Elixir made this almost trivial to implement, and also lightweight enough to run happily on just a [Heroku hobby dyno](https://devcenter.heroku.com/articles/dyno-types).

Starting out, the service was maybe sending a couple thousand requests to the FedEx API every few hours or so, and their API seemed to keep up fine. However, as our data anlysis needs grew, so did the amount of requests we were making. While our small Elixir app running on a single core and 512MB RAM could easily send almost a million requests within an hour or two, FedEx wasn't too excited about this. They called us and asked that we please stop because their system was suffering from the traffic. We since spread out the requests being made to give their API server time to breath.

I don't necessarily think attacking other peoples' systems is something to be proud of, but I was astounded at the capabilities of this small Elixir app running on its humble little server.

tl;dir: built a system that polled FedEx's API for data, ended up accidentally DDoS'ing FedEx with a tiny Elixir app running on a Heroku hobby dyno.


## Contribution: Gleam

### A statically typed programming language for the BEAM!

[GitHub Repo](https://github.com/gleam-lang/gleam)

This is a project I'm really excited about! I'm a static typing fanatic, and also enjoy the Erlang VM and ecosystem, so it very much aligns with my interests. I've made a few contributions to the stdlib and docs, and plan on doing a lot more.


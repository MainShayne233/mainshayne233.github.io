---
layout: page
title: Accomplishments
subtitle: 
---

Here are some neat things I've gotten to do, or got to be a part of:

### `feature_flag`: An Elixir macro library for altering function behaviors at runtime

[GitHub Repo](https://github.com/MainShayne233/feature_flag)

This came out of solving a problem at work:

- We had some old functionality we wanted to replace with new, better code.
- We wanted to be able to switch back to the old behavior if thinks went sideways.
- We didn't want switching back to require an entire new build and deploy.

`feature_flag` solved this by allowing us to dictate the behavior of a function using a configuration value. Elixir configuration values can be mutated at runtie, so if we needed to switch back to the old behavior, it was as simple as connecting to the app and executing some code to modify that configuration value. Also, `feature_flag` allowed all of this while still have you define your function mostly like a normal function, just with multiple behaviors.

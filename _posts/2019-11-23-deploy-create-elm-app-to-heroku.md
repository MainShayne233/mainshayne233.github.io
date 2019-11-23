---
layout: post
title: Deploy a create-elm-app to Heroku
subtitle: With a little help from PHP 😬
---

TL;DR: You can deploy your `create-elm-app` to Heroku as a static site with a small PHP shim and a neat git trick.

## The Situation

[create-elm-app](https://github.com/halfzebra/create-elm-app) is a really cool tool that will bootstrap a fully functioning [Elm](https://elm-lang.org/) app for you with a single command. Along with setting up the project, the tool also provides some handy-dandy commands for running a server, building a production version of the app, etc. Check out the [project's page](https://github.com/halfzebra/create-elm-app) for more details.

I often reach for this tool when spiking out a standalone Elm app, and often find myself wanting to throw my app up on a server in order to share it. For these purposes, I typically lean on [Heroku](http://heroku.com/) and their [free hobby tier](https://www.heroku.com/free).

I recently had a `create-elm-app` I wanted to deploy to Heroku, but couldn't find much regarding an easy way to do so. Most ecosystems have a decent [Heroku Buildpack](https://devcenter.heroku.com/articles/buildpacks), and though some do exist for Elm, I was running into issues regarding new Elm versions, etc.

Since my app was purely a client-side application (no backend), I thought maybe I'd just try and deploy it as a static site. Luckily, there's a pretty simple method for deploying static sites to Heroku that involves a teensy-tiny PHP shim. More details can be found in the [blog post](https://blog.teamtreehouse.com/deploy-static-site-heroku) where I disocvered this trick, but the basic idea is that you add a simple `index.php` file that will point to your main `index.html`, and as long as your static assets are pushed to the server, Heroku will serve your static site!

## How It's Done

Enough talk, let's deploy!

Starting from scratch, we will build a new `create-elm-app` and get it deployed to Heroku.

### Prerequisites

- [Install Elm](https://guide.elm-lang.org/install/elm.html)
- [Install create-elm-app](https://github.com/halfzebra/create-elm-app#installation)
- [Sign up for Heroku](https://heroku.com)
- [Install the Heroku CLI](https://devcenter.heroku.com/articles/heroku-cli)

(hopefully I'm not missing anything)

### Create a new app

In your directory of choice, use `create-elm-app` to, well, create an Elm app:

```sh
create-elm-app MyApp
cd MyApp
elm-app start
```

If all goes well, your browser should open to reveal your brand new Elm app!

(you may kill the server now; it's not necessary for the deployment)

### Setup app to be deployed

First things first, we're relying on `git` for our deployment, so we'll want to `git init` our new app:

```sh
# in ./MyApp
git init
git add -A
git commit -m 'Create new project using create-elm-app'
```

Now we'll use the Heroku CLI to create a new Heroku app to deploy to:

```sh
# in ./MyApp
# sign in if you haven't yet
heroku auth:login

# create new app
heroku create my-app-name
```

And now for the fun part!

I wrote a handy-dandy `bash` script that will build the app for the currently checked out branch and deploy the build to Heroku, no questions asked.

The script:

```sh
#!/usr/bin/env bash

##
# Given that you have a `heroku` remote added to your local repo, this will:
# - Get the current branch name
# - Check to see if the current branch is in a clean state
# - Force delete the deploy branch if it exists
# - Checkout a fresh deploy branch
# - Build a static version of the app
# - Create an index.php file to allow Heroku to serve the static site
# - Create a composer.json file to silence the Heroku warnings
# - Force add the build to git
# - Force push the build directory to Heroku
# - Checkout the original branch
# - Delete the deploy branch

main() {
  local current_branch=$(git rev-parse --abbrev-ref HEAD)
  if ! [ -z "$(git status --porcelain)" ]; then
    echo "The current branch (${current_branch}) isn't clean."
    echo "Commit or remove these changes and try again."
    exit 1
  fi
  git branch -D deploy >>/dev/null 2>&1
  git checkout -b deploy
  elm-app build
  echo "<?php header( 'Location: /index.html' ) ;  ?>" > build/index.php
  echo "{}" > build/composer.json
  git add --force build
  git commit -m 'Build for deploy'
  git push heroku $(git subtree split --prefix build deploy):refs/heads/master --force
  git checkout "${current_branch}"
  git branch -D deploy
}

main "$@"
```

The comments in the script mostly describe what is going on, but at a high level all we're doing is:
- Checking out a deploy branch
- Building the app
- Adding the `build` dir to `git`
- Force pushing the deploy branch's `build` to Heroku
- Cleaning up

It's not necessary to understand in order to use the script, but one of the more interesting parts of this script is the `git push` step which is utilizing `git subtree` in order to push the `build` directory to Heroku as the project's root. I don't completely understand what it's doing, but it's been working well for me :p.

Add the script to your project like so:

```sh
# in ./MyApp
mkdir bin
<user-whatever-method-to-put-script-at bin/deploy>
chmod +x ./bin/deploy
git add ./bin/deploy
git commit -m 'Add deploy script'
```

Once the deploy script is in place and made executable, deploying your app is as simple as running:

```sh
# in ./MyApp
./bin/deploy
```

Once deployed, you can open your app up in your browser via:

```sh
# in ./MyApp
heroku open
```

That's it!

## Closing Thoughts

A special thanks to Andrew Chalkley for their blog post: [How to Deploy a Static Site to Heroku](https://blog.teamtreehouse.com/deploy-static-site-heroku).

I hope someone else can find this useful! If that is the case, or you run into issues with my instructions, or have any sort of feedback, reaching out is very encouraged! :-)



---
layout: post
title: "Running a Single Page App in Docker - Part 1"
---

## Setting the Stage

Recently, I've [posted][container-post] about what the high level of what a container does, how [sessions][sessions-post], how a [JWT][jwt-post] works, and how [Google Sign-in works][google-post].  I've been exploring these topics through a simple personal finance application that I am working on.  In this post, I would like to synthesize a lot of what I've been doing into running this application in a container.

## Why a container?

Primarily, I want learn more about how containers work in practice.  If I didn't want to do that, it could make sense for me to just ship my application to a small virtual machine somewhere and run it.  One tangible benefit I can see for this application is that once I have the container running locally, I can have higher confidence that it will work on the server (assuming I set up the container engine properly in both places).

I decided to go with Docker since from my limited experience it was very user friendly whereas I wasn't sure about `rkt`.  I plan on exploring `rkt` more in the future.

## The Architecture

I wanted to explore having totally a separate front-end and back-end which communicate via JSON.  That separation sounds nice in theory, so I wanted to see how it worked out in practice.  For this application, the back-end is written in Go and the front-end is built in React starting with [`create-react-app`][react-app].

For the sake of keeping the front-end and back-end entirely separate, I want to have nginx serve the front-end content statically and proxy the API requests to the Go server.

It is worth noting that there are a number of simpler architectures that could work here.  For instance, the Go server could simply server the `build` directory created by the react app itself.  This would obviate the need for nginx and would make my deploy only require building the JS app and then running the Go app.

Additionally, having a single app do both the front-end and back-end would be simple as well.  For instance, a Flask app using Jinja templates for HTML would be very simple, and if I were optimizing for getting something out the door, could be a better choice.

However, like I said, my goal was to explore the totally separate front-end / back-end architecture.

## The Cast

I recorded my terminal session using [asciicinema](asciinema.org) below.  This session was not rehearsed, so it is not the most efficient path forward.  In the first session, I:

* Install Docker
* Get the back-end running locally
     - I needed to install go, fix up some package naming issues etc.
* Get the back-end running in a container
     - I started using `FROM golang:onbuild` in my Dockerfile which does some things automatically, but I found 2 issues.  The first was that I needed to copy a configuration file over, and it was not clear where I should copy it based on all the magic happening for me.  The second was that rebuilding the container for minor changes took too long because for some reason Docker had to start all the way over each time.  (See below about Docker container "caching").  Fortunately, the [Go blog post about Docker][go-docker] spells out what you need in a Dockerfile if you don't want to use the `golang:onbuild` method.
* Get nginx running in a container
* Get the frontend running in a container


(The arrow keys let you move around the cast more quickly as does typing 1 to navigate to 10% through, 2 to 20% through etc.)
<script type="text/javascript" src="https://asciinema.org/a/123753.js" id="asciicast-123753" async></script>

## Wrapping Up

So, to recap, after this session, I had 2 Dockerfiles: 1 for the back-end, 1 for the front-end.  My next goal was to make this into 1 container that would run both and expose a single port.  I'll cover that in a follow up post.


 [docker-install]:https://docs.docker.com/engine/installation/linux/ubuntu/
 [go-docker]:https://blog.golang.org/docker
 [react-app]:https://github.com/facebookincubator/create-react-app
 [container-post]:{% post_url 2017-04-19-containers %}
 [sessions-post]:{% post_url 2017-04-27-sessions %}
 [jwt-post]:{% post_url 2017-05-31-jwt%}
 [google-post]:{% post_url 2017-03-27-google-sign-in %}

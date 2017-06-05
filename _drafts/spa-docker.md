---
layout: post
title: "Running a Single Page App in Docker"
---


# Setting the Stage

Recently, I've [posted][container-post] about what the high level of what a container does, how [sessions][sessions-post], how a [JWT][jwt-post] works, and how [Google Sign-in works][google-post].  I've been exploring these topics through a simple personal finance application that I am working on.  In this post, I would like to synthesize a lot of what I've been doing into running this application in a container.

# The Architecture

One of the goals of my application is to explore new technologies, which means that the architecture may be a little over engineered.  If I were optimizing for getting something out the door, I would probably do a Flask app that used Jinja templates for its pages.  This architecture is simple and familiar.

Instead, I wanted to learn about React to get a better sense of how it works in practice so that I can better make a determination on my own for whether it is a good fit for future projects.  I also wanted to try having the back-end expose a simple JSON API rather than do any templating.  This is another one of those ideas that seems really nice in theory, but I wasn't sure how it would play out in practice.

# Serving

With this architecture of a Single Page App with a JSON backend, I have a couple options of how to serve it.  I used [`create-react-app`][react-app] to start out which gives a script to generate a `build` directory that contains everything needed to run the front-end.  Pointing any kind of simple HTTP server at that directory would do the trick.

One option is to have the back-end statically serve that directory.  That would have the nice feature that I would only need a single process to serve both the front-end and the back-end. One potential draw-back is that it couples the front-end and back-end together

I decided to first explore keeping the two separate.  I had read that `nginx` could be helpful here, so I wanted to try that out.

### Brief aside: What is nginx?



# Containers

In the spirit of trying new things, I also wanted to run these in containers.  I decided to go with Docker at first since from my limited experience it seemed pretty user friendly whereas I wasn't sure about `rkt`.  I plan on exploring `rkt` more in the future.


# The Cast

I recorded my terminal session using [asciicinema][asciinema.org] below.  This session was not rehearsed, so it is not the most efficient path forward.  In the first session, I:
 * Get the back-end running locally
 ** I needed to install go, fix up some package naming issues etc.
 * Get the back-end running in a container
 ** I started using `FROM golang:onbuild` in my Dockerfile which does some things automatically, but I found 2 issues.  The first was that I needed to copy a configuration file over, and it was not clear where I should copy it based on all the magic happening for me.  The second was that rebuilding the container for minor changes took too long because for some reason Docker had to start all the way over each time.  (See below about Docker container "caching").  Fortunately, the [Go blog post about Docker][go-docker] spells out what you need in a Dockerfile if you don't want to use the `golang:onbuild` method.
 * Get nginx running in a container
 * Get the frontend running in a container

 So, to recap, after this session, I had 2 Dockerfiles: 1 for the back-end, 1 for the front-end.  My next goal was to make this into 1 container that would run both and expose a single port.

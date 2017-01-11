---
layout: post
title: "Pixi Buckles - Using PixiJS with BuckleScript"
---


# Introduction

In my [last post][1], I talked about what BuckleScript is and how to get started.  I found that when writing posts, my weakest posts come after I have been spending a lot of time on a topic and then go back and try to fit a post around it.  That's what happened with that post as I had been fooling around with BuckleScript for several weeks before writing the post.  This time, I'm beginning with the end in mind by starting the post at the beginning of the project.

# Goal

My goals for this project are:
1. Learn more about BuckleScript's JS interoperation
2. Make something pretty

# Why PixiJS?

I chose [PixiJS][4] pretty arbitrarily.  I actually started down this whole BuckleScript path after playing around a little bit with [Halite.io][2].  That site is an AI programming competition.  I wanted to be able to simulate different scenarios in that game, and I found they wrote their visualization with [PixiJS][4].  I didn't end up doing anything with [halite.io][2] partially because my friend introduced me to [Screeps][3] which I got hooked on.  Still, I liked the visualization they used, and I've been wanting to create some visualizations that are sort of outside the intended audience for things like [d3.js][5] which I've worked with before.

# Getting started

First, I want to draw a circle without using BuckleScript to make sure I can get that working.  What I want is a single javascript file `main.js` that calls the PixiJS drawing API's.  I wanted to get this working without any other distractions like "webpack" and npm scripts, so I made a simple `index.html` that included `main.js` in the body.

```
// Set up renderer and append to body. This ends up as a canvas.
var renderer = PIXI.autoDetectRenderer(800, 600, { antialias: true });
document.body.appendChild(renderer.view);

// create the root of the scene graph
var stage = new PIXI.Container();
var graphics = new PIXI.Graphics();
stage.addChild(graphics);

// Create a function to draw a circle with our
// graphics object.
function drawCircle(x, y, r) {
    graphics.lineStyle(0);
    graphics.beginFill(0xFFFF0B, 0.5);
    graphics.drawCircle(x, y, r);
    graphics.endFill();
}

// Initialize variables.  We're drawing a circle
// with radius littleR and it follows the path
// centered at (centerX, centerY) with radius bigR
var t = 0;
var centerX = 300;
var centerY = 300;
var bigR = 200;
var littleR = 60;

// run the render loop
animate();

function animate() {
    var x = bigR*Math.sin(t) + centerX;
    var y = bigR*Math.cos(t) + centerY;

    graphics.clear();
    drawCircle(x, y, littleR);
    t += .05;

    renderer.render(stage);
    requestAnimationFrame( animate );
}
```



[1]:{% post_url 2017-01-02-bucklescript-1 %}
[2]:https://halite.io/
[3]:https://screeps.com/
[4]:http://www.pixijs.com/
[5]:https://d3js.org/

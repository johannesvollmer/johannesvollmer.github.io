---
title: Announcing "WebGL of Life" Online GPU Simulation and Editor
# jekyll-seo-tag
description:  "Simulating Conway's Game of Life on a website using the GPU"
#image:        "http://placehold.it/400x200" # why nothing visible?
author:       "johannesvollmer"
published: 	  false
---

Hey! In this post, I am going to explain some of the challenges 
I had to face when programming Conway's Game of Life on the GPU.

# What is this game?

If you have never heard of "Conway's Game of Life", you will learn something new today! Hurray!

The Game consists of cells, which are arranged in a grid. 
Each cell can either be alive or dead, as specified by the player. 
Then, the simulation starts. Over and over again, 
the game's rules are applied to each cell on the board, 
producing a new generation each time. 
The rules specify if a cell should survive or cease, 
based on the number of neighbours surrounding it.

For example, one rule states a new cell is born 
where exactly three neighbours are living.

![A new cell being born]({{ site.baseurl }}/img/webgl-of-life/cell-is-born.svg)

[Here's the english wikipedia article](https://en.wikipedia.org/w/index.php?title=Conway%27s_Game_of_Life&oldid=906060432),
if you want to read some more.

# The approach

The fundamental idea of this project was to simulate all cells in parallel
using the computational power of the graphics processor. This would
allow for large boards being simulated in the browser without the 
performance implications of using JavaScript.

Currently, the only way to utilize the GPU on the web is by using WebGL.
OpenGL is used to display 2D and 3D graphics using the GPU. 
The cell board is be a texture with each pixel representing a cell. 
The board is rendered in WebGL for realtime performance.
Of course, the simulation of each evolution is also computed in WebGL.
For this purpose, I wrote a shader which 
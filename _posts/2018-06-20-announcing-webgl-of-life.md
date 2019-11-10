---
title:          Implementing Conway's "WebGL of Life" on the GPU in the web
description:    Simulating Conway's Game of Life on a website using the GPU
image:          "{{ site.baseurl }}/img/webgl-of-life/thumbnail.jpg"
logo:           "{{ site.baseurl }}/img/logo.svg"
author:         johannesvollmer
published: 	    true
---

Hey! In this post, I am going to explain some of the challenges 
I had to face when programming 
[Conway's Game of Life](https://johannesvollmer.github.io/webgl-of-life/)
on the GPU.

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
The current generation of cells is stored in a texture, with each pixel representing a cell. This board is rendered in WebGL for realtime performance. Of course, the simulation of each evolution is also computed in WebGL.

For this purpose, I wrote a shader which transforms the current board pixel per pixel. This shader is rendered to a second texture, which afterwards contains the next generation. These textures are internally swapped after each generation. 

# Fattening the Cells

In the final app, each pixels contains more information than just a simple `on` or `off`, though. In RGB Textures, we have more space available to us. To simplify visualization and computation, the `green` channel of a pixel in the current board always caches the count its neighbours. Also, the `blue` channel stores the state of that cell in the previous generation. This enables interpolating between those two states, achieving a simple opacity transition, resulting in a smoother look. 

# Painting the Texture

Drawing cells on the board is done in JavaScript. This allows for complex board modifications in the future. Displaying the brush contents is achieved using SVG.


# WebGL of Life

Here's the finished app: [WebGL of Life](https://johannesvollmer.github.io/webgl-of-life/).
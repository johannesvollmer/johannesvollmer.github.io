---
title:        "(Ab)using WebGL to simulate Conway's Game of Life"
# jekyll-seo-tag
description:  "Utilizing the parallelization of the GPU for simulation of cellular automatons"
#image:        "http://placehold.it/400x200" # why nothing visible?
author:       "johannesvollmer"
---
*Utilizing the parallelization of the GPU for simulation of cellular automatons*
<!-- TODO: in the website template, add the frontmatter description as sub title in the post -->


---
# THIS ARTICLE IS CURRENTLY BEING WRITTEN AND NOT FINISHED YET
---

## Good afternoon! 
I hope it's afternoon for you, because that's when people eat cake and drink coffee. 
Anyways. Nice to have you here. Also, let's talk about OpenGL, and how I used it to compute Conway's Game of Life in my browser :)

__Check out [the finished Game of Life on my website](//johannesvollmer.github.io/webgl-of-life)__,

or read the source code any time by visiting the [project on Github](https://github.com/johannesvollmer/webgl-of-life).


## But what __is__ that strange Game?

If you have never heard of "Conway's Game of Life", you will learn something new today! Hurray!

The Game consists of cells. Each cell can either be alive or dead, as specified by the player. Then, the simulation starts. Over and over again, the game's rules are applied to each cell on the board, producing a new generation each time. The rules specify if a cell should survive or cease, based on the number of neighbours surrounding it.

Of course we'll cover the details later, when we talk about the specific implementation :)


## WebGL
As this is a rather in-depth post, it is helpful if you already have some basic understanding about OpenGL. 

<!--For decades, a library called 'OpenGL' has been used to speed up graphics on computers. The high speed is achieved by using specialized hardware, the GPU, which can operate in a highly parallel manner. OpenGL gives any native application acces to the GPU, and just recently, it reached a state where most web devices should support WebGL. -->

To summarize what WebGL actually is: WebGL makes it possible to use the GPU in a modern browser. It does so by providing a javascript interface which is based on the OpenGL ES 2.0 specification. 

Here's what Wikipedia thinks of 
[OpenGL](https://en.wikipedia.org/w/index.php?title=OpenGL&oldid=844606672) and
[WebGL](https://en.wikipedia.org/w/index.php?title=WebGL&oldid=846821661).


## The Idea
I had a plan and I had a deadline: One month time.

My idea was to use OpenGL shaders and textures. I could render the new generation to an empty texture, using the old generation as an input. As the data would be loaded into the GPU anyways, we could simply display the cells also using WebGL.

Using render-to-texture, the GPU would compute the new generation for many (if not all) cells in parallel, potentially being faster than simple javascript. 

## Let's dive in!
Sometimes it can be hard to begin coding a project, because you simply don't know _where_ to start. But that's okay, because eventually, you'll find some little thing that you'd want to try out, which then could later maybe extended to a real product.

__The first thing I wanted to try was: have some cells and display them.__

## Step I  --  Storing Cells in Memory
Most people use OpenGL for threedimensional rendering, but at its core, OpenGL is actually twodimensional. That's great, as the game we want to code is twodimensional, too. Hurray :)

As textures are two-dimensional grid of values, they would be a good fit to store all the cells.
Each pixel should contain the data for a single cell. 

That way of representing cells as a finite texture limits my Game, it does not allow an infinite simulation space. But I was okay with that. 

Most of the time, textures in OpenGL contain three values per pixel: red, green, and blue. But my texture does not need three values, only a single one per cell: either 1 or 0, representing a living or dead cell respectively. Thus I decided to use a Grayscale texture to store the state of the cells.

__Let's do that!__

First, we'll need some boilerplate code to initialize OpenGL (Yay, finally some code!):
<!-- TODO: configure the code formatter to not break lines -->


``` javascript
// try loading a webgl context for our canvas into window.gl
window.gl = (() => {
	for (let contextName of ['experimental-webgl', 'webgl']){
		try {
			const gl = canvas.getContext(contextName, { antialias: true })
			if (gl) return gl
		}
		catch (ignored) {}
	}
})()

if(!window.gl)
	alert("WebGL could not be loaded. Please use a browser which supports WebGL.")

```
See it in context on github: [index.js](https://github.com/johannesvollmer/webgl-of-life/blob/master/js/index.js)


Because we want to fill the whole screen with our drawings, we will keep the resolution of the context the same as the window size (We need this later to correctly determine the mouse position):
``` javascript
// update the size of the webgl canvas to be full-width and full-height
function onCanvasResize(){
	canvas.width = window.innerWidth
	canvas.height = window.innerHeight
	// to do: re-draw all contents after resizing
}

// register our function, and
window.addEventListener('resize', onCanvasResize, {passive: true})

// trigger once to initialize the canvas' size
onCanvasResize() 
```
Github: [index.js](https://github.com/johannesvollmer/webgl-of-life/blob/master/js/index.js)

The event listener being `passive` prevents blocking the browsers rendering. Read [this article on medium.com](https://medium.com/@devlucky/about-passive-event-listeners-224ff620e68c) if you want to learn more.

The problem with non-fullsize canvases is, that it's (sadly) still not possible to transform coordinates between dom elements. To correctly determine the local mouse position of an element, you'd have to manually go through all parents of the element, and add all their positions, margins, paddings, translations, rotations, and scalings. There is a [non-standard webkit-only function](https://developer.mozilla.org/en-US/docs/Web/API/Window/convertPointFromPageToNode) that would enable us to simply call one single function to convert between element coordinate spaces, but it's not implemented in most browsers.

Anyways, we have our WebGL canvas. Now we need to create some cells, which we can load into a texture later. 
``` javascript
// the grid of cells, 255 is alive, 0 is dead
let width = 4
let height = 4
let cell_data = new Uint8Array([
	0,   0, 255, 0,
	0, 255,   0, 0,
	0, 255, 255, 0,
	0,   0,   0, 0,
])
```


Now we need a texture, containing the actual cell data. Because the WebGL context is now contained in `window.gl`, we can simply call `gl.createTexture()` to obtain a handle to the texture which was created on the GPU. But to fill in our cell-data, we need to bind the texture first.
``` javascript
const cell_texture_id = gl.createTexture()
gl.bindTexture(gl.TEXTURE_2D, cell_texture_id)

// transfer only one byte at once, because otherwise our texture dimensions would have to be perfectly divisible by some arbitrary number like 4 or 8 
gl.pixelStorei(gl.UNPACK_ALIGNMENT, 1)
gl.pixelStorei(gl.PACK_ALIGNMENT, 1)

gl.texImage2D(
	gl.TEXTURE_2D,
	0, // mip map level
	gl.LUMINANCE, // format, LUMINANCE = one value per pixel
	width,
	height,
	0, // border size in px
	format,
	gl.UNSIGNED_BYTE, // data type of each r,g,b, or luminance, UNSIGNED_BYTE = 8bit, values are 0-255
	cell_data 	// actual content (uint8array, or image element, or canvas element, or imagebitmap)
)
```
Because binding and unbinding textures is such a common task, I wrote a simple abstraction function called `Texture2D`. You can look it up on github: [Texture.js](https://github.com/johannesvollmer/webgl-of-life/blob/master/js/gl/Texture.js)


Great! Now we have the data in our texture. 


# Also, look at
- [A video of __Daniel Shiffmann__](https://youtu.be/FWSR_7kZuYg) performing an alternative implementation of Conway's game, ~38 minutes
- [The website __Shadertoy__](https://www.shadertoy.com/), which hosts many really, really crazy and creative usages of WebGL

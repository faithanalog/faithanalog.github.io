---
layout: post
title: "Prelude of the Chambered in Dart"
---
I was watching `[REDACTED]` write what was essentially a re-implementation of Doom in
Dart recently, and it inspired me to port Prelude of the Chambered (PotC) to
Dart.

Prelude of the Chambered is a game made by `[REDACTED]` for the [Ludum
Dare](http://www.ludumdare.com/compo/) competition awhile ago. It can run in
the browser, but the problem is it's a java applet. These days, browsers throw
warning after warning at anyone loading an applet, and that's assuming they
even have the java plugin installed. I figured porting it to Dart would solve
that problem, and be pretty simple as well, because Dart code is very similar
in syntax to Java.

I've posted the code to [Github](https://github.com/unknownloner/potc-dart), or
you can try it out [Here](http://potc.unknownloner.com).  Since a lot of people
seem to get somewhat confused by the controls, know that space bar is used to
select menu options and attack. WASD/Arrow keys are for movement. A and D
strafe, left and right arrows turn.


## Game Logic

Most of the game's logic could be straight up copy-pasted with a few minor
corrections (mostly involving Dart's requirement of a '.0' after whole numbers
to indicate they're doubles). I opted not to do one dart file per class,
instead grouping all entities in a 'entity.dart', all blocks in a 'block.dart',
and so on. The only actual logic that had to be rewritten was the level loading
code.

Prelude of the Chambered stores all its levels as png files. The RGB value of
each pixel is used to determine the tiles of the level, while the alpha value
is used to link triggers. For example, a button with an alpha of 1 would open
the door with an alpha of 1, or a pressure plate with an alpha of 254 would
open the door with the same alpha. This meant that to load levels from the
images, all color channels must be preserved.

The simplest way to load image data involves drawing it to a canvas and getting
the canvas colors. I found, however, that some browsers would return different
colors from the original when reading the canvas. I suspect the canvas might be
using pre-multiplied alpha when drawing, although I'm not entirely sure. I also
tried drawing the image with webgl to a framebuffer, and then reading the
pixels back in, but that also sometimes failed probably for the same reasons.
The solution was to use this [dart image
library](https://pub.dartlang.org/packages/image) along with HttpRequests to
decode the images, avoiding the browser's image decoder entirely.

## Rendering

Porting the rendering was a bit more complex. PotC's original rendering engine
was custom written during the competition, and doesn't use an API like OpenGL
or DirectX. With WebGL available in the browser, it'd be a bit silly to do the
original per pixel effects instead of rewriting them to use WebGL. Most of the
world rendering was actually fairly simple. PotC has a method for drawing
walls, and a method for drawing sprites, so all that needed to be done was to
re-implement those methods. In WebGL it's fairly trivial since it's just
drawing two triangles. Floors were also simple, the code is almost identical to
the wall rendering code. Truthfully, most of the interesting rendering code
involves the shaders.

For the 3d world, the first thing that needs to be done is to pass the depth of
each vertex to the fragment shader. The value needed is the Z coordinate of the
position vector after being multiplied by the projection matrix. This gives us
the distance (in blocks) away from the camera of the vertex. In the 3d fragment
shader, this depth is used to implement fog. The original PotC also has some
weird post-processing effect which uses the pixel position to create a sort-of
noise over the 3d viewport. Fortunately, that can also be implemented in the 3d
fragment shader, so it doesn't need a separate pass. Finally, PotC's textures
use magenta `(#FFFF00)` to represent transparency, so the shaders discard any
fragment which reads magenta from the source texture.

The other interesting shader is the shader used to draw the 'hurt' texture. The
hurt texture is displayed any time the player is hurt in the game. In the
original, the effect is created in real time by constructing a Random object
with a preset seed, and comparing random values for each pixel with a function
of time so that less and less of the hurt effect shows up each frame.
Re-implementing that exactly wouldn't be very efficient in WebGL, so I took a
different approach. The texture is generated once when the game launches,
saving the random value used in the original PotC as the alpha channel of the
texture. Then, time can be passed as a uniform to a fragment shader which has
the comparison code in it.

## Audio

Audio was the quickest part of the port. The WebAudio API works well, and all
the sound clips are .wav files so they load pretty much everywhere. It was as
simple as loading the audio clips and playing them.

## What I've learned

Creating this port was an interesting process. I've found that a lot of java
code is actually pretty easy to port to Dart, even without taking the time to
understand what the original code does. I think that a lot of games could also
be ported without too much trouble, as long as they don't depend too much on
third party libraries. I have a better understand now, too, of the versatility
of GLSL shaders. I think this is yet more proof that the web APIs are ready for
larger and more complex games. I'm interested to work on some of my own games
in Dart, and see where it takes me.

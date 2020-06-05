## OpenGL Particle System Demo

I hope to discuss in this repository different ways particle engines are usually made, and I intend on implementing one using a geometry shader.

For more information, see the official [OpenTK Getting Started tutorials](https://opentk.net/learn).

#### Intro: What are the requirements?
Before we begin building anything, we need to know the details. We can all imagine fancy particles flying around, but let's start by making a list of the features we will try to implement.

First let's cover the basics. Each particle should have the following attributes:
 - position
 - velocity
 - texture
 - tint

More advanced features of our particle engine would be include:
 - rotation
 - gravity (in addition to velocity)
 - pulsating color (or alpha)
 - changing color over time (including fade in/out)

These are the different methods of achieving this:
1. [Immediate Rendering (yuck!)](#1-immediate-rendering)
2. [Buffer Streaming](#2-buffer-streaming)
3. [Geometry Shader](#3-geometry-shader)
4. [Time-Based Geometry Shader](#4-time-based-geometry-shader)

For more information, see the official [OpenTK Getting Started tutorials](https://opentk.net/learn).

#### 1. Immediate Rendering
**For reasons involving performance this method is deprecated and should not be used in real production applications.** Nonetheless, this is the simplest OpenGL solution, since it requires no advanced or modern techniques. I'm mentioning this here simply to give an understanding of how the particle engine will be structured.

It's quite straightforward. Think of each particle as a small texture on the screen that is moving in some direction. We want to have hundreds, thousands or more of these particles.

If we think of this in terms of immediate rendering it's not hard to arrive at the conclusion that we're going to have a list or array holding data for each particle, and each particle will be rendered by looping through that collection.

If we were using immediate mode that's exactly how we'd do it. Each item in the array would be a struct or class containing fields indicating the current position, how much to offset the position per frame, which texture to use, what color to tint it, etc. We will ultimately do something similar to this, but we want to push the load to the GPU since we want an efficient design-- and immediate mode is too slow to achieve that.

#### 2. Buffer Streaming
We can start by making a vertex buffer on the GPU. Then, we can stream the current state of all active particles on the screen to the buffer in each frame. This will help reduce the number of OpenGL API calls. We can push the state of all particles in one [CopyBufferSubData](https://www.khronos.org/registry/OpenGL-Refpages/gl4/html/glCopyBufferSubData.xhtml) API call.

Since this method/solution does not use a geometry shader, we would use the regular TRIANGLES primitive. This means each particle needs to have the position and texcoord data for the four corners (or 6 if not using a index rendering).

Then by using a vertex and fragment shader and the DrawArrays API function we can render our particles onto the screen. The fragment shader would handle the texture sampling and tinting.

#### 3. Geometry Shader
We can improve on the previous method by using a geometry shader. For instance, if we know the position and size of a particle, then it's possible to compute the four vertices at render time.

The state of all particles can still be streamed to the the vertex buffer on the GPU, but computing the vertices ,means we can stream less data per frame.

#### 4. Time-Based Geometry Shader
This method is what we're going to implement. In the previous method we would have streamed the particle data to the buffer per frame (since it is modified per frame) and we would compute some additional vertex data on the GPU using the geometry shader.

This method further improves on that by not requiring the particle data to be streamed per frame. Rather, a uniform indicating the current time (as a float) is passed to the shader, which can then be used to compute the current state of each particle. This computation must occur per frame, in order to know the latest state of each particle and where/how to render it. The reasoning this method is based on the assumption that the GPU is be faster at computing the state of the particles than the CPU. This assumption is logical, given that the GPU specializes in what we're trying to do.

Implementation and additional details coming soon.

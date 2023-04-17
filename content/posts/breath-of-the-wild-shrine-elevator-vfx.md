---
title: "Breath of the Wild Shrine Elevator VFX with VFX Graph and Shader Graph"
date: 2020-11-27T16:42:04+01:00
draft: false
toc: false
description: "This is a breakdown of my attempt at reproducing the shrine elevator VFX from The Legend of Zelda: Breath of the Wild. This was my first project using VFX graph."
thumbnail: "/img/posts/breath-of-the-wild-shrine-elevator-vfx/thumbnail.png"
tags:
  - unity
  - shader
  - shadergraph
  - vfx
  - vfxgraph
pager: false
---

I’ve been tempted to use VFX Graph for a long time now, and when I decided of my next VFX to make, I decided to only use VFX Graph instead of particle systems.

Original:

![](/img/posts/breath-of-the-wild-shrine-elevator-vfx/botw.png)

Here is the result!

{{< youtube e6HNK9ufojw >}}

![](/img/posts/breath-of-the-wild-shrine-elevator-vfx/Unity_2020-11-21_23-55-07.png)


# Breakdown
---

## The Bars
The bars have two major effects, both are related to the normal of the cylinder mesh it’s on. First, the centre is slightly faded out. This makes the character inside more visible and the bars feel less like a prison cell and more like energy bars or lasers. Second, the outer edges are faded out. This avoids having too many bars overlapping each other on the edges which would result in a bulk of colour instead of clearly defined bars.

![](/img/posts/breath-of-the-wild-shrine-elevator-vfx/Unity_2020-11-27_14-32-38.png)

## The Smoke
The smoke cloud pattern was one of the trickiest effect to find how to do. Not unlike the bars, it fades out slightly in the centre for the same reason. However, because the outer edges are not faded out, it gives a subtle but very satisfying illusion of wrapping around the bars, as if the tube was bigger, even though it’s the exact same mesh and scale.

![](/img/posts/breath-of-the-wild-shrine-elevator-vfx/Unity_2020-11-27_14-33-01.png)

The pattern is the result of the subtraction of a single noise texture to itself but with a small offset in the black/white edge, using two smoothstep nodes with slightly different parameters.

![](/img/posts/breath-of-the-wild-shrine-elevator-vfx/Unity_2020-11-27_15-00-46.png)

## The Particles
The dust and “upward raindrops“ are a single VFX graph. 
![](/img/posts/breath-of-the-wild-shrine-elevator-vfx/upward_raindrops.png)

As you can see, the raindrops are actually tall quads. The trick for their animation resides in their shader graph.

The shader graph is given the scaled (0-1) age of the particle by the VFX graph. It combines that with the Y value of the UV to animate a reveal of the texture from bottom to top.

![](/img/posts/breath-of-the-wild-shrine-elevator-vfx/Unity_2020-11-27_14-38-15.png)

---
title: "The Darksaber from Star Wars in Unity"
date: 2020-10-15T18:41:39+01:00
draft: false
toc: false
description: "I attempt to reproduce the Darksaber from Star Wars in Unity. I used Shader Graph, the Universal Render Pipeline and its handy renderer features."
thumbnail: "/img/posts/the-darksaber-from-star-wars-in-unity/thumbnail.jpg"
tags:
  - unity
  - shader
  - shadergraph
  - vfx
aliases:
  - /blog/the-darksaber-from-star-wars-in-unity
---

The design of the Darksaber has always fascinated me. While lightsabers are already not physically possible, emitting a bright white light from a black source of light is even more surreal. Sounds like a good challenge in Unity!

I went for the Clone Wars and Rebels look of the Darksaber, especially because of the magnificent trail it leaves behind when moved that adds even more to that surreal look.

Here is the result!

{{<youtube kQVx14uW5Ko>}}

## The Dark Glow

The glow of the blade is composed of two key components. There is a first blade mesh that’s white and glows white too. 
And a second blade, that’s black and rendered AFTER the post processing has happened.

A first blade mesh is rendered purely white, with a high HDR intensity.

![](/img/posts/the-darksaber-from-star-wars-in-unity/Unity_2020-10-15_13-31-37.png)

The Bloom post processing effect will take care of making that white blade glow.

![](/img/posts/the-darksaber-from-star-wars-in-unity/Unity_2020-10-15_13-32-58.png)

I then use a Render Objects renderer feature to render my black blade after the post processing.

![](/img/posts/the-darksaber-from-star-wars-in-unity/Unity_2020-10-15_13-42-26.png)

## The Trail

The trail is a custom script that will record the position of the blade every frame and generate a mesh. The mesh is then applied to two different mesh renderers, one for the glow of the trail, the other for the black that’s rendered after the post processing, just like explained in section The Dark Glow.

Here is the script:

{{< gist MadStark 432be0ace133d6582edfa8cdaca2a44a SwordTrail.cs >}}

## The Blade Shader

I made the blade shader in Shader Graph. The electric pattern varies a lot throughout the different media, so I decided to make one of my own. It relies on a caustic texture and a Perlin noise texture to add some more variation. It’s very simple but I’m very pleased with the result.

Here is the graph:

![](/img/posts/the-darksaber-from-star-wars-in-unity/Unity_2020-10-15_13-56-19.png)

![](/img/posts/the-darksaber-from-star-wars-in-unity/darksaber_pattern.gif)

## The Depth Issue

One of the big challenges with this project was to make sure the black blade and trail always render on top of the white ones, from any angle.

While I don’t yet know a way to change the depth test to Always in a shader graph, the Render Objects feature conveniently offers to do that. So all objects it renders (here, any dark component) will be rendered on top of everything.

While this is almost what I want, it now renders on top of everything, including my character.

![](/img/posts/the-darksaber-from-star-wars-in-unity/Unity_2020-10-15_14-22-30.png)

So I made a custom depth test, that will sample the depth texture, and check if the distance is small enough to justify drawing on top of it.

I plugged that into the alpha property and the Alpha Clip Threshold takes care of discarding any pixel that should not be drawn.

![](/img/posts/the-darksaber-from-star-wars-in-unity/Unity_2020-10-15_14-29-17.png)

---
title: "Making an ECS Pathfinding in Unity"
date: 2020-10-28T17:57:45+01:00
draft: false
toc: false
description: "For this first dev log, I talk about the two approaches I tried in order to achieve a pathfinding system in Unity ECS."
thumbnail: "/img/posts/ecs-pathfinding/thumbnail.jpg"
tags:
  - unity
  - ecs
  - dots
  - csharp
---

For this first dev log of making a game in ECS using Unity DOTS, I will talk about my experiments with pathfinding. The goal is simple, get an entity, here, the enemies, to a target location, here, the player.

## Attempt #1: Whiskers Collision Avoidance

My first attempt at pathfinding was to move the entity towards the target, while each step, scan the surroundings and move away from objects. I named it the “Whiskers Pathfinding“.

![](/img/posts/ecs-pathfinding/Unity_2020-10-28_17-55-32.png)

While I’m really proud of that solution, and despite the fact that it looks awesome, it turned out to be rather unreliable, and very limited in features. I might combine this system with another pathfinding system later though!

## Attempt #2: Unity NavMesh

Unity provides a fair few tools to generate a navmesh. So I decided to start over and use those instead.

![](/img/posts/ecs-pathfinding/Unity_2020-10-28_17-23-14.jpg)

{{<youtube i6ZSjcMHZ3M>}}

The result looks much more advanced, and I’m very happy with this approach so far. The key component of this was to use the struct NavMeshQuery, provided by Unity, to request a path between the agent and the target.

While this is a good start, it could definitely use more polish. I will likely add agent avoidance and support for connections between navmeshes, sometime in the future.
---
title: "Wayfinder in Unity: Pathfinding on Custom Maps"
date: 2020-08-29T19:30:28+01:00
draft: false
toc: false
description: "An Editor tool along with a path-finding algorithm to make some sort of Google Maps for custom floor-plans within Unity."
thumbnail: "/img/posts/wayfinder-in-unity-pathfinding-on-custom-maps/wayfinder_thumbnail.jpg"
tags:
  - unity
  - unityeditor
  - csharp
  - tool
aliases:
  - /blog/wayfinder-in-unity
---

I made what I call a “wayfinder” plugin in Unity in order to do something similar to Google Maps but for a custom map layout, such as a private venue or a campus.

It is formed of two main components: the Editor utility, to easily trace the paths of the custom map, and the runtime Wayfinder, which calculates and outputs the best path to go from one location to another.

In this example, I will use a Disneyland map I’ve found online, trace out it’s paths and check for the shortest path between a few points of interest.

{{<youtube IVmNwgkqiGA>}}

## The Editor Utility

The goal of the Editor utility is to offer a quick and easy way to trace the map. It’s very manual but quite efficient though. It took me about 20 minutes to map out more than half of the Disneyland map.

### Creating Nodes

In “additive mode”, any left clicks on the scene view, will create a new node.

With two nodes selected, pressing L on the keyboard will link the nodes, creating a path between them.

![](/img/posts/wayfinder-in-unity-pathfinding-on-custom-maps/create_nodes.gif)

### Following a Path

Holding shift while creating a new node will automatically link it to the currently selected node.

This is especially useful to create a path following a curve.

![](/img/posts/wayfinder-in-unity-pathfinding-on-custom-maps/link_nodes.gif)

## The Runtime Wayfinder

The runtime wayfinder is a class that takes a set of nodes and links, and provides a function to get the shortest path between two nodes.
The credit for this rather efficient algorithm goes to [Jules](https://twitter.com/JulesLagard).

![](/img/posts/wayfinder-in-unity-pathfinding-on-custom-maps/get_shortest_path.gif)

It works in realtime, here calculating the shortest path every frame.

```csharp
using System.Collections.Generic;
using System.Linq;
using UnityEngine;
using UnityEngine.Assertions;
using UnityEngine.Profiling;

namespace MadStark.Wayfinder
{
    public class JulesStarBrain
    {
        readonly Dictionary<Transform, List<Link>> linksLuT;
        readonly List<Path> openPaths;
        readonly HashSet<Link> alreadyExploredLinks;

        private JulesStarBrain() { }

        public JulesStarBrain(Dictionary<Transform, List<Link>> anchorsAndLinks)
        {
            linksLuT = anchorsAndLinks;
            openPaths = new List<Path>();
            alreadyExploredLinks = new HashSet<Link>();
        }

        public Path GetBestPath(IEnumerable<Transform> startNodes, HashSet<Transform> ends)
        {
            Profiler.BeginSample("JulesStar.GetBestPath");

            openPaths.Clear();
            foreach (Transform node in startNodes)
                openPaths.Add(new Path(node));

            alreadyExploredLinks.Clear();

            Path r = JulesStarRecursive(openPaths, ends);

            Profiler.EndSample();

            return r;
        }

        private Path JulesStarRecursive(List<Path> paths, HashSet<Transform> targetNodes)
        {
            if (paths.Count == 0)
                return null; // Start and end are not connected

            paths = paths.OrderBy(x => x.calculatedLength).ToList();
            var pathToExplore = paths.First();
            paths.Remove(pathToExplore);

            if (targetNodes.Contains(pathToExplore.lastNode))
                return pathToExplore; // Done, return best path

            List<Link> connections = linksLuT[pathToExplore.lastNode];
            for (int i = 0; i < connections.Count; i++)
            {
                Link connection = connections[i];
                if (alreadyExploredLinks.Contains(connection))
                    continue; // Node already explored: skip

                alreadyExploredLinks.Add(connection);

                Transform previousAnchor = pathToExplore.lastNode;
                Transform nextAnchor = connection.GetOtherNode(previousAnchor);

                Path pathBranch = new Path(pathToExplore);
                do
                {
                    pathBranch.links.Add(connection);
                    pathBranch.calculatedLength += connection.CalculateCost();

                    if (targetNodes.Contains(nextAnchor))
                        break;

                    var connectionsToNextAnchor = linksLuT[nextAnchor];
                    if (connectionsToNextAnchor.Count != 2)
                        break;

                    connection = connectionsToNextAnchor[0];
                    if (previousAnchor == connection.GetOtherNode(nextAnchor))
                        connection = connectionsToNextAnchor[1];

                    Assert.IsFalse(alreadyExploredLinks.Contains(connection), "Path should not have been explored already.");

                    previousAnchor = nextAnchor;
                    nextAnchor = connection.GetOtherNode(nextAnchor);

                } while (true);

                alreadyExploredLinks.Add(connection);
                pathBranch.lastNode = nextAnchor;
                paths.Add(pathBranch);
            }

            // Continue exploring this path
            return JulesStarRecursive(paths, targetNodes);
        }
    }
}
```

### The full code is available on [github](https://github.com/MadStark/UnityWayfinder).

## To Go Further

There’s a few ways this plugin can be extended.

For instance, there’s an opportunity to have the application figure out where the user is on the map, in order to provide the best path from current location to a destination. One option would be to use GPS. However, as this plugin is mostly made for small, maybe even indoor areas, this may not be the best option. The best option in my opinion would be to place QR codes around the venue that the user can then scan and those codes would contain location information.
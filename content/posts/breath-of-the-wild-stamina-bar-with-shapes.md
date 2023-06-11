---
title: "Breath of the Wild Stamina Bar With Shapes"
date: 2020-08-16T19:50:32+01:00
draft: false
toc: false
description: "Trying out Shapes in Unity by Freya Holmér. I made the Breath of the Wild stamina wheel in a matter of minutes!"
thumbnail: "/img/posts/breath-of-the-wild-stamina-bar-with-shapes/thumbnail.png"
tags:
  - unity
  - csharp
  - vfx
aliases:
  - /blog/breath-of-the-wild-stamina-bar-with-shapes
---

Shapes is a plugin for Unity made by [Freya Holmér](https://twitter.com/FreyaHolmer). I’ve been looking for a quick project to try my hands at it.

Breath of the Wild is one of my main inspirations when it comes to simple and effective animations and VFX. The game’s stamina wheel has been recreated in Unity by quite a few people using UGUI and `fillAmount` property on the Image component. I thought that it could be a lot easier with Shapes, especially the immediate mode.

![](/img/posts/breath-of-the-wild-stamina-bar-with-shapes/Screenshot+2020-08-06+at+10.23.44.png)

{{<youtube NUYfKfR9Ygw>}}

## Shapes Preparations

Following the quick startup guide, to draw UI with Shapes, I used a script on the main camera that has a function OnPostRender. From this function, I will draw my UI with the Shapes immediate API.

The first thing I do is setup the global settings to fit my needs: 

```csharp
private void OnPostRender()
{
    Draw.Matrix = transform.localToWorldMatrix;
    Draw.BlendMode = ShapesBlendMode.Transparent;

    float maxFill = Mathf.Clamp(this.maxFill, 1, 3);
    float fillAmount = Mathf.Clamp(this.fillAmount, 0, maxFill);

    // Draw wheels here
}
```

This will:
* Draw any shape in local space of my camera.
* Allow some shapes to be transparent. Here, the background of the wheel is black with transparency.

I also clamp my class properties to avoid any visual glitches if they are set wrong. Therefore I limit the max stamina to 3 just like in the game, and clamp the current stamina between zero and the max stamina possible.

## Drawing the Main Stamina Bar

The first step was to draw the main wheel. The background is a ring, and Shapes does have a ring function. However, to prepare for the next section, I am going to use the Arc function even for the background of the main wheel.

Some variables are declared on a class level, so I can edit them in inspector. I also need to rotate it all 90 degrees anti-clockwise, which I did by passing a Quaternion roll as second parameter of both Shapes functions. Here is the code for drawing a stamina wheel.

```csharp
private void OnPostRender()
{
    // ...
    
    DrawWheel(mainRadius, mainThickness, maxFill: 1, fillAmount);
}

private void DrawWheel(float radius, float thickness, float maxFill, float fillAmount)
{
    // Draw background
    float maxAngleEnd = maxFill * ShapesMath.TAU;
    Draw.Arc(localPosition, roll, radius, thickness * 0.95f, 0, maxAngleEnd, backgroundColor);

    // Draw fill
    float fillAngleEnd = fillAmount * ShapesMath.TAU;
    Draw.Arc(localPosition, roll, radius, thickness, 0, fillAngleEnd, fillColor);
}
```

Already, the design is usable in a game!

![](/img/posts/breath-of-the-wild-stamina-bar-with-shapes/Screenshot+2020-08-15+at+12.49.02.png)

## Drawing Stamina Upgrades

In Breath of the Wild, you can increase the max stamina throughout the game. And this will add up to 2 bigger and thiner rings, around the main one.

If the main wheel is 1.0 stamina, each upgrade in game will grant you +0.2 stamina.

Here, with 3 upgrades, I have a total of 1.6 stamina.

![](/img/posts/breath-of-the-wild-stamina-bar-with-shapes/Screenshot+2020-08-06+at+16.01.27.png)

Using the generic function I wrote in the above section, I can simply do the following:

```csharp
private void OnPostRender()
{
    // ...
    
    float secondaryWheelMax = Mathf.Clamp01(maxFill - 1);
    float secondaryWheelFill = Mathf.Clamp01(fillAmount - 1);
    DrawWheel(secondaryRadius * scale, secondaryThickness * scale, secondaryWheelMax, secondaryWheelFill);

    float tertiaryWheelMax = Mathf.Clamp01(maxFill - 2);
    float tertiaryWheelFill = Mathf.Clamp01(fillAmount - 2);
    DrawWheel(tertiaryRadius * scale, secondaryThickness * scale, tertiaryWheelMax, tertiaryWheelFill);
}
```

I could have made a for loop to handle any amount of bonus stamina rings but that would have been unnecessary as it never goes above 3 complete wheels in the game.

## Extra Polish

The stamina wheel in Breath of the Wild has two more features. Firstly, it shows how fast the stamina wheel is getting consumed by the current action (running, climbing, etc). And secondly, if you fully consume the stamina it will stay red until it fills back up.

I called those the ‘consumption’ and the ‘recovery’ features.

Here is the final component!

```csharp
using UnityEngine;
using Shapes;

[ExecuteInEditMode]
public class StaminaUI : MonoBehaviour
{
    [Header("Transform")]
    public float scale = 1;
    public Vector3 localPosition;
    
    [Header("Visuals")]
    public float mainRadius;
    public float mainThickness;
    public float secondaryRadius;
    public float secondaryThickness;
    public float tertiaryRadius;
    
    [Header("Stats")]
    public float maxFill;
    public float fillAmount;
    public float consumption;
    public bool recovering;

    [Header("Colours")]
    [ColorUsage(true, true)]
    public Color backgroundColor;
    [ColorUsage(true, true)]
    public Color fillColor;
    [ColorUsage(true, true)]
    public Color consumptionColor;
    [ColorUsage(true, true)]
    public Color recoveringColor;
    
    private static readonly Quaternion roll = Quaternion.AngleAxis(90, Vector3.forward);

    
    private void OnPostRender()
    {
        Draw.Matrix = transform.localToWorldMatrix;
        Draw.BlendMode = ShapesBlendMode.Transparent;

        float maxFill = Mathf.Clamp(this.maxFill, 1, 3);
        float fillAmount = Mathf.Clamp(this.fillAmount, 0, maxFill);
        
        DrawWheel(mainRadius * scale, mainThickness * scale, 1, Mathf.Clamp01(fillAmount));
        DrawWheel(secondaryRadius * scale, secondaryThickness * scale, Mathf.Clamp01(maxFill - 1), Mathf.Clamp01(fillAmount - 1));
        DrawWheel(tertiaryRadius * scale, secondaryThickness * scale, Mathf.Clamp01(maxFill - 2), Mathf.Clamp01(fillAmount - 2));
    }
    
    private void DrawWheel(float radius, float thickness, float maxFill, float fillAmount)
    {
        bool consuming = consumption > 0f && fillAmount > 0f && fillAmount < 1f;

        // Draw background
        float maxAngleEnd = maxFill * ShapesMath.TAU;
        Draw.Arc(localPosition, roll, radius, thickness * 0.95f, 0, maxAngleEnd, backgroundColor);
        
        if (consuming && !recovering)
        {
            float fillAngleEnd = Mathf.Clamp01(fillAmount - consumption) * ShapesMath.TAU;
            // Draw fill
            Draw.Arc(localPosition, roll, radius, thickness, 0, fillAngleEnd, fillColor);
            // Draw consumption
            Draw.Arc(localPosition, roll, radius, thickness, fillAngleEnd, fillAmount * ShapesMath.TAU, consumptionColor);
        }
        else
        {
            // Draw fill
            Draw.Arc(localPosition, roll, radius, thickness, 0, fillAmount * ShapesMath.TAU, recovering ? recoveringColor : fillColor);
        }
    }
}
```

And here is the inspector configuration for the nicest results!

![](/img/posts/breath-of-the-wild-stamina-bar-with-shapes/Screenshot+2020-08-06+at+10.24.43.png)


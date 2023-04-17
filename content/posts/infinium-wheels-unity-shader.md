---
title: "Infinium Wheels Unity Shader"
date: 2020-07-27T20:02:24+01:00
draft: false
toc: false
description: "Attempt at reproducing my favourite wheels from Rocket League. I talk shaders, Universal Render Pipeline and view direction."
thumbnail: "/img/posts/infinium-wheels-unity-shader/thumbnail.png"
tags:
  - unity
  - shader
  - vfx
---

## The Infinium wheels

Those are my favourite wheels in Rocket League, of course because since the day they came out, I wanted to make a shader of them.

![](/img/posts/infinium-wheels-unity-shader/ps4-infinium-TitaniumWhite.png)

It was when I landed on [this post by Harry Alisavakis](https://halisavakis.com/my-take-on-shaders-parallax-effect-part-ii/), that I finally got around making it.

To make it a little bit harder for me, I wanted to make them as Universal Render Pipeline as possible. So it means not using UnityCG.cginc and a lot of digging around to find the functions that do what I want. But it was worth it because in my opinion, it makes the code look that much more modern.

Here is the final result!

![](/img/posts/infinium-wheels-unity-shader/infinium.png)

{{<youtube X8AZJ8GJoB0>}}

Now let’s break it down.

## The variables

In Varyings, I have the usual suspects: vertex, color and uv. But what I also need is the view direction, in tangent space. I named the property viewDireTS.

To calculate this one, I need to ask for the normal and the tangent in object space.

![](/img/posts/infinium-wheels-unity-shader/rider64_2020-07-26_22-20-19.png)

## The vertex function

To get the tangent of the view direction, I decided to use the function TransformWorldToTangent. It takes as parameter, the view direction, and a tangent world.

Conveniently, there’s a function CreateTangentWorld in the includes of the scriptable render pipeline package. I need to pass it the normal and the tangent in world space, and the tangent sign.

I get the tangent sign using the technique shown in Harry’s post. I convert the normal and tangent that Unity gives me in object space, to world space.

![](/img/posts/infinium-wheels-unity-shader/rider64_2020-07-26_22-46-18.png)

When making a shader for Universal Render Pipeline, I do not include the usual `UnityCG.cginc`. Instead I go for the includes that are provided with URP, or Scriptable RP packages. Here, I used `Core.hlsl` for the basic functions such as `GetVertexPositionInput` and `SpaceTransforms.hlsl` to get all the nice functions that help me convert vectors to the desired space.

```hlsl
#include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Core.hlsl"
#include "Packages/com.unity.render-pipelines.core/ShaderLibrary/SpaceTransforms.hlsl"
```

## The frag function

This is where the magic happens.

I offset the sampling of the texture by the view direction, giving this fake depth effect. And I make sure the furthest layers are also more and more faded out.

`UNITY_UNROLL` is not necessary, but it ensures Unity will duplicate and replace this loop at compilation, as many times as the loop would have run.

![](/img/posts/infinium-wheels-unity-shader/image-asset.png)

## Final tweaks

By default, a texture is set to Repeat, in Unity. In our case, this caused an issue, as the furthest layers would start looping, looking from the side. 

![](/img/posts/infinium-wheels-unity-shader/Unity_2020-07-26_23-25-43.png)

Simply changing the Wrap Mode to Clamped gets rid of the problem.

![](/img/posts/infinium-wheels-unity-shader/Unity_2020-07-26_23-22-09.png)

## The 3D model

The 3D model is the same as the one in [my post about the Zomba wheels](/posts/zomba-wheels-in-shader-graph). Refer to that if you want to know more.

## The code

Here is the code for the shader.

```hlsl
Shader "Custom/Wheels/Infinium"
{
    Properties
    {
        _BaseMap ("Base Map", 2D) = "white" {}
        _BaseColor ("Base Color", Color) = (1,1,1,1)
        _ParallaxOffsetStep ("Parallax Offset Step", Vector) = (0.05, 0.05, 0, 0)
    }
    SubShader
    {
        Tags { "RenderType"="Opaque" "Queue"="Geometry" }
        ZWrite On

        Pass
        {
            Name "Unlit"
            HLSLPROGRAM
            #pragma vertex vert
            #pragma fragment frag

            #include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Core.hlsl"
            #include "Packages/com.unity.render-pipelines.core/ShaderLibrary/SpaceTransforms.hlsl"

            struct Attributes
            {
                float4 positionOS : POSITION;
                float2 uv : TEXCOORD0;
                float3 normalOS : NORMAL;
                float4 tangentOS : TANGENT;
            };

            struct Varyings
            {
                float4 vertex : SV_POSITION;
                float4 color : COLOR;
                float2 uv : TEXCOORD0;
                float3 viewDirTS : TEXCOORD1;
            };

            TEXTURE2D(_BaseMap); SAMPLER(sampler_BaseMap); float4 _BaseMap_ST;
            half4 _BaseColor;
            float2 _ParallaxOffsetStep;

            Varyings vert(Attributes input)
            {
                Varyings output = (Varyings)0;

                VertexPositionInputs vertexInput = GetVertexPositionInputs(input.positionOS.xyz);
                output.vertex = vertexInput.positionCS;
                output.color = _BaseColor;
                output.uv = TRANSFORM_TEX(input.uv, _BaseMap);

                float tangentSign = input.tangentOS.w * unity_WorldTransformParams.w;
                float3 normalWS = TransformObjectToWorldNormal(input.normalOS);
                float3 tangentWS = TransformObjectToWorldDir(input.tangentOS.xyz);
                float3x3 tangentToWorld = CreateTangentToWorld(normalWS, tangentWS, tangentSign);
                float3 viewDir = vertexInput.positionWS.xyz - _WorldSpaceCameraPos;
                output.viewDirTS = TransformWorldToTangent(viewDir, tangentToWorld);

                return output;
            }

            half4 frag(Varyings input) : SV_Target
            {
                half4 color = (half4)0;

                int count = 10;
                float offset = _ParallaxOffsetStep.x;
                float3 viewDirTS = normalize(input.viewDirTS);

                UNITY_UNROLL
                for (int i = count; i >= 0; i--)
                {
                    // Offset by view direction and sample the texture
                    float2 texoffset = input.viewDirTS.xy * offset;
                    float fadeaway = (float)i / count;
                    half4 layer = SAMPLE_TEXTURE2D(_BaseMap, sampler_BaseMap, input.uv + texoffset) * fadeaway;

                    // Blend layer with the previous layers
                    color = max(layer, color);

                    // Increase offset
                    offset += _ParallaxOffsetStep.y;
                }

                color *= input.color;
                return color;
            }
            ENDHLSL
        }
    }
}
```

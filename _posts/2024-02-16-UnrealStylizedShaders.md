---
title: Unreal Stylized Shader - Specialisation project
categories: [Graphic, UnrealEngine]
tags: [unrealengine, c++, shader, post-process, hlsl]
image: assets/images/ueshaderwork/FireFly.gif
---

During the last year of bachelor's degree in Game Programming at the SAE-Institute, the students of the Games Programming section had to create a game in collaboration with the Game Art and Audio Engineering sections. The purpose of the module was to simulate what was, for some, a first work experience in a professional-like environment.

## Contextualisation

For this project I had the opportunity to work as a Graphic Programmer on two game projects made in the Unreal Engine 5.3.

Project Girl & Kitty  | Project VR
:-----:|:-----:
![]({{ site.baseurl }}/assets/images/ueshaderwork/GK_Map_01.PNG "GK_First_Map"){: width="100%"} | ![]({{ site.baseurl }}/assets/images/ueshaderwork/VR_Map_01.PNG "VR_First_Map"){: width="100%"} 

For both project we had a limited production time, roughly 8 months, with a team of 27 people.

The tasks I worked on were the following:
- Creation of [shaders](https://en.wikipedia.org/wiki/Shader) and functions to replicate a watercolour style
- Graphic tools for the artists
- Asset integration, colorization, test and review
- Creation of Landscape shaders, [Virtual Texturing](https://docs.unrealengine.com/4.26/en-US/RenderingAndGraphics/VirtualTexturing/)
- Creation of a stylized water shader
- Creation of specific objects shaders and code (Plant Growth, Glowing Outline, Vertex deformation)

The VR project, being [PBR](https://en.wikipedia.org/wiki/Physically_based_rendering) based, didn't require specific shaders. For that reason, the focus will be emphasized on the research, iterations and global work done for the Girl & Kitty project.

The first elements to acknowledge is the art direction. One of the references given for the art style was "Dordogne"

![]({{ site.baseurl }}/assets/images/ueshaderwork/Dordogne.PNG "Dordogne in game footage"){: width="100%"}

The only problem was that compared to Dordogne, a game made with hand-painted textures by "[Un je ne sais quoi](https://unjenesaisquoi.fr/)" a professional french studio, we didn't have as much time and would risk having incoherent texturing from the artists. For that reason a major part of the texturing was done with shaders.

## Shaders

In this blogpost, I am going to detail the steps of the major task I have been doing during these projects as a graphic programmer.

In this section the following shaders will be exposed and detailed: 
- [Stepped cell shading](#stepped-cell-shading)
- [Watercolor shader](#watercolor-shader)
- [Rim function](#rim-function)
- [Landscape and vitrual texturing](#landscape-and-vitrual-texturing)
- [Water shader](#water-shader)  

Multiple iterations were done before being validated by the "Art Director", the head of the Game Art section.
![]({{ site.baseurl }}/assets/images/ueshaderwork/GK_Iterations.PNG "Shader Iterations"){: width="100%"}

### Stepped cell shading

Since a major part of stylized games use cell shading, that was the first shader I tried to produce.
After multiple tests using sequenced unreal `if` nodes, I finally created a custom [HLSL](https://en.wikipedia.org/wiki/High-Level_Shader_Language) shader using the custom unreal node.

![]({{ site.baseurl }}/assets/images/ueshaderwork/MF_CellShade.PNG "Cell Shader Material Function"){: width="100%"}

In this material function (MF_) I get the SkyAtmosphereLightDirection and do a dot product with the `VertexNormalWS` (Object's Vertex Normal in World Space) to set a multiplication value of the BaseColor according to the position of the SkyLight. The internal code of the shader is the following:

```c++
float3 col;
float T = 0;
int steps = Steps;
float Tstep = 1.0/steps;

for(int i = 0; i < steps; i++)
{
    if(LightIntensity < T)
    {
        col = BaseColor * T;
        return col;
    }
     T += Tstep;
}

return BaseColor;
```

To use this material function it is necessary to branch the function as shown below:

![]({{ site.baseurl }}/assets/images/ueshaderwork/MF_CellShade_Use.PNG "How to use the Material Function"){: width="100%"}

The variables to input are:
- `BaseColor`: The object's color
- `Exposure`: The value by which we want to multiply the final colors.
- `Steps`: The amount of desired separations 

The result is the following:

![]({{ site.baseurl }}/assets/images/ueshaderwork/MF_CellShade_Result.PNG "Material Function result"){: width="100%"}

### Watercolor shader

This shader had the principle task of replicating a watercolor effect on the assets in local/world space.
After multiple trials the final version of my shader was transformed in a material function to simplify it's modification.

![]({{ site.baseurl }}/assets/images/ueshaderwork/MF_Watercolor.PNG "Watercolor Material Function"){: width="100%"}

This function gave the user the ability to change the watercolor effect of each material instance that used this material function.

![]({{ site.baseurl }}/assets/images/ueshaderwork/MF_Watercolor_Params.PNG "watercolor shader use"){: width="100%"}

The different parameters are the following:
- `Metalness & Roughness`: Value between 0 and 1 to define these values.
- `Enable_Texture`: Enables the possibility to input a texture as `BaseColor`.
- `Enable_Second_Colour`: Enables the possibility to have a second color channel.
- `Second_Texture_Multiplier`: Changes the saturation value of the second texture

The `Second_` values are used with a Noise material function to give the diffusion effect to objects.
![]({{ site.baseurl }}/assets/images/ueshaderwork/MF_Noise.PNG "Noise Material Function"){: width="100%"}

The Noise parameters are the following: 
- `Noise_Scale_X & _Y`: Used to scale the given noise.
- `Black & White_Values`: Used to clamp the Black and White values of the given noise.
- `Noise_Texture`: The Noise used to "spread" colors.
- `Invert_Noise`: Inverts the black and white values of the Noise texture.

To use it in a Material, the `MF_Noise` is given as an alpha to a `Lerp` with the `BaseColors`
![]({{ site.baseurl }}/assets/images/ueshaderwork/Watercolor_Use.PNG "Watercolour Use"){: width="100%"}


The final result gives some sort of a color "spread".
![]({{ site.baseurl }}/assets/images/ueshaderwork/GK_Render.PNG "Boat with watercolor shader"){: width="100%"}

### Rim function
Since Dordogne has a specific outline on the characters that depends of the light source, I decided to create the same function in Unreal.

![]({{ site.baseurl }}/assets/images/ueshaderwork/dordogne.jpg "dordogne outline"){: width="100%"}

It was accomplished by using a `Fresnel Node` and subtracting it with the inverse of the dot between the light source and the `VertexNormalWS` (vertex normal world space) node.

![]({{ site.baseurl }}/assets/images/ueshaderwork/MF_Rim.PNG "Rim material function"){: width="100%"}


To use it I connected it to either the `Base Color` or the `Emissive Color` The final result was the following:

![]({{ site.baseurl }}/assets/images/ueshaderwork/Rim_Shader.gif "Dynamic rim shader"){: width="100%"}


### Landscape and vitrual texturing

### Water shader

The final result of the watercolour shader was the following:

## Post Processing

For this part of the blog post, we are not going to go in depth trough the work process since it is not the intended focus point. Instead a simple overview of the final shader's state will be exposed.

The idea of using a post processing shader came first of all from the necessity of having a screen space effect that would give a diffusion of colour between objects in a scene. To accomplish that effect a simple shader using [HLSL](https://fr.wikipedia.org/wiki/High-Level_Shading_Language) was created to obtain a [Kuwahara](https://en.wikipedia.org/wiki/Kuwahara_filter) filter effect in screen space.

The Unreal Engine material node looked like this:

![]({{ site.baseurl }}/assets/images/ueshaderwork/GK_KuwaharaNode.PNG "Kuwahara filter node in UnrealEngine 5.3"){: width="100%"}

and the HLSL code was the following:

```c++
float3 mean[4] = 
{
    {0, 0, 0},
    {0, 0, 0},
    {0, 0, 0},
    {0, 0, 0}
};
float3 sigma[4] = 
{
    {0, 0, 0},
    {0, 0, 0},
    {0, 0, 0},
    {0, 0, 0}
};
float2 offsets[4] = 
{
    {-RADIUS, -RADIUS}, 
    {-RADIUS, 0}, 
    {0, -RADIUS}, 
    {0, 0};
}
float2 pos;
float3 col;
for(int i = 0; i < 4; i++){
    for(int j = 0; j <= RADIUS; j++){
        for(int k = 0; k <= RADIUS; k++){
            pos = float2(j,k) + offsets[i];
            float2 uvpos = UV + pos/VIEWSIZE;
            col = SceneTextureLookup(uvpos, 14, false);

            mean[i] += col;
            sigma[i] += col * col;
        }
    }
}

float n = pow(RADIUS + 1, 2);
float sigma_f;
float min = 1;
for(int l = 0; l < 4; l++)
{
    mean[l] /= n;
    sigma[l] = abs(sigma[l] / n - mean[l] *  mean[l]);
    sigma_f = sigma[l].r + sigma[l].g + sigma[l].b;

    if(sigma_f < min){
        min = sigma_f;
        col = mean[l];
    }
}
return col;
```


To use it, we just had to create a [PostProcessVolume](https://docs.unrealengine.com/5.0/en-US/post-process-effects-in-unreal-engine/) in the scene and plug it in "Rengering Features -> Post Process Materials -> Array"

![]({{ site.baseurl }}/assets/images/ueshaderwork/GK_PP_use.PNG "Kuwahara filter use"){: width="100%"}

Here you can see the result of the post processing shader being used:

![]({{ site.baseurl }}/assets/images/ueshaderwork/GK_KuwaharaFilter.gif "Kuwahara filter effect"){: width="100%"}

## Final result

After multiple trials and exchanges with colleges and teachers, the results were the following:

Shader Only  | Shader + PostProcess
:-----:|:-----:
![]({{ site.baseurl }}/assets/images/ueshaderwork/Final_without_PP.PNG "Watercolour shader without post-process"){: width="100%"} | ![]({{ site.baseurl }}/assets/images/ueshaderwork/Final_with_PP.PNG "Watercolour shader with post-process"){: width="90%"} 

Final result of shaders
{: style="text-align: center"}
![]({{ site.baseurl }}/assets/images/ueshaderwork/Final_PP_Scene.PNG "Final Scene"){: width="100%"}

## Conclusion

123

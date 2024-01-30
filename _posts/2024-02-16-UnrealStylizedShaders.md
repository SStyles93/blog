---
title: Unreal Stylized Shader - Specialisation project
categories: [Graphic, UnrealEngine]
image: assets/images/ueshaderwork/FireFly.gif
---

Creating stylized shaders with a specific artistic direction in a student game project was the reason of the research and work exposed in this blogpost.

## Contextualization
During the last year of bachelor's degree in Game Programming at the SAE-Institute, the students of the Games Programming section had to create a game in collaboration with the Game Art and Audio Engineering sections. The purpose of the module was to simulate what was, for some, a first work experience in a professional-like environment.

For this project I had the opportunity to work as a Graphic Programmer on two game projects made in the Unreal Engine 5.3.

Project Girl & Kitty  | Project VR
:-----:|:-----:
![]({{ site.baseurl }}/assets/images/ueshaderwork/GK_Map_01.PNG "GK_First_Map"){: width="100%"} | ![]({{ site.baseurl }}/assets/images/ueshaderwork/VR_Map_01.PNG "VR_First_Map"){: width="100%"} 

For both project we had a limited production time, roughly 8 months, with a team of 27 people.

The tasks I worked on were the following:
- Creation of [shaders](https://en.wikipedia.org/wiki/Shader) and functions to replicate a watercolor style
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
- [Landscape and vitrual texturing](#landscape-and-virtual-texturing)
- [Water shader](#water-shader)  

Multiple iterations were done before being validated by the "Art Director", the head of the Game Art section.
![]({{ site.baseurl }}/assets/images/ueshaderwork/GK_Iterations.PNG "Shader Iterations"){: width="100%"}

### Stepped cell shading
#### Cell shading function
Since a major part of stylized games use cell shading, that was the first shader I tried to produce.
After multiple tests using sequenced unreal `if` nodes, I finally created a custom [HLSL](https://en.wikipedia.org/wiki/High-Level_Shader_Language) shader using the [Custom unreal node](https://docs.unrealengine.com/5.2/en-US/custom-material-expressions-in-unreal-engine/).

![]({{ site.baseurl }}/assets/images/ueshaderwork/MF_CellShade.PNG "Cell Shader Material Function"){: width="100%"}

In this material function (MF_) I get the SkyAtmosphereLightDirection and do a dot product with the `VertexNormalWS` (see [Coordinates Expressions](https://docs.unrealengine.com/4.27/en-US/RenderingAndGraphics/Materials/ExpressionReference/Coordinates/)) to set a multiplication value of the BaseColor according to the position of the SkyLight. 

The node requires to explicitly add the `LightDirection`, `Basecolor` and `Steps` variables to the custom node.

![]({{ site.baseurl }}/assets/images/ueshaderwork/MF_CellShade_Var.PNG "Cell Shading Variables"){: width="50%"}

To use this material function it is necessary to branch the function as shown below:

![]({{ site.baseurl }}/assets/images/ueshaderwork/MF_CellShade_Use.PNG "How to use the Material Function"){: width="100%"}

The variables to input are:
- `BaseColor`: The object's color/texture
- `Exposure`: The value by which we want to multiply the final colors.
- `Steps`: The amount of desired separations 

#### HLSL code
The custom's node internal code was the following:

```c++
float3 col;
float T = 0;
int steps = Steps;
float Tstep = 1.0/steps;

for(int i = 0; i < steps; i++)
{
    if(LightDirection < T)
    {
        col = BaseColor * T;
        return col;
    }
     T += Tstep;
}

return BaseColor;
```
In this code, the `float3` "col" is the color that is returned at every iteration of the for loop.  
The `T` variable is the current value of a step.  
`Tstep` is the value of one step, obtained by dividing 1 by the amount of steps that are input.
`LightDirection` is the value of the vector that is evaluated to define the limitation of colors.

The `for` loop iterated until the amount of `steps` is met. Its content evaluates the `LightDirection` according to the current step `T`. If it is lower, the returned color is equal to the `BaseColor` multiplied by the step's value. The current step's value is then incremented by `Tstep`. When the `for` loop has ended the `BaseColor` is returned for the "non-shaded" parts.

#### Result
The result of this node used with plane white (1.0, 1.0, 1.0, 1.0) and 3 steps is the following:

![]({{ site.baseurl }}/assets/images/ueshaderwork/MF_CellShade_Result.PNG "Material Function result"){: width="100%"}

### Watercolor shader
#### Watercolor function
This shader had the principle task of replicating a watercolor effect on the assets in local/world space.
After multiple trials the final version of my shader was transformed in a material function to simplify it's modification.

![]({{ site.baseurl }}/assets/images/ueshaderwork/MF_Watercolor.PNG "Watercolor Material Function"){: width="100%"}

This function gave the user the ability to change the watercolor effect of each material instance that used this material function.

![]({{ site.baseurl }}/assets/images/ueshaderwork/MF_Watercolor_Params.PNG "watercolor shader use"){: width="100%"}

The different parameters are the following:
- `Metalness & Roughness`: Value between 0 and 1 to define these values.
- `Enable_Texture`: Enables the possibility to input a texture as `BaseColor`.
- `Enable_Second_color`: Enables the possibility to have a second color channel.
- `Second_Texture_Multiplier`: Changes the saturation value of the second texture

The `Second_` values are used with a Noise material function to give the diffusion effect to objects.

#### Noise function
The use of a noise function was the key element to have a "spreading" effect of both colors declared before
![]({{ site.baseurl }}/assets/images/ueshaderwork/MF_Noise.PNG "Noise Material Function"){: width="100%"}

The Noise parameters are the following: 
- `Noise_Scale_X & _Y`: Used to scale the given noise.
- `Black & White_Values`: Used to clamp the Black and White values of the given noise.
- `Noise_Texture`: The Noise used to "spread" colors.
- `Invert_Noise`: Inverts the black and white values of the Noise texture.

To use it in a Material, the `MF_Noise` is given as an alpha to a `Lerp` with the `BaseColors`.
![]({{ site.baseurl }}/assets/images/ueshaderwork/Watercolor_Use.PNG "Watercolor Use"){: width="100%"}

#### Result
The final result gives some sort of a color "spread" that is one of the key elements in the watercolor style.
![]({{ site.baseurl }}/assets/images/ueshaderwork/GK_Render.PNG "Boat with watercolor shader"){: width="100%"}

### Rim Shader
Since Dordogne has a specific outline on the characters that depends of the light source, I decided to create the same function in Unreal.

![]({{ site.baseurl }}/assets/images/ueshaderwork/dordogne.jpg "dordogne outline"){: width="100%"}

#### Rim Function
It was accomplished by using a `Fresnel Node` and subtracting it with the inverse of the dot between the light source and the `VertexNormalWS` (vertex normal world space) node.

![]({{ site.baseurl }}/assets/images/ueshaderwork/MF_Rim.PNG "Rim material function"){: width="100%"}

To use it, connected it to either the `BaseColor` or the `Emissive Color`.

BaseColor | Emissive Color
:-----:|:-----:
![]({{ site.baseurl }}/assets/images/ueshaderwork/MF_Rim_Use.PNG "Rim function use BC"){: width="100%"} | ![]({{ site.baseurl }}/assets/images/ueshaderwork/MF_Rim_Use1.PNG "Rim function use EC"){: width="100%"}

#### Result
The final result connected to the `Emissive Color` of a plane white material was the following:

![]({{ site.baseurl }}/assets/images/ueshaderwork/Rim_Shader.gif "Dynamic rim shader"){: width="100%"}


### Landscape and virtual texturing
After discussion with a Game Artist friend participating in the project she pointed out the necessity of having a landscape material, that would blend vegetation and the ground seemingly.

#### Landscape material
![]({{ site.baseurl }}/assets/images/ueshaderwork/VT_Path.PNG "Virtual Texturing Path base"){: width="100%"}
![]({{ site.baseurl }}/assets/images/ueshaderwork/VT_Grass.PNG "Virtual Texturing Grass base"){: width="100%"}
![]({{ site.baseurl }}/assets/images/ueshaderwork/VT_MakeMAT.PNG "Virtual Texturing make material"){: width="100%"}
![]({{ site.baseurl }}/assets/images/ueshaderwork/VT_RTVT.PNG "Virtual Texturing Run-time Virt.Texturing"){: width="100%"}

![]({{ site.baseurl }}/assets/images/ueshaderwork/VT_Blend.gif "Virtual Texturing blend"){: width="100%"}

### Water shader
## Post Processing
For this part of the blog post, we are not going to go in depth trough the work process since it is not the intended focus point. Instead a simple overview of the final shader's state will be exposed.

The idea of using a post processing shader came first of all from the necessity of having a screen space effect that would give a diffusion of color between objects in a scene. To accomplish that effect a simple shader using [HLSL](https://fr.wikipedia.org/wiki/High-Level_Shading_Language) was created to obtain a [Kuwahara](https://en.wikipedia.org/wiki/Kuwahara_filter) filter effect in screen space.

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
![]({{ site.baseurl }}/assets/images/ueshaderwork/Final_without_PP.PNG "Watercolor shader without post-process"){: width="100%"} | ![]({{ site.baseurl }}/assets/images/ueshaderwork/Final_with_PP.PNG "Watercolor shader with post-process"){: width="90%"} 

Final result of shaders
{: style="text-align: center"}
![]({{ site.baseurl }}/assets/images/ueshaderwork/Final_PP_Scene.PNG "Final Scene"){: width="100%"}

## Conclusion

123

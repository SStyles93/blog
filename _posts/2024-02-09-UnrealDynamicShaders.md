---
title: Unreal Dynamic Shaders - Specialisation project
categories: [Graphic, UnrealEngine]
tags: [unrealengine, c++, shader, post-process, hlsl]
image: assets/images/ueshaderwork/Outline_Use.gif
---

The necessity of creating interactable visual effects in an Unreal Engine game resulted in this research, the work that followed and finally this blogpost.

## Contextualisation

During the last year of bachelor's degree in Game Programming at the SAE-Institute, the students of the Games Programming section had to create a game in collaboration with the Game Art and Audio Engineering sections. The purpose of the module was to simulate what was, for some, a first work experience in a professional-like environment.

For this module I had the opportunity to work as a Graphic Programmer on two game projects made in the Unreal Engine 5.3.
The project that demanded the most work in terms of shader work was first called "The Project Girl & Kitty".
![]({{ site.baseurl }}/assets/images/ueshaderwork/GK_Map_01.PNG "GK_First_Map"){: width="100%"}
_Project Girl & Kitty Scene render_

For both project we had a limited production time, roughly 8 months, with a team of 27 people.
The tasks I worked on were the following:
- Creation of [shaders](https://en.wikipedia.org/wiki/Shader) and functions to replicate a watercolour style
- Creation of Landscape shaders, [Virtual Texturing](https://docs.unrealengine.com/4.26/en-US/RenderingAndGraphics/VirtualTexturing/)
- Creation of a stylized water shader
- Asset integration, colorization, test and review
- Creation of specific objects shaders and code (Plant Growth, Glowing Outline, Vertex deformation)
- Graphic tools for the artists

In this blogpost, I am going to detail the work done towards the creation of the following interactive shaders:
- [Outline function](#outline-function)
- [Distortion function](#distortion-function)
- [Object fading](#object-fading)

## Interactable class and dynamic materials

Before going in detail into the shader creation, the first thing to acknowledge is that in Unreal Engine, you can not modify objects materials in run-time since they are static. For that reason I had to code an "Interactable" component class so that every object that had it would override its materials and create a dynamic version of them.  

Below you can see the `Interactable.h` code related to the dynamic material creation:
```c++
#pragma once

#include "CoreMinimal.h"
#include "Components/ActorComponent.h"
#include "Interactable.generated.h"


UCLASS( ClassGroup=(Custom), meta=(BlueprintSpawnableComponent) )
class PROJECTGIRLANDKITTY_API UInteractable : public UActorComponent
{
	GENERATED_BODY()

public:	

	UInteractable();

	//...
	
protected:

	virtual void BeginPlay() override;


private:
	
	TArray<UMaterialInstanceDynamic*> DynMaterials;

    //...
};
```
As you can see the `DynMaterials` is an `Array` that stores the materials created at run-time.

And here the `Interactable.cpp` code:
```c++
#include "Interactable.h"

UInteractable::UInteractable()
{
	//...
}

void UInteractable::BeginPlay()
{
	Super::BeginPlay();

	TArray<UStaticMeshComponent*> Meshes;
	GetOwner()->GetComponents<UStaticMeshComponent>(Meshes, true);
	for (int32 i = 0; i < Meshes.Num(); i++)
	{
		UStaticMeshComponent* StaticMeshComponent = Meshes[i];
		UMaterialInterface* Material = StaticMeshComponent->GetMaterial(0);
		UMaterialInstanceDynamic* DynMaterial = UMaterialInstanceDynamic::Create(Material, GetOuter());
		DynMaterials.Add(DynMaterial);
		StaticMeshComponent->SetMaterial(0, DynMaterial);
	}

	//...
}
```
The code above takes all the `Meshes` and creates a dynamic material instance and stores it in the `DynMaterials` array.
Thanks to this component class, all exposed variables in shaders are now accessible via code or blueprint.

### Outline function

The necessity to have a dynamic outline, that reacted to the players position and rotation when he crossed interactable objects, brought me to create this "Coloured_Outline" Material Function. Using the `Fresnel`, `Time` & `Sine` Nodes I created an "Emissive" effect.

![]({{ site.baseurl }}/assets/images/ueshaderwork/MF_Outline_Colour.PNG "MF_Coloured_Outline"){: width="100%"}

The `Enable_Outline` parameter (float) is used in the C++ code to activate it or not. I first went with a `Static bool` parameter but after research, since it is static, you can not modify it in code thus rendering it's activation impossible. Here is the code related to the "Interactable" objects:

Interactable.h
```c++
UPROPERTY(EditAnyWhere, BlueprintReadWrite, Category = "Visuals|Outline", meta = (AllowPrivateAccess = "true", ClampMin = "0.0", ClampMax = "1.0", UIMin = "0.0", UIMax = "1.0"))
float MaxOutlineThickness = 0.3f;
UPROPERTY(EditAnyWhere, BlueprintReadWrite, Category = "Visuals|Outline", meta = (AllowPrivateAccess = "true"))
float OutlineThickness = 0.0f;	
UPROPERTY(EditAnyWhere, BlueprintReadWrite, Category = "Visuals|Outline", meta = (AllowPrivateAccess = "true"))
float CurrentOutlineValue = 0.0f;
//This float is used instead of a bool since materials only have static bools
float OutlineEnabled = 0.0f;
```
Thanks to the `OutlineThickness` I was able to get the value in blueprints and shader, thus enabling the possibility to modify them in game.

Interactable.cpp
```c++
void UInteractable::TickComponent(float DeltaTime, ELevelTick TickType, FActorComponentTickFunction* ThisTickFunction)
{
	Super::TickComponent(DeltaTime, TickType, ThisTickFunction);
	
	if (EnableOutline)
	{

		OutlineThickness >= MaxOutlineThickness ? OutlineThickness = MaxOutlineThickness : OutlineThickness += DeltaTime;
		OutlineEnabled = 1.0f;
	}
	else
	{
		OutlineThickness <= 0.0f ? OutlineThickness = 0.0f : OutlineThickness -= DeltaTime;
		OutlineEnabled = 0.0f;
	}
	if (CurrentOutlineValue != OutlineThickness)
	{
		CurrentOutlineValue = OutlineThickness;
		SetOutline(OutlineThickness);
	}
```
The code above gradually changes the value of the `OutlineTickness` when enabled.

The MF_Outline is the base function used to create the outline of objects. It simply colors every part of an object that has a normal vector direction according to the camera's position with a value superior to the `Outline_Thickness`.

![]({{ site.baseurl }}/assets/images/ueshaderwork/MF_Outline.PNG "MF_Outline"){: width="100%"}

I also linked an [Unreal Material Parameters Collection](https://docs.unrealengine.com/4.26/en-US/RenderingAndGraphics/Materials/ParameterCollections/) to it. The idea was to give the artist the possibility to change its color in a simple "Color palette" for that purpose 

![]({{ site.baseurl }}/assets/images/ueshaderwork/Outline_Use.gif "Outline Use"){: width="100%"}

### Distortion function
The distortion function was a simple function that just took the `Time`, `Panner` and `VertexNormalWS` nodes with a Normal map in a `TextureParameter2D` node to deform the object .

![]({{ site.baseurl }}/assets/images/ueshaderwork/MF_Distortion.PNG "Distortion MF"){: width="100%"}

The only specificity was to multiply the `R` and `G` channels by the `Distortion_Power`, that was controlled in the C++ code, so that the effect could be enabled or not.

Below you can find the related code in the `.h` file:
```c++
private:

	bool tempDistortionState = false;
	UPROPERTY(EditAnyWhere, BlueprintReadWrite, Category = "Visuals|Distortion", meta = (AllowPrivateAccess = "true", ClampMin = "0.0", ClampMax = "100.0", UIMin = "0.0", UIMax = "100.0"))
	float DistortionPower = 20.0f;
	UPROPERTY(EditAnyWhere, BlueprintReadWrite, Category = "Visuals|Distortion", meta = (AllowPrivateAccess = "true"))
	float DistortionSpeed = 0.25f;
	UPROPERTY(EditAnyWhere, BlueprintReadWrite, Category = "Visuals|Distortion", meta = (AllowPrivateAccess = "true", ClampMin = "-1.0", ClampMax = "1.0", UIMin = "-1.0", UIMax = "1.0"))
	float DistortionDirectionX = 0.0f;
	UPROPERTY(EditAnyWhere, BlueprintReadWrite, Category = "Visuals|Distortion", meta = (AllowPrivateAccess = "true", ClampMin = "-1.0", ClampMax = "1.0", UIMin = "-1.0", UIMax = "1.0"))
	float DistortionDirectionY = 0.03f;

	void SetOutline(float value);
	void SetDistortion(bool value);
	void SetDistortionValues();

```
and in the `.cpp` file:
```c++

void UInteractable::BeginPlay()
{
	Super::BeginPlay();

	//...

	SetDistortionValues();
}

void UInteractable::TickComponent(float DeltaTime, ELevelTick TickType, FActorComponentTickFunction* ThisTickFunction)
{
	Super::TickComponent(DeltaTime, TickType, ThisTickFunction);

    //...

#if WITH_EDITOR
	SetDistortionValues();
#endif

	if (tempDistortionState != EnableDistortion)
	{
		SetDistortion(EnableDistortion);
		tempDistortionState = EnableDistortion;
	}
}

/// <summary>
/// Sets the Outline Thickness
/// </summary>
/// <param name="Value">The value of the outline thickness</param>
void UInteractable::SetOutline(float value)
{
	for (auto& DynMat : DynMaterials)
	{
		DynMat->SetScalarParameterValue("Outline_Thickness", value);
		DynMat->SetScalarParameterValue("Enable_Outline", OutlineEnabled);
	}
}

/// <summary>
/// Sets the state of the distortion effect
/// </summary>
/// <param name="value">True = Enabled</param>
void UInteractable::SetDistortion(bool value)
{
	switch (value)
	{
	case true:
		for (auto& DynMat : DynMaterials)
		{
			DynMat->SetScalarParameterValue("Distortion_Power", DistortionPower);
		}
		break;
	case false:
		for (auto& DynMat : DynMaterials)
		{
			DynMat->SetScalarParameterValue("Distortion_Power", 0.0f);
		}
		break;
	}
}

/// <summary>
/// Sets the value of distortion for the Interactable Shaders
/// </summary>
void UInteractable::SetDistortionValues()
{
	for (auto& DynMat : DynMaterials)
	{
		DynMat->SetScalarParameterValue("Distortion_Speed", DistortionSpeed);
		DynMat->SetScalarParameterValue("Distortion_Power_X", DistortionDirectionX);
		DynMat->SetScalarParameterValue("Distortion_Power_Y", DistortionDirectionY);
	}
}

```

To use it I just had to branch it to world position offset in the object's material node.

![]({{ site.baseurl }}/assets/images/ueshaderwork/Distortion_Use.PNG "Distortion MF Use"){: width="100%"}

After branching it to the materials and calling

![]({{ site.baseurl }}/assets/images/ueshaderwork/Distortion_Use.gif "Distortion Use"){: width="100%"}

### Object fading



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

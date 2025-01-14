---
title: Depth of Field Post Process Effect in Unreal Engine with CPP
date: 2025-01-13 21:20:00 +0100
categories: ["Unreal Engine", "Post Process", "CPP"]
tags: [UE]
---

### Foreword

![Final Result](https://github.com/aiiaiiiyo/aiiaiiiyo.github.io/blob/main/assets/img/depthoffield.gif?raw=true)

In my upcoming horror game, I wanted to implement a Depth of Field effect on the camera. If you're not familiar, Depth of Field is the effect where the foreground appears sharp while the background is blurred.

There are two ways I can think of to achieve this:
- Modifying the Post-Process settings on the player's camera
- Modifying the Post-Process Volume in the scene

In this post, I'll focus on the first approach. However, I'll also briefly outline how the second method could be implemented.

To achieve the desired effect, we need a mechanism to call our function continuously. While it might seem straightforward to place this call in the `Tick` function, I prefer having finer control over the call frequency. Instead, we’ll use a timer function to call our update function at a specified interval.

### Define Variables and Update Function

First, we define the variables and the update function required for this process:

```cpp
UPROPERTY()
FTimerHandle DOFUpdateTimerHandle;

UPROPERTY(EditAnywhere, Category=" Camera")
float DOFUpdateRate = .2f; //Rate the update function is called

UFUNCTION()
void DepthOfFieldUpdate() const;
```
{: file="Character.h" }

The camera is already defined in our character class and was set up as follows:

```cpp
UPROPERTY(VisibleAnywhere, Category=" Camera")
TObjectPtr<UCameraComponent> FollowCamera = nullptr;
```
{: file="Character.h" }

It is initialized in the constructor and attached to a `SpringArmComponent`:

```cpp
FollowCamera = CreateDefaultSubobject<UCameraComponent>(TEXT("FollowCamera"));
FollowCamera->SetupAttachment(SpringArmComponent, USpringArmComponent::SocketName);
```
{: file="Character.cpp" }

### Set the Timer in BeginPlay
In the `BeginPlay` function, we set the timer with the required parameters:

```cpp
GetWorld()->GetTimerManager().SetTimer(
  DOFUpdateTimerHandle,
  this,
  &AOMCharacter::DepthOfFieldUpdate,
  DOFUpdateRate,
  true);
```
{: file="Character.cpp" }

- `DOFUpdateTimerHandle`: This timer handle manages the `DepthOfFieldUpdate` function, ensuring it's called repeatedly at the specified interval.
- `this`: Object to call the timer function on.
- `DepthOfFieldUpdate`: Method to call when timer fires, defined with the scope.
- `DOFUpdateRate`: This is the interval at which the function is called. In this case, it's set to `0.2` seconds.
- `true`: Keep firing at `DOFUpdateRate` intervals, false to fire only once.

### DepthOfFieldUpdate function
We need to cast a LineTrace from the camera position into the distance in a straight line and then set the distance as the Focal Distance on the camera.

First, we define the start and end points for the line trace:

We use the camera's world position as the starting point (`CamLocation`).
We multiply the camera's forward vector by an arbitrarily large number (e.g., `10000` in this case).
We add the starting point and the scaled forward vector to calculate the endpoint.
```cpp
FVector CamLocation = FollowCamera->GetComponentLocation();
FVector FwdVector = UKismetMathLibrary::Multiply_VectorFloat(FollowCamera->GetForwardVector(), 10000.f);
FVector End = UKismetMathLibrary::Add_VectorVector(CamLocation, FwdVector);
```

We cast a line trace and store the result in a `bool` named `bIsBlockingHitFound`.

```cpp
FHitResult HitResult;
bool bIsBlockingHitFound = GetWorld()->LineTraceSingleByChannel(
    HitResult,
    CamLocation,
    End,
    ECC_Visibility
);
```
{: file="Character.cpp" }

![Line Trace](https://github.com/aiiaiiiyo/aiiaiiiyo.github.io/blob/main/assets/img/linetrace.png?raw=true)
_The green line represents the distance we are trying to determine._

If we have a blocking hit, we subtract the hit result location from the camera position, and the length of this vector will be the distance to the object hit.
This distance can then be used as the Focal Distance in the Post-Process settings.
```cpp
if(bIsBlockingHitFound)
{
    float FocusLocation = UKismetMathLibrary::Subtract_VectorVector(HitResult.Location, FollowCamera->GetComponentLocation()).Length();
    
    FollowCamera->PostProcessSettings.bOverride_DepthOfFieldFocalDistance = true;
    FollowCamera->PostProcessSettings.DepthOfFieldFocalDistance =
        FMath::FInterpTo(FollowCamera->PostProcessSettings.DepthOfFieldFocalDistance, FocusLocation, DOFUpdateRate, 2.f);
}
```
{: file="Character.cpp" }

We need to set two additional parameters for this to work within our `BeginPlay`
```cpp
FollowCamera->PostProcessSettings.bOverride_DepthOfFieldFstop = true;
FollowCamera->PostProcessSettings.DepthOfFieldFstop = 1.f;
FollowCamera->PostProcessSettings.bOverride_DepthOfFieldMinFstop = true;
FollowCamera->PostProcessSettings.DepthOfFieldMinFstop = 11.f;
```
{: file="Character.cpp" }

> One important detail when setting post-process values from C++ is that you first need to set the `bOverride_` variable of the specific parameter to `true`, otherwise, it will not work.
{: .prompt-tip }

### Modifying the Post-Process Volume in the scene
If you want to approach the problem differently, you can set the same post-process parameters on one of the post-process volumes in the scene. Please note that a post-process volume must be present in your level, and you need to specify the correct index for it in the code below.

```cpp
FPostProcessVolumeProperties PostProcessVolumeProps = GetWorld()->PostProcessVolumes[0]->GetProperties();
FPostProcessSettings* PostProcessVolumeSettings = (FPostProcessSettings*)PostProcessVolumeProps.Settings;
PostProcessVolumeSettings->bOverride_DepthOfFieldFstop = true;
PostProcessVolumeSettings->DepthOfFieldFstop = 1.f;
PostProcessVolumeSettings->bOverride_DepthOfFieldMinFstop = true;
PostProcessVolumeSettings->DepthOfFieldMinFstop = 11.f;
PostProcessVolumeSettings->bOverride_DepthOfFieldFocalDistance = true;
PostProcessVolumeSettings->DepthOfFieldFocalDistance = FocusLocation;
```
{: file="Character.cpp" }

With this setup, you now have a Depth of Field effect that updates periodically.

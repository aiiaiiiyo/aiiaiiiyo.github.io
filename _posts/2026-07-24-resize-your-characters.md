---
title: Resize Your Characters Skeleton for Unreal Using Blender
date: 2026-07-24 23:54:00 +0100
categories: ["Unreal Engine", "Blender"]
tags: [ue, blender, workflow, export, import, skeleton, skeletalmesh, resize]
---

### Foreword

![The Problem](https://github.com/aiiaiiiyo/aiiaiiiyo.github.io/blob/main/assets/img/resizeproblem.png?raw=true)

I had an issue with one of my Skeletal Mesh asset because it was too large. At first, I tried to fix it directly in Unreal Engine. While I managed to solve one problem, it created two others.

I decided to use Blender to resize the character instead. However, I ran into several issues during the export, import, and scaling process.

### Export from Unreal Engine

To export your skeletal mesh, right-click it and select `Asset Actions > Export...`
![Export Menu](https://github.com/aiiaiiiyo/aiiaiiiyo.github.io/blob/main/assets/img/resize_export.png?raw=true)


Under Export Options, choose `FBX 2013` and uncheck all other options.
![Export Options](https://github.com/aiiaiiiyo/aiiaiiiyo.github.io/blob/main/assets/img/resize_export_options.png?raw=true)

### Import into Blender

To import your skeletal mesh into Blender, select `File → Import → FBX (.fbx)`.
![Import Menu](https://github.com/aiiaiiiyo/aiiaiiiyo.github.io/blob/main/assets/img/resize_import.png?raw=true)

No special settings are required here. Leave everything at its default values.

### Resize Your Skeleton in Blender

There is a scale discrepancy between Unreal Engine and Blender. To compensate for this, set the Scene `Unit Scale` to `0.01` in Blender.

![Unit Scale](https://github.com/aiiaiiiyo/aiiaiiiyo.github.io/blob/main/assets/img/resize_unitscale.png?raw=true)

Next, scale your object by 100×, resulting in a scale of `1.000`. This will make the model appear much larger in Blender, but it will have the correct proportions when imported back into Unreal Engine.
![Object Scale](https://github.com/aiiaiiiyo/aiiaiiiyo.github.io/blob/main/assets/img/resize_scale.png?raw=true)

In my case, I wanted to make the character smaller, so I set the `X`, `Y`, and `Z` scale values to `0.770`.
![Object Scale Smaller](https://github.com/aiiaiiiyo/aiiaiiiyo.github.io/blob/main/assets/img/resize_scale2.png?raw=true)

If you plan to continue editing the model, you can apply the scale by pressing `Ctrl + A` and selecting `Scale`.
![Set Scale Base](https://github.com/aiiaiiiyo/aiiaiiiyo.github.io/blob/main/assets/img/resize_scale3.png?raw=true)

### Export From Blender

To export the model from Blender, select `File → Export → FBX (.fbx)`.
![Export Menu](https://github.com/aiiaiiiyo/aiiaiiiyo.github.io/blob/main/assets/img/resize_export_blender.png?raw=true)

There is one important setting to change. Under `Geometry`, set `Smoothing` to `Face`. Otherwise, Unreal Engine will display an error similar to the following during import:

> No smoothing group information was found for this mesh '`NameOfYourMesh`' in the FBX file. Please make sure to enable the 'Export Smoothing Groups' option in the FBX Exporter before exporting the file.
{: .prompt-warning }

![Smoothing](https://github.com/aiiaiiiyo/aiiaiiiyo.github.io/blob/main/assets/img/resize_smoothing.png?raw=true)

Another issue I encountered was that the `Armature` name appeared as the skeleton's root bone, becoming the parent of the actual root bone.
![Root Bone](https://github.com/aiiaiiiyo/aiiaiiiyo.github.io/blob/main/assets/img/resize_armature_name.png?raw=true)

The easiest way to fix this without deleting anything is to rename the `Armature` object in Blender to `armature`. Unreal Engine recognizes this name and imports the skeleton correctly, with the proper root bone hierarchy.
![Root Bone Fix](https://github.com/aiiaiiiyo/aiiaiiiyo.github.io/blob/main/assets/img/resize_armature_name_fix.png?raw=true)

### Final Result

When importing the mesh back into Unreal Engine, leave all import settings at their default values.

The final result can be seen on the right side of the image below.
![Root Bone Fix](https://github.com/aiiaiiiyo/aiiaiiiyo.github.io/blob/main/assets/img/resize_problemfixed.png?raw=true)

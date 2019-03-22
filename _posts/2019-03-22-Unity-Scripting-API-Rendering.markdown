---
layout:     post
title:      "Unity Rendering API OverView"
subtitle:   ""
date:       2018-05-31 14:18:00
author:     "ly"
header-img: "img/post-bg-Unity.jpg"
tags:
    - 翻译
---

> [Unity API Web](https://docs.unity3d.com/ScriptReference)

# **UnityEngine.Rendering**

## *AsyncGPUReadback(class)* && *AsyncGPUReadbackRequest(struct)*
允许异步读取GPU回传的资源
[项目示例](https://github.com/keijiro/AsyncCaptureTest)

## *CommandBuffer(class)*
要执行的图形命令列表 
[官方教程](https://docs.unity3d.com/Manual/GraphicsCommandBuffers.html)

## *GPUFence(struct)*
用于管理异步计算队列和图形队列的任务之间的同步(目前只支持PS4平台)
[SystemInfo.supportsGPUFence](https://docs.unity3d.com/ScriptReference/SystemInfo-supportsGPUFence.html)

## *GraphicsSettings(class)*
修改[GraphicsSettings](https://docs.unity3d.com/Manual/class-GraphicsSettings.html)的接口类

## *PlatformKeywordSet(struct)* && *ShaderKeywordSet(struct)* && *ShaderKeyword(class)*
Shader代码的宏定义设置接口

## *ReflectionProbeBlendInfo(struct)*
包含混合探针所需的信息
[ReflectionProbe](https://docs.unity3d.com/Manual/ReflectionProbes.html)

## *RenderTargetBinding(struct)*(未理解)
描述具有一个或多个颜色缓冲区的渲染目标，深度/模板缓冲区以及渲染目标处于活动状态时应用的关联加载/存储操作。

## *RenderTargetIdentifier(struct)*
在CommandBuffer中标识RenderTexture

## *SortingGroup(class:Behaviour)*
挂在GameObject上使得子节点作为一个Group参加排序

## *SphericalHarmonicsL2(struct)*
二阶球形谐波（3个频段，9个系数），球谐函数标识各方向上的函数或信号，并且通常用于有效的模拟平滑光照的计算。

## *SplashScreen(class)*
 Unity开机图接口类

## *GL(class)*
低级图形库，立即执行，所以通常写在OnPostRender()或者OnRenderImage()函数中，不然会被摄像机渲染时清理掉。[GL API](https://docs.unity3d.com/ScriptReference/GL.html)

## *Graphics(class)*
Unity绘制方法的源生接口，这是优化网格绘制的便捷方法。  
主要方法：
|方法|功能|
-|-|-  
|DrawMesh()| Draw a mesh
|DrawMeshInstanced()| Draw the same mesh multiple times using GPU instancing.
|DrawMeshInstancedIndirect()|Draw the same mesh multiple times using GPU instancing.
|DrawMeshNow()|Draw a mesh immediately.
|DrawProcedural()|Draws a fully procedural geometry on the GPU.
|DrawProceduralIndirect()|Draws a fully procedural geometry on the GPU.
|DrawTexture|Draw a texture in screen coordinates.
|ExecuteCommandBuffer|Execute a command buffer.
|ExecuteCommandBufferAsync|
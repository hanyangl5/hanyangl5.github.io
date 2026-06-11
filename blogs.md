---
permalink: /blogs.html
---

[**Projects**](/projects.md) | [**Blogs**](/blogs.md) | [**Misc**](/misc.md)

---

## [Fresnel](blogs/fresnel/fresnel.md)

This article originates from a frequently asked question during my interview for a graphics programmer position: What is the Fresnel term in microfacet BRDF, and what does it mean in physics?

## [Auto-Res Virtual Shadow Map](blogs/virtualshadowmap/vsm.md)

Steps to implement a virtual shadow map, presenting an idea to avoid readback operations in virtual shadow map rendering.

## [Unreal Opt: Asynchronous Resource Initialization with Auxiliary Render Thread](blogs/auxiliaryrhi/auxiliaryrhi.md)

Discusses techniques for optimizing Unreal Engine by moving resource initialization tasks from the main render thread to auxiliary threads. Offers solutions to minimize game thread stuttering caused by render resource creation.

## [Unreal Tip: Packaging Android Without Android Studio](blogs/UnrealTipPackagingAndroidWithoutAndroidStudio/index.md)

Package Android with Unreal without Android Studio! If you don't want UE to pollute your local Android environment or need to package Android with multiple versions of UE, try this method!

## [Profile GPU Counters without SnapdragonProfiler & Streamline](blogs/gpucounters/gpucounters.md)

Use perfetto to capture and profile gpu counters on Snapdragon & Mali gpu.


## [Async Compute on Mali: Lessons from a Mobile Ray Tracing Experiment](blogs/maliasynccompute/maliasynccompute.md)

A postmortem of an unsuccessful async-compute experiment on Mali GPUs, and what GPU counters revealed about the real parallelism between the Tiler and Execution Engine.

## [Mali Binning Bottleneck Caused by VkCmdDrawIndirect and gl_BaseInstance](blogs/malimdiperf/index.md)

A Bistro benchmark that ran at 150+ FPS on Adreno 840 dropped to around 40 FPS on MTK9500 / Mali, with nearly 20 ms spent in binning. After several failed geometry-side optimizations, the issue turned out to be the combination of `VkCmdDrawIndirect` and `gl_BaseInstance`-based per-draw indexing.
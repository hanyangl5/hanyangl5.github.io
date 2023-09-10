# Auto-res Virtual Shadow Map

Last year, I participated in an R&D project developing a real-time rendering engine where we researched virtual shadow mapping and integrated it into the engine. One limitation we faced was the need to read back tile counts to the CPU for texture allocation, introducing latency. This post presents a method to enable auto-resolution virtual shadow mapping without readbacks.

## Virtual Shadow Map

Virtual shadow mapping is a technique that allows for efficient rendering of shadows by dividing a shadow map into tiles and only rendering those tiles that contribute to the scene. It splits a large shadow map to many tiles and maintains an indirection table to map virtual tiles to physical tiles. Only active tiles contributing to shading are rendered, discarding inactive regions.

Steps of virtual shadow map can be summarized as follows:

### step 1

Split shadow map into uniform virtual tiles

```
// a shadow map in virtual space
+-----+-----+-----+-----+
|     |     |     |     |
+-----+-----+-----+-----+
|     |     |     |     |
+-----+-----+-----+-----+
|     |     |     |     |
+-----+-----+-----+-----+
|     |     |     |     |
+-----+-----+-----+-----+
```

### step 2

Dispatch a full-screen pass to project pixels into shadow space, marking active virtual tiles., only shadow map in these tiles contribute to shading.

```
+-----+-----+-----+-----+
|  x  |     |     |     |
+-----+-----+-----+-----+
|     |  x  |  x  |     |
+-----+-----+-----+-----+
|     |     |  x  |     |
+-----+-----+-----+-----+
|  x  |     |     |     |
+-----+-----+-----+-----+

```

### step 3

Build an indirection table mapping active virtual tiles to physical tiles.

```
+-----+-----+-----+-----+
|  0  |     |     |     |
+-----+-----+-----+-----+
|     |  1  |  2  |     |
+-----+-----+-----+-----+
|     |     |  3  |     |
+-----+-----+-----+-----+
|  4  |     |     |     |
+-----+-----+-----+-----+

```

### step 4

Read back the physical tile count to the CPU for texture allocation.

```

+-----+-----+-----+-----+-----+...
|  0  |  1  |  2  |  3  |  4  |
+-----+-----+-----+-----+-----+
.
.
.
```

### step 5

Apply culling like view frustum/backface/occlusion culling.

### step 6

Rasterize scene geometry, using the indirection table to write shadow depths(manually `InterlockedMax/atomicMax` to write depth).

```
// depth in virtual space
+-----+-----+-----+-----+
|  A  |     |     |     |
+-----+-----+-----+-----+
|     |  B  |  C  |     |
+-----+-----+-----+-----+
|     |     |  D  |     |
+-----+-----+-----+-----+
|  E  |     |     |     |
+-----+-----+-----+-----+

--->

// depth in physical tile space
+-----+-----+-----+-----+-----+...
|  A  |  B  |  C  |  D  |  E  |
+-----+-----+-----+-----+-----+
.
.
.
```

---

## Read back issues

As shown in step 4, the physical tile count must be read back to the CPU to allocate the shadow map texture. This readback synchronizes the CPU and GPU, which is not supported in some game engines or has 1-2 frame latency.

the engine we developed adopted a method similar with [triple buffering](https://developer.apple.com/library/archive/documentation/3DDrawing/Conceptual/MTLBestPracticesGuide/TripleBuffering.html) model to overlap resource upload/download, the difference is our design has 3 frames latency to readback a gpu buffer, which is not accpetable.

![](https://developer.apple.com/library/archive/documentation/3DDrawing/Conceptual/MTLBestPracticesGuide/Art/ResourceManagement_TripleBuffering_2x.png)

figure from [Metal Best Practice]()

## Solution

Inspired by [CWS](https://research.activision.com/publications/2021/10/shadows-of-cold-war--a-scalable-approach-to-shadowing), I came up with a method that don't require a buffer readback.

We can predefine a budget for the whole virtual shadow map system, such as 128MB with a 16384 x 8192 depth texture(1048576 tiles). During required tile counting, if the required tiles exceed the budget, we dynamically scale down the shadow map by 1/2^N and adjust the indirection table. The viewport for rasterization must also be scaled down by 1/2^N accordingly. Multiview rendering with scoped viewports could achieve this efficiently.

## Experiment and Code

TBD

## Reference

- Shadows of Cold War: A Scalable Approach to Shadowing - Kevin Myers
- A Scalable Real-Time Many-Shadowed-Light Rendering System - Bo Li
- A Deep Dive into Nanite Virtualized Geometry - Brian Karis

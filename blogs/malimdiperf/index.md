# Mali Binning Bottleneck Caused by VkCmdDrawIndirect and gl_BaseInstance

Recently we were benchmarking mobile GPU performance with the Bistro scene from `11_final` in [3D Graphics Rendering Cookbook, Second Edition](https://github.com/PacktPublishing/3D-Graphics-Rendering-Cookbook-Second-Edition).

On Adreno 840, the scene ran at **150+ FPS**.

On MTK9500 / Mali, it only reached around **40 FPS**, and the profiler showed something suspicious: the **binning / tiler stage alone was taking around 20 ms**.

At first this looked like a normal geometry bottleneck, but it turned out to be more specific.

## First Attempts

We tried several common optimizations:

- splitting vertex position and attributes into separate buffers
- using a more compact vertex format
- sorting draw calls
- remove other render stages

These helped a little, but not enough. The binning cost on Mali was still much higher than expected.

## First Clue: Empty View, Still Expensive

One important clue came from moving the camera to a place where almost no mesh was visible.

The binning time was still very high even all geometry was culled.

That suggested the problem was not just “too many visible triangles”. Some draw commands were still creating heavy tiler work even when they produced no visible geometry.

Looking at counters, we noticed a high number of:

```text
Empty Tiler Job
```

The sample handled culled objects by keeping their indirect draw commands, but setting:

```cpp
instanceCount = 0;
```

In theory, a draw with zero instances should produce nothing. But on Mali, this was not free. A large number of zero-instance indirect draws still created tiler jobs, which showed up as expensive empty tiler work.

We changed the culling path to actually remove culled draw commands from the indirect command list instead of keeping them with `instanceCount = 0`.

After that, when all meshes were culled, FPS increased from around 40 to **100+ FPS**.

That fixed one issue, but not the main one.

## The Real Problem: VkCmdDrawIndirect + gl_BaseInstance

When no meshes were culled, performance was still bad.

After narrowing it down, the problematic combination was:

```cpp
vkCmdDrawIndirect(...)
```

with shader code using:

```glsl
gl_BaseInstance
```

The original pipeline used `firstInstance` from the indirect command as a per-draw object index. In the shader, object/material/transform data was fetched through `gl_BaseInstance`, roughly like this:

```glsl
uint objectId = gl_BaseInstance;
ObjectData object = objects[objectId];
```

This worked fine on desktop GPUs and Adreno, but on Mali it pushed the binning stage onto a very slow path.

The fix was to change the draw data layout and use:

```glsl
uint objectId = gl_DrawID;
ObjectData object = objects[objectId];
```

After switching from `gl_BaseInstance` to `gl_DrawID`, the binning cost dropped significantly and the frame time became much more reasonable.

## Takeaways

The main lessons from this debugging session:

1. Zero-instance indirect draws are not always free.

   On Mali, keeping many culled draws in the command list with `instanceCount = 0` can still generate a lot of empty tiler work. Compacting the indirect command list and removing culled draws is safer.

2. Be careful with `VkCmdDrawIndirect` and `gl_BaseInstance`.

   Using `gl_BaseInstance` as a per-draw data index worked well on other GPUs, but caused a severe binning bottleneck on Mali in this case. Replacing it with `gl_DrawID` fixed the issue.


This was easy to misread as a generic vertex throughput problem, but the real bottleneck was in the frontend / tiler path caused by a specific indirect draw pattern.

> This issue was reproduced on MTK9500 with driver version 54.1. Other Mali driver versions may behave differently.

Final result: Mali performance improved from around **40 FPS** to a much healthier range, and the binning stage was no longer dominating the frame.
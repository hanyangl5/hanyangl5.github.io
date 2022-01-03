deferred shading

pass1：存gbuffer
pass2：读取gbuffer，计算着色

变种


deferred lighting

pass1：材质信息albedo,metal,roughtness不用gbuffer存储
pass2：计算diff,spec 存到一张Lbuffer中
pass3：从gbuffer，lbuffer中读取信息，计算光照

可以支持多种材质

TBDR, Clustered Deferred Renderin

# ref
https://zhuanlan.zhihu.com/p/102134614
http://www.realtimerendering.com/blog/deferred-lighting-approaches/
http://gameangst.com/?p=141


deferred lighting 和 deferred shading比较

1：
- The attributes stage:
Reads material color textures
Reads material normal maps
Writes depth to a D24S8 target
Writes surface normal and specular exponent to an A8R8G8B8 target
Writes diffuse albedo to an X8R8G8B8 target
Writes specular albedo to an X8R8G8B8 target
Writes emissive to an X8R8G8B8 target
- The deferred Stage:
Reads depth, surface normal, specular exponent, diffuse albedo, and specular albedo
Blends exit radiance additively into an X16R16G16B16 target.

2.

- The attributes stage:
Reads material normal maps
Writes depth to a D24S8 target
Writes surface normal and specular exponent to an A8R8G8B8 target
- The deferred stage:
Reads depth, surface normal, and specular exponent
Blends specular irradiance additively into an X16R16G16B16 target.
Blends diffuse irradiance additively into an X16R16G16B16 target
- The shading stage:
Reads material color textures
Reads diffuse and specular irradiance
Writes exit radiance into an X16R16G16B16 target


比较：

|comp| deferred shading      | deferred lighting ||
|---| ----------- | ----------- |---|
|memory| pass1:20, pass2:8   | pass1:8, pass2:16, pass3:8   |28:32|
|bandwidth|writes 20 bytes per pixel + an additional 8 bytes per lit pixel and reads 24 bytes per lit pixel  |writes 16 bytes per pixel plus an additional 16 bytes per lit pixel and reads 16 bytes per pixel plus an additional 24 bytes per lit pixel         | |

title: GLSL内置变量
date: 2016-5-2 21:11:28
tags: OpenGL GLSL

---
　  GLSL定义了许多内置变量，这些变量一般被用于固定管线的运算，这里主要介绍下各个变量的内容与用法。

# Vertex shader inputs

```
in int gl_VertexID;
in int gl_InstanceID;
```


## gl_VertexID​
因为Vertex Shader对每个Vertex执行一次，所以gl_VertexID是Vertex Shader当前处理的顶点的索引。

gl_InstanceID​
在执行Instance绘制时，gl_InstanceID是当前实例的索引值，当没有进行Instance绘制时它是默认的0值。

<!--more-->

# Vertex shader outputs

```
out gl_PerVertex
{
  vec4  gl_Position;
  float gl_PointSize;
  float gl_ClipDistance[];
};
```

## gl_PerVertex​
Vertex Shader的输出结构，在开启光栅化时会被传输。

## gl_Position
一般是在Vertex Shader经过Model View Proj矩阵变换过的Vertex数据，此时数据位于clip-space， 数据是齐次坐标[x, y, z, w].

## gl_PointSize​
绘制像素点大小，在绘制point primitives时有效。

## gl_ClipDistance​
同于用户自定义的裁剪计算，非负值说明顶点在裁剪面内，负值则被裁剪。
---

# Tessellation Control Shaders input:

```
 in int gl_PatchVerticesIn;
 in int gl_PrimitiveID;
 in int gl_InvocationID;
 ```

## gl_PatchVerticesIn​
每个patch多少个vertices

## gl_PrimitiveID​
当前patch索引值

## gl_InvocationID​
当前patch中vertices的索引值。


Tessellation control shader接受vertex shader的输出作为输入，需要注意的是，由于shader对每个patch执行一次，所以该结构是数组模式。

 ```
in gl_PerVertex
{
  vec4 gl_Position;
  float gl_PointSize;
  float gl_ClipDistance[];
} gl_in[gl_MaxPatchVertices];

 ```
结构内的变量含义在vertex shader那里已经说过了，gl_MaxPatchVertices是patch的顶点数，注意跟gl_PatchVerticesIn的区别。


# Tessellation Control Shaders output:

 ```
patch out float gl_TessLevelOuter[4];
patch out float gl_TessLevelInner[2];
 ```
 这个两个分量主要分别设置了patch的内外细分度

Tessellation Control Shader需要将vertex shader传入的数据继续传输下去，所以有类似的输出的结构体：

 ```
out gl_PerVertex
{
  vec4 gl_Position;
  float gl_PointSize;
  float gl_ClipDistance[];
} gl_out[];
 ```


# Tessellation evaluation shader inputs

 ```
in vec3 gl_TessCoord;
in int gl_PatchVerticesIn;
in int gl_PrimitiveID;
 ```

gl_TessCoord​
细分坐标，利用插值可以产生新的细分顶点

gl_PatchVerticesIn​
与control中类似

gl_PrimitiveID​
当前patch的索引值

patch in float gl_TessLevelOuter[4];
patch in float gl_TessLevelInner[2];

```
in gl_PerVertex
{
 vec4 gl_Position;
 float gl_PointSize;
 float gl_ClipDistance[];
} gl_in[gl_MaxPatchVertices];
```

# Tessellation evaluation shader outputs

```
out gl_PerVertex {
  vec4 gl_Position;
  float gl_PointSize;
  float gl_ClipDistance[];
};
```

gl_Position​
产生细分数据的clip-space坐标值

gl_PointSize​
像素点大小

gl_ClipDistance​
自定义裁剪变量，前面已说明

---

# Geometry shader inputs

```
in gl_PerVertex
{
  vec4 gl_Position;
  float gl_PointSize;
  float gl_ClipDistance[];
} gl_in[];
```

```
in int gl_PrimitiveIDIn;
in int gl_InvocationID;  //Requires GLSL 4.0 or ARB_gpu_shader5
```

gl_PrimitiveIDIn​

gl_InvocationID​

# Geometry shader outputs

```
out gl_PerVertex
{
  vec4 gl_Position;
  float gl_PointSize;
  float gl_ClipDistance[];
};
```

out int gl_PrimitiveID;
gl_PrimitiveID = gl_PrimitiveIDIn;


层次渲染通过下面的变量实现：

```
out int gl_Layer;
out int gl_ViewportIndex; //Requires GL 4.1 or ARB_viewport_array.
```

---

# Fragment Shaders input.

```
in vec4 gl_FragCoord;
in bool gl_FrontFacing;
in vec2 gl_PointCoord;
```

## gl_FragCoord​
fragment在window space的位置量， [x,y,z]其中z值会被写入到gl_FragDepth中，而w值等于 1 /w(clip), w(clip)是插值后的gl_Position
的w值。gl_FragCoord的原点可以通过下面命令改变，默认原点在bottom-left：
layout(origin_upper_left) in vec4 gl_FragCoord;

## gl_FrontFacing​
用于判断是否处于正面，如果是back-face，结果为负。

## gl_PointCoord​
Point primitive在的位置，范围[0, 1]，原点在upper-left,可以用来实现Spirtes，可以通过下面命令修改原点位置：
glPointParameteri(GL_POINT_SPRITE_COORD_ORIGIN, GL_LOWER_LEFT);​

OpenGL 4.0增加了几个系统产生的输入变量：

```
in int gl_SampleID;
in vec2 gl_SamplePosition;
in int gl_SampleMaskIn[];
```

## gl_SampleID​
整数值代表当前fragment的sample标志。

##gl_SamplePosition​
当前sample在fragment的位置，[0, 1]。

## gl_SampleMaskIn​
用于多重采样

```
in float gl_ClipDistance[];
in int gl_PrimitiveID;
```

## gl_ClipDistance​
经过插值过的裁剪变量。

## gl_PrimitiveID​
当前图元索引值

OpenGL 4.3增加了下面两个内置输入变量：

```
in int gl_Layer;
in int gl_ViewportIndex;
```

## gl_Layer​
当前layer索引值

## gl_ViewportIndex​
当前viewport索引值

# Fragment shader outputs

```
out float gl_FragDepth;
out in gl_SampleMask[];
```

## gl_FragDepth​
一般为gl_FragCoord;

---

# Compute Shaders input.

```
in uvec3 gl_NumWorkGroups;
in uvec3 gl_WorkGroupID;
in uvec3 gl_LocalInvocationID;
in uvec3 gl_GlobalInvocationID;
in uint  gl_LocalInvocationIndex;
```

## gl_NumWorkGroups​
工作组总的数量

## gl_WorkGroupID​
当前工作组索引值

## gl_LocalInvocationID​
工作组内本地工作组的索引值，XYZ处于[0,gl_NumWorkGroups.XYZ]。

## gl_GlobalInvocationID​
全局的实例索引值，等于下面的值：
gl_WorkGroupID * gl_WorkGroupSize + gl_LocalInvocationID;

## gl_LocalInvocationIndex​
本地工作组内索引值：
```
  gl_LocalInvocationIndex =
          gl_LocalInvocationID.z * gl_WorkGroupSize.x * gl_WorkGroupSize.y +
          gl_LocalInvocationID.y * gl_WorkGroupSize.x +
          gl_LocalInvocationID.x;
```

# Compute shader other variables
const uvec3 gl_WorkGroupSize;   // GLSL ≥ 4.30

# Shaders  uniforms

```
struct gl_DepthRangeParameters
{
    float near;
    float far;
    float diff;
};
uniform gl_DepthRangeParameters gl_DepthRange;

uniform int gl_NumSamples; //GLSL 4.20
```

详细介绍[GLSL Build-In Variable](https://www.opengl.org/wiki/Built-in_Variable_(GLSL)#Fragment_shader_outputs)

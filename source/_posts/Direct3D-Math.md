title: Direct3D-Math
date: 2015-12-20 22:48:31
tags:
    - Direct3D
    - Math

---
# Introduction
　　本文主要介绍Direct3D开发中使用的数学运算库。


# Content

## D3DXMath
　　这是Direct3D旧版本的数学运算库，主要有d3dx9math, d3dx10math.

## xnamath
　　在DirectX11的SDK中包含了一个新的数学库xnamath，它主要是为XNA平台开发编写的，采用了SIMD提供运算效率，在PC程序开发中也可以使用。

## DirectXMath
　　在Windows8.x之后DirectX SDK不再以单一组件形式提供，D3DXMath也已经被抛弃，微软推荐采用新使用C++编写的DirectXMath数学库，它可以看做xnamath的升级版本。

<!--more-->

```
typedef _m128 XMVECTOR
```
为了能充分利用SIMD的效率，在使用时可以采用以下步骤进行操作：
- 局部或全局变使用XMVECTOR， XMMATRIX
- 类的数据成员使用XMFLOAT*， XMFLOAT*x*
- 在进行运算前，使用相应的loading函数将XMFLOAT* / XMFLOAT*x*转换为XMVECTOR/ XMMATRIX
- 使用XMVECTOR / XMMATRIX进行运算
- 采用相应的storage函数将结果从XMVECTOR/XMMATRIX转换回XMFLOAT* / XMFLOAT*x*

```
struct XMMATRIX;

// Fix-up for (1st) XMMATRIX parameter to pass in-register for ARM64 and vector call; by reference otherwise
#if ( defined(_M_ARM64) || _XM_VECTORCALL_ ) && !defined(_XM_NO_INTRINSICS_)
typedef const XMMATRIX FXMMATRIX;
#else
typedef const XMMATRIX& FXMMATRIX;
#endif

// Fix-up for (2nd+) XMMATRIX parameters to pass by reference
typedef const XMMATRIX& CXMMATRIX;

#ifdef _XM_NO_INTRINSICS_
struct XMMATRIX
#else
__declspec(align(16)) struct XMMATRIX
#endif
{
#ifdef _XM_NO_INTRINSICS_
    union
    {
        XMVECTOR r[4];
        struct
        {
            float _11, _12, _13, _14;
            float _21, _22, _23, _24;
            float _31, _32, _33, _34;
            float _41, _42, _43, _44;
        };
        float m[4][4];
    };
#else
    XMVECTOR r[4];
#endif

```

```
XMVECTOR XMLoadFloat4(COSNT XMFLOAT4 *pSource);
VOID     XMStoreFloat4(XMFLOAT4 *pDestination, FXMVECTOR V);
FLOAT XMVectorGetX(FXMVECTOR V);
XMVECTOR XMVectorSetX(FXMVECTOR V, FLOAT x);

```

```
const float XM_PI           = 3.141592654f;

inline float XMConvertToRadians(float fDegrees) { return fDegrees * (XM_PI / 180.0f); }
inline float XMConvertToDegrees(float fRadians) { return fRadians * (180.0f / XM_PI); }
```

可以参考MSDN博客上的这篇文章[Introducing DirectXMath](http://blogs.msdn.com/b/chuckw/archive/2012/03/27/introducing-directxmath.aspx)
#

---
title: 使用NShader在VS中语法高亮GLSL，HLSL等Shader文件
date: 2016-05-12 09:13:49
tags:
  - Shader

---
# 介绍
[NShader](http://nshader.codeplex.com/)是一款VS中Shader语法高亮的插件，主要支持下面几种高亮：

```
.fx, .fxh, .hlsl, .vsh, .psh files         : HLSL syntax highlighter.
.glsl, .frag, .vert, .fp, .vp, .geom files : GLSL syntax highlighter.
.cg, .cgfx, files                          : CG syntax highlighter.
```

# 使用

在官网中下载Nshader然后安装。打开注册表，删除下面的键值：
```
HKEY_CURRENT_USER\Software\Microsoft\VisualStudio\11.0\FontAndColors\Cache\ HKEY_CURRENT_USER\Software\Microsoft\VisualStudio\11.0_Config
```

<!--more-->

VS2012需要删除下面文件内所有fx，hlsl的行，这样VS默认的hlsl语法高亮不会影响NShader的语法高亮：
```
 C:\Program Files (x86)\Microsoft Visual Studio 11.0\Common7\IDE\CommonExtensions\Microsoft\VC\VC_Pkg_Core_Registration.pkgdef
```

如果需要配置高亮方案，可以在VS（Tools-Options-Environment-Fonts and Colors)右侧的Display items中设置带有NShder开头的项,这样就可以分别设置语法高亮方案。

# 扩展
由于官网上Nshader只支持VS2012，下面我主要讲下怎么在在VS2013/VS2015中使用NShader实现Shader的语法高亮，并说下怎么添加自定义文件后缀进行语法高亮。在我github中下[nshader](https://github.com/byhj/nshader)的源码，然后使用VS2013/VS2015进行编译。找到生成NShader.vsix进行安装。你可以通过修改NShader.pkgdef文件来支持你自定义后缀的Shader文件进行语法高亮。NShader.pkgdef文件在：

```
C:\User(用户)\你当前的用户\AppData\Local\Microsoft\VisualStudio\12.0(VS2015是14.0)\Extensions\
然后直接搜索NShader.pkgdef文件
```
打开NShader.pkgdef文件然后安装里面的案例进行修改就可以了，下面给出我的自定义方案:
```
OpenGL:
- .vert : GL_VERTEX_SHADER
- .tcs  : GL_TESS_CONTROL_SHADER
- .tes  : GL_TESS_EVALUATION_SHADER
- .geom : GL_GEOMETRY_SHADER
- .farg : GL_FRAGMENT_SHADER
- .comp : GL_COMPUTE_SHADER

Direct3D:
- .vsh: D3D_VERTEX_SHADER
- .hsh: D3D_HULL_SHADER
- .dsh: D3D_DOMAIN_SHADER
- .gsh: D3D_GEOMETRY_SHADER
- .psh: D3D_PIXEL_SHADER
- .csh: D3D_COMPUTE_SHADER

```
Nshader.pkgdef:
```
[$RootKey$\InstalledProducts\NShader]
@="#110"
"Package"="{dd9b8cde-96a0-4442-bf6f-083a7f43d453}"
"PID"="1.54"
"ProductDetails"="#112"
"LogoID"="#400"
[$RootKey$\Packages\{dd9b8cde-96a0-4442-bf6f-083a7f43d453}]
@="NShader"
"InprocServer32"="$WinDir$\SYSTEM32\MSCOREE.DLL"
"Class"="NShader.NShader"
"CodeBase"="$PackageFolder$\NShader.dll"
[$RootKey$\Languages\File Extensions\.cg]
@="{20d6b815-3b2a-303d-bb0a-d13bd201825a}"
[$RootKey$\Languages\File Extensions\.cgfx]
@="{20d6b815-3b2a-303d-bb0a-d13bd201825a}"
[$RootKey$\Languages\File Extensions\.cginc]
@="{20d6b815-3b2a-303d-bb0a-d13bd201825a}"
[$RootKey$\Languages\File Extensions\.comp]
@="{20d6b815-3b2a-303d-bb0a-d13bd201825a}"
[$RootKey$\Languages\File Extensions\.fp]
@="{20d6b815-3b2a-303d-bb0a-d13bd201825a}"
[$RootKey$\Languages\File Extensions\.frag]
@="{20d6b815-3b2a-303d-bb0a-d13bd201825a}"
[$RootKey$\Languages\File Extensions\.fsh]
@="{20d6b815-3b2a-303d-bb0a-d13bd201825a}"
[$RootKey$\Languages\File Extensions\.fx]
@="{20d6b815-3b2a-303d-bb0a-d13bd201825a}"
[$RootKey$\Languages\File Extensions\.fxh]
@="{20d6b815-3b2a-303d-bb0a-d13bd201825a}"
[$RootKey$\Languages\File Extensions\.geom]
@="{20d6b815-3b2a-303d-bb0a-d13bd201825a}"
[$RootKey$\Languages\File Extensions\.glsl]
@="{20d6b815-3b2a-303d-bb0a-d13bd201825a}"
[$RootKey$\Languages\File Extensions\.hlsl]
@="{20d6b815-3b2a-303d-bb0a-d13bd201825a}"
[$RootKey$\Languages\File Extensions\.psh]
@="{20d6b815-3b2a-303d-bb0a-d13bd201825a}"
[$RootKey$\Languages\File Extensions\.shader]
@="{20d6b815-3b2a-303d-bb0a-d13bd201825a}"
[$RootKey$\Languages\File Extensions\.slfx]
@="{20d6b815-3b2a-303d-bb0a-d13bd201825a}"
[$RootKey$\Languages\File Extensions\.vert]
@="{20d6b815-3b2a-303d-bb0a-d13bd201825a}"
[$RootKey$\Languages\File Extensions\.tcs]
@="{20d6b815-3b2a-303d-bb0a-d13bd201825a}"
[$RootKey$\Languages\File Extensions\.tes]
@="{20d6b815-3b2a-303d-bb0a-d13bd201825a}"
[$RootKey$\Languages\File Extensions\.vp]
@="{20d6b815-3b2a-303d-bb0a-d13bd201825a}"
[$RootKey$\Languages\File Extensions\.vsh]
@="{20d6b815-3b2a-303d-bb0a-d13bd201825a}"
[$RootKey$\Languages\File Extensions\.hsh]
@="{20d6b815-3b2a-303d-bb0a-d13bd201825a}"
[$RootKey$\Languages\File Extensions\.dsh]
@="{20d6b815-3b2a-303d-bb0a-d13bd201825a}"
[$RootKey$\Languages\File Extensions\.tsh]
@="{20d6b815-3b2a-303d-bb0a-d13bd201825a}"
[$RootKey$\Languages\File Extensions\.gsh]
@="{20d6b815-3b2a-303d-bb0a-d13bd201825a}"
[$RootKey$\Languages\File Extensions\.csh]
@="{20d6b815-3b2a-303d-bb0a-d13bd201825a}"
[$RootKey$\Languages\File Extensions\.xsh]
@="{20d6b815-3b2a-303d-bb0a-d13bd201825a}"
[$RootKey$\Languages\Language Services\NShader]
@="{20d6b815-3b2a-303d-bb0a-d13bd201825a}"
"Package"="{dd9b8cde-96a0-4442-bf6f-083a7f43d453}"
"LangResID"=dword:00000072
"RequestStockColors"=dword:00000000
"EnableFormatSelection"=dword:00000001
"EnableCommenting"=dword:00000001
"EnableLineNumbersOption"=dword:00000001
[$RootKey$\Services\{20d6b815-3b2a-303d-bb0a-d13bd201825a}]
@="{dd9b8cde-96a0-4442-bf6f-083a7f43d453}"
"Name"="Shader Language Service"
```

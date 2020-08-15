---
layout: post
title:  "OpenGL学习笔记一 安装配置"
date:   2020-07-11 06:00:00
categories: OpenGL
tags: OpenGL, C++
excerpt_separator: <!--more-->
---

* content
{:toc}

学习材料主要是[Learn OpenGL](https://github.com/JoeyDeVries/LearnOpenGL).

本篇主要介绍 OpenGL 的相关基础，GLFW 和 GLAD 的安装和配置。
<!--more-->

## OpenGL相关基础
* OpenGL 是由[khronos]（https://www.khronos.org）维护的相关规范，不是 API。
* Core-profile 模式和 Immediate mode， 现代 OpenGL 推荐使用 core-profile 模式。两者主要区别是前者会更了解 OpenGL 实际是如何操作的， 而后者可能更容易学习，教程里主要是 Core-profile 模式。3.3 版本以后支持 core-profile 模式。
* Graphics card 可能会有拓展（extensions），暂时没被 OpenGL 包含。在使用时应该判断这些 extensions 是否可获得。
* State machine 通常指 OpenGL context，可以通过 setting options, manipulating buffer 和 render current context 来改变。通过会使用到两类函数 state-changing functions 和 state-using functions。
* Objects 通常是代表着 OpenGL 的状态的选项的集合。工作流程很多时候会通过对象生成（指定 Id）, 绑定对象，操作，解绑对象来完成的。教程后续主要工作方式也是如此，示例如下：
```cpp
// create object
unsigned int objectId = 0;
glGenObject(1, &objectId);
// bind/assign object to context
glBindObject(GL_WINDOW_TARGET, objectId);
// set options of object currently bound to GL_WINDOW_TARGET
glSetObjectOption(GL_WINDOW_TARGET, GL_OPTION_WINDOW_WIDTH, 800);
glSetObjectOption(GL_WINDOW_TARGET, GL_OPTION_WINDOW_HEIGHT, 600);
// set context target back to default
glBindObject(GL_WINDOW_TARGET, 0);
```

## 安装GLFW及相关运行配置
* 下载并 Building GLFW
   * 下载地址。[Cmake 地址](https://www.cmake.org/cmake/resources/software.html)，[GLFW 下载地址](https://www.glfw.org/download.html).此处均配置为 Visual Studio2019 和 64 位版本；
   * CMake 编译。source code 选择~/glfw-3.3~,builder folder 选择新建的 build 文件夹->Configure->project 选择 Visual Studio 16 2019->finish->两次 Configure 确认->generate；
   * Compilation。 在 VS 中选择**生成解决方案**，不是运行；
   * 将~/glfw-3.3~/include 下。h 文件拷贝入自建的~/OpenGL/include 内，将~/glfw-3.3~/build/src/Debug/glfw3.lib 拷贝入自建的~/OpenGL/lib 文件夹内，待后续在 VS 内指定。
* 新建 VS 项目
   * VS 选择生成版本选择为 x64；
   * 指定包含目录~/include，指定库目录~/lib，指定链接附加依赖项 glfw.lib。
   * 包含了 opengl32.lib 库
   * 可以些包含库 `#inlcude <glfw3.h>`
* GLAD
   * OpenGL 的函数使用需要 retrieve the location of the functions and store them in function pointers for later use.
   * [下载 GLAD](https://glad.dav1d.de/)，包括两个文件夹一个 `.c` 文件；
   * 存放库文件的文件夹放入 `~/OpenGL/include`，`.c` 文件放入工程文件中。

至此安装配置工作基本完成。

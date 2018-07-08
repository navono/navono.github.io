---
title: Windows使用CLion
date: 2018-07-08 10:03:06
categories: [笔记]
tags: [杂项]
---

# 安装
1. CLion
从[官网](https://www.jetbrains.com/)下载安装

2. Linux环境（msys2）
从[sourceforge](https://sourceforge.net/projects/msys2/)上下载，安装，安装好后运行以下命令更新：
> pacman -Syu

更新完后，安装工具链：
> pacman -S mingw-w64-x86_64-toolchain

安装完成后此时基本就可以使用，但是如果要使用`LLVM`的话，还得安装：
> pacman -S mingw-w64-x86_64-llvm mingw-w64-x86_64-clang

全部安装完成后，环境也就基本搭建好了。

# 配置
在`CLion`的`Settings`界面的`Build, Execution, Deployment`中找到`Toolchains`。新建一个运行环境，选择`MinGW`，选择通过`msys2`安装好的路径，此时后续的`CMake`等会自动检测。如果使用默认的`GCC`工具链，那么此时配置就算完成了。但是想要使用`LLVM`，还得在`CMake`选项中进行配置，主要配置的选项是：

>CMake options: -D_CMAKE_TOOLCHAIN_PREFIX=llvm-<br>
>Environment: CC=path//to//clang.exe;CXX=path//to//clang++.exe

# 可能遇到的问题
1. `xxx is not able to compile a simple test program.`
这有可能是权限问题，也有可能是缓存问题。可以尝试将`C:\\Users\\xxx\\.CLion`清空，然后重新配置`CLion`。

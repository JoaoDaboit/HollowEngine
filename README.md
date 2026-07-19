# HollowEngine 0.3

![C++](https://img.shields.io/badge/language-C++-blue.svg)
![SDL2](https://img.shields.io/badge/library-SDL2-orange.svg)
![ImGui](https://img.shields.io/badge/UI-ImGui-brightgreen.svg)
![Status](https://img.shields.io/badge/status-beta-yellow.svg)
![Platform](https://img.shields.io/badge/platform-Linux-lightgrey.svg)

---

## 技术文档 (Technical Documentation)

### 项目简介
HollowEngine 是一个使用 C++、SDL2 和 ImGui（cimgui）开发的实验性 3D 游戏引擎。本项目从零开始开发，旨在深入学习计算机图形学、3D 数学、游戏引擎架构以及编辑器开发。

### 构建说明 (Build Instructions)
本项目目前针对 Linux 环境优化。
1. **安装依赖**:
   ```bash
   sudo apt update
   sudo apt install build-essential pkg-config libsdl2-dev libsdl2-image-dev

2. **编译**:
     ```bash
   g++ -g $(pkg-config --cflags sdl2 SDL2_image) -Iinclude -Iinclude/imgui Engine.cpp include/imgui/imgui.cpp include/imgui/imgui_draw.cpp include/imgui/imgui_tables.cpp include/imgui/imgui_widgets.cpp include/imgui/backends/imgui_impl_sdl2.cpp include/imgui/backends/imgui_impl_sdlrenderer2.cpp -o Engine $(pkg-config --libs sdl2 SDL2_image)
     
3. **运行**:
      ```bash
      ./Engine

功能特点
编辑器: 实时 3D 视口、ImGui 用户界面、属性检查器。
对象操作: 点击选择、移动、旋转、缩放 Gizmo。
材质系统: 支持 Albedo/Normal/Roughness 贴图。
光照: 方向光与基础漫反射照明。


   


## English Documentation

## Project Overview
HollowEngine is an experimental 3D game engine developed from scratch in C++, utilizing SDL2 and cimgui (Dear ImGui). It is a learning project focused on computer graphics, 3D math, and engine architecture.
Build Instructions
This project is currently optimized for Linux systems.

1. **Install Dependencies**:
   ```bash
   sudo apt update
   sudo apt install build-essential pkg-config libsdl2-dev libsdl2-image-dev

2. **Compile**:
     ```bash
   g++ -g $(pkg-config --cflags sdl2 SDL2_image) -Iinclude -Iinclude/imgui Engine.cpp include/imgui/imgui.cpp include/imgui/imgui_draw.cpp include/imgui/imgui_tables.cpp include/imgui/imgui_widgets.cpp include/imgui/backends/imgui_impl_sdl2.cpp include/imgui/backends/imgui_impl_sdlrenderer2.cpp -o Engine $(pkg-config --libs sdl2 SDL2_image)
     
3. **Run**:
      ```bash
      ./Engine

## Key Features
Editor: Real-time 3D viewport, Inspector, and Object creation panel.
Tools: Object selection, transformation Gizmos (Move, Rotate, Scale), and live transform editing.
Materials: Albedo/Normal/Roughness map support with per-object material management.
Lighting: Directional light system with diffuse shading.
Camera: Professional-style free-movement with mouse control.
Developer: João Paulo


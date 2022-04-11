---
title: blender入门一：界面UI和基本操作
date: 2022-04-11 18:51:12
tags: blender
---

第一节课的成果：

![Snipaste_2022-04-10_23-00-06](E:\画画\Snipaste_2022-04-10_23-00-06.jpg)

### 一、视角调整

- 鼠标中键（滚轮）：旋转画面
- shift+中键：移动画面

- ctrl+中键：缩放
  - 默认上下滑动控制
  - 修改：Edit->Preference->Navigation->Zoom Axis:Vertival改为Horization
- Alt+中键：吸附到固定角度

建议关掉Auto Perspective

- 右上角gizmo（小装置）点击X/-X/Y/-Y/Z/-Z轴切换视角
- 数字键5/gizmo最后一个按钮：切换为透视/正投影视角

### 二、选择元素

- 鼠标点击选择元素，按住shift多选

- 左侧工具栏长按Tweak按钮弹出选择菜单
  - box select/B键：用一个box选中多个
  - circle select:圈选

  - lasso select:套索


### 三、修改元素基本属性

- G键：移动
  - 移动过程中按住X键，吸附在X轴移动（YZ同理）
  - 按住中键可以选择吸附方向
  - alt+G移动清零

- R键：rotate旋转

  - 按住ctrl每5度移动一格

  - 按住shift精确移动
  - alt+R旋转清零

- S键：Scale缩放
  - alt+S大小清零

- 右侧工具栏

  - Item->transform：查看、修改移动、旋转和缩放数据

  - View ->lock to 3D cursor围绕固定点缩放旋转

### 五、新建模型

- shift+A键/上方菜单add->mesh：新建物体

  - 改变基础模型属性：展开add菜单


- 自动平滑：右侧Object Data Properties->Normals开启Auto Smooth，右键将Shading Flat修改为Shading Smooth
- 新建object时保存参数设置，下次使用：Operator Presets -> + 
- X键：删除物体

### 六、切换透明/不透明模式

- Shift+Z键/上方工具栏Viewport Shading：切换透明模式


- toggle x-RAY是否显示重叠

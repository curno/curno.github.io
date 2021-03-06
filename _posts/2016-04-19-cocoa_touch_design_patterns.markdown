---
layout: post
title: "Cocoa Touch中的设计模式"
description: ""
tags: [iOS, Design]
---

今天想讨论下Cocoa Touch这个`iOS`开发的最基本系统库。作为`iOS`开发者每天日常工作的基础，我们对它再熟悉不过了。在它纷繁的接口背后，是苹果开发工程师的设计思想和考量权衡。本文从设计模式的角度出发，总结了在Cocoa Touch的接口设计中存在的那些设计模式应用。了解如何这些设计模式和思考为什么要应用设计模式能够帮助我们理解苹果开发工程师的想法，一来可以促使自己更好的使用系统库，不掉入系统库的某些坑；二来也可以增强自己的设计直觉，在类似的问题上效仿借鉴。顺便通过这次总结再回顾下设计模式。

#### 写在前面的题外话

本文所说的设计模式，主要指的是*GoF*所著的经典著作中的设计模式。一般认为在OO的开发环境中，这些设计模式最易应用。有些人认为OO倾向于导致过于复杂的设计，很多时候设计模式的应用产生过度设计的后果，增加软件复杂度。我同意这个看法，但是在`iOS`开发领域，或者更广的客户端/GUI开发领域，OO仍然是效率最高的编程方法，也是苹果开发工程师主张的开发方法。在进行OO开发时，要利用设计模式产生良好设计，又要警惕“过于依赖”、“强行套用”设计模式带来的过度设计问题。只有不断积累实践经验，汲取大师之法，才可以用好OO这把双刃剑。


下面的总结按照设计模式逐条说明，不分先后。

<!-- brief-remark -->


#### 单例模式

`iOS`系统管理着很多软硬件资源。而最普遍的，管理这些资源的接口，使用了单例模式。如：

* UIScreen 管理主屏幕
* UIDevice 管理设备信息
* CMMotionManager 管理设备Motion
* ...

类似的例子还有很多。不难看出，由于硬件的唯一性，接口设计成单例模式是自然而然的选择。

#### 组合模式

几乎所有系统的视觉组件库都将视觉组件类建模为组合模式，这也是由于视觉组件的排布为树形结构是最方便的。如：

* UIView/subViews
* CALayer/subLayers
* UIViewController/childViewControllers

T'B'C

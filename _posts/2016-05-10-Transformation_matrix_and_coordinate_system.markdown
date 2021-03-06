---
layout: post
title: "矩阵变换和坐标系"
description: ""
tags: [mathematics, theory, graphics]
---

最近抽空尝试了MAC OS X下神奇效果的实现，基本的思想是将`UIView`截图之后，对渲染区域计算三次贝塞尔曲线的三角形逼近，使用`OpenGL`进行三角形渲染。

在使用`OpenGL`进行渲染时，为了接口的简洁直观，需要将`UIView`的屏幕坐标系转换为`OpenGL`的视图坐标系。坐标系的转换是一个基本操作，但之前我进行类似操作的时候总是不够熟练，试错几次之后才能转换成功，原因在于自己之前一直没有总结整理转换时的要点。因此，趁着这次又重新经历了这样的过程，彻底整理一下相关的要点，希望以后就不要在坐标转换上出问题。

<!-- brief-remark -->

#### 二维 & 三维

在进行绘图或者坐标变换的时候，往往使用的是齐次坐标，因为其次坐标可以表达无穷远点。二维的齐次坐标为三元坐标，变换矩阵为3x3矩阵；三维的齐次坐标为四元坐标，变换矩阵为4x4矩阵。不失一般性，这里只讨论三维的情况。

#### 变换矩阵

基本的变换矩阵有三种：平移、旋转和放缩。作为基本的API，大部分图形编程接口都提供了这三种变换矩阵的创建接口。对于每一种变换矩阵的具体构成，可以参考[这里](https://en.wikipedia.org/wiki/Transformation_matrix)

##### 变换矩阵的左乘和右乘

变换矩阵的左乘和右乘决定了齐次坐标是通过何种方式进行变换。变换矩阵如果是左乘，则齐次坐标为列坐标；否则为行坐标。左乘和右乘在我看来只是一个习惯问题。在`OpenGL`和`iOS UIKit`等库中，变换矩阵都是左乘的。

    设M为变换矩阵，P为齐次坐标，则变换后的齐次坐标为M * P

#### 多次变换

一个齐次坐标可以进行多次变换，这种情况下可以将最终的变换矩阵表示出分步变换的变换矩阵相乘的形式:

    P' = M1 * M2 * P

对于多次变换，矩阵相乘的顺序是重要的，比如说一个点首先平移后旋转，与先旋转后平移，最终的位置很有可能是不同的，（从数学的角度讲，矩阵相乘没有交换律）。根据使用的编程接口的定义，有时候可以预先创建出相乘的变换矩阵，后对点进行变换（从数学的角度讲，矩阵相乘满足结合律），例如对于上述的式子，可以通过以下方式计算最终的变换矩阵：

    M = M2
    M = transform M2 with M1

对于矩阵相乘，可以有两种理解方法，这两种方式可以叫做**点变换理解**和**坐标系变换理解**，它们具有不同的思维模型，但是可以得到相同的结果。

##### 点变换理解

点变换理解指的是将矩阵部分看做是对**将要和它相乘的点上附加的变换**。在这种理解下，每一个应用变换的店通过**从右往左**的若干变换，变换到最终位置，在上面的例子中，点首先进行M2变换，然后进行M1变换。例如，如果我们希望对每一个点首先放大`2`倍，然后平移`(100, 100, 0)`，则可以这样构造变换矩阵：

    GLKMatrix4 projectionMatrix = GLKMatrix4Identity;
    projectionMatrix = GLKMatrix4Translate(projectionMatrix, 100, 100, 0);
    projectionMatrix = GLKMatrixScale(projectionMatrix, 2, 2, 2);

可以看到，虽然概念上是先放大，后平移，但是这种理解下矩阵从右往左进行应用，则构造矩阵的时候先创建平移矩阵，再右乘放大矩阵。


##### 坐标系变换理解

坐标系变换理解将矩阵部分看做是**对坐标系的变换**。对于一个点变换，也可以理解成坐标系发生了变化。比如上面这个例子，可以理解为点进行了变换到达新的位置，也可以理解为点所在的坐标系发生了变化，由于坐标系的变化导致了点坐标的变化。在这样的理解下，坐标系通过**从左往右**的顺序进行了若干变换。在上面的例子中，可以理解为坐标系首先平移了`(100, 100, 0)`，然后放大`2`倍，对于某一个点，相当于点先扩大后平移。因为坐标系变换从左往右进行变换，因此得到的变换矩阵是相同的。

#### 将`UIView`坐标系变换到`OpenGL View Port`坐标系

在实际应用中，比较常用的变换就是将`OpenGL View Port`坐标系转换到`UIView`的坐标系了。`UIView`坐标系以左上角为原点，`x`向右，`y`向下，坐标精度为`UIView`大小，定为`(w, h)`；`OpenGL View Port`为单位坐标系，以中心为原点，`x`向右，`y`向上，坐标精度从-1到1，为`(2, 2)`。进行坐标系变换时，可以先翻转`y`轴，然后平移到左上角，然后缩放坐标精度。按照这样的坐标系变换理解，变换矩阵就可以构造出来了：

    GLKMatrix4 projectionMatrix = GLKMatrix4Identity; // 单位矩阵
    projectionMatrix = GLKMatrix4Scale(projectionMatrix, 1.0, -1.0, 1.0); // 翻转 y
    projectionMatrix = GLKMatrix4Translate(projectionMatrix, -1.0, -1.0, 0.0); // 平移到左上
    projectionMatrix = GLKMatrix4Scale(projectionMatrix, 2 / w, 2 / h, 1.0); // 缩放坐标系精度

经过这样的变换，就可以在接口中使用熟悉的`UIView`坐标系中的坐标了。

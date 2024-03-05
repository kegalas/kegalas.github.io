---
title: 从零开始的软渲染器 模型库
date: 2023-06-25T14:37:37+08:00
draft: false
tags:
  - 图形学
  - 渲染
  - 干货
categories: 图形学
mathjax: true
markup: goldmark
image: cover.jpg
---

<u>**[导航页面](../从零开始的软渲染器-导航/)**</u>


# OBJ格式简介

.obj格式是描述几何模型的一种简单办法。它可以描述顶点的坐标、法向量、纹理坐标，以及三角面的顶点数据等等。

**注释**

.obj格式里的注释是以#开头的一行

**顶点坐标**

例如

```
v 0.511631 -0.845357 0.127832
v 0.608654 -0.568839 -0.416318
v 0.424663 -0.649937 -0.567418
...
```

以v开头，后面带三个小数。注意这个坐标是模型相对于自己原点的坐标，而不是我们渲染过程中的世界坐标。它不会提供总数，只会一条一条地提供。

读取的时候需要注意自己统计总数，另外还要记录序号（写在数组里没有这个问题）。以及序号从$1$开始计数。

**纹理坐标**

例如

```
vt  0.617 0.575 0.000
vt  0.623 0.573 0.000
vt  0.627 0.576 0.000
...
```

以vt开头，后面还是带三个数。通常我们的纹理是二维的，所以第三个会是零。这里的纹理坐标指的是在纹理图片文件上的坐标。显然你会怀疑，纹理图片的大小不是800x800这种形式的吗？为什么这里是小数？

我认为是为了统一，我们只要用坐标乘以宽度、长度，再四舍五入即可得到图片上的像素坐标。

**法向量方向**

例如

```
vn  -0.250 0.462 -0.851
vn  -0.192 0.568 -0.801
vn  -0.359 0.279 0.891
...
```

描述了法向量指向的方向。

注意可能并不是单位向量，我没有找到限制必须是单位向量的规定。

**注意**

上面这三种数据总数不一定是一样的，它们之间的关联也不是一定是序号对应的。大部分时候不一样，也不对应。它们只是描述了一组这样的数据而已，具体到三角面上，才会体现如何使用这些数据。

使用这些数据时，序号就变得重要了起来。再次提醒序号从$1$开始计数。

**三角面信息**

例如

```
f 1206/1252/1206 1207/1254/1207 1205/1255/1205
f 1206/1252/1206 1208/1256/1208 1086/1257/1086
f 1206/1252/1206 1086/1257/1086 1087/1253/1087
...
```

以f开头，带了三组数据。每组都有三个数字。带有三个数字是因为要描述三角面的三个顶点。

第一组，描述了三个顶点的（模型）空间坐标。这三个数字是序号，我们用序号去访问数组即可得到坐标。

第二组，描述了三个顶点的材质uv坐标，也是序号。

第三组，描述了三个顶点的法向量坐标，也是序号。

我们目前了解以上信息即可。

# model.h

首先我们需要定义一下model里面要有什么内容。

```cpp
std::vector<geo::vec3f> vertices{};
std::vector<geo::vec2f> tex_coords{};
std::vector<geo::vec3f> norms{};
std::vector<size_t> face_vi{};
std::vector<size_t> face_ti{};
std::vector<size_t> face_ni{};
```

我们需要三维的顶点坐标、二维的纹理uv坐标，以及三维的顶点法向量坐标。同时也需要记录每个三角面它的顶点坐标序号，纹理坐标序号以及法向量坐标序号。

再看看我们可能需要什么方法：

```cpp
Model(std::string const & dir);
~Model();

size_t getFaceSize();
geo::vec3f getVert(size_t faceid, size_t nth);
bool getTriangle(std::array<geo::vec3f,3> & dist, size_t faceid);
bool getTriangle(std::array<geo::vec4f,3> & dist, size_t faceid);
bool getNorm(std::array<geo::vec3f,3> & dist, size_t faceid);
```

一个构造函数和一个析构函数不用多说。

之后我们可能需要提供获取三角面数量的函数、获取（某个特定）顶点坐标的函数，获取一个三角面上所有顶点坐标的函数（在这里我按需要提供了四维坐标和三维坐标的版本），以及获得三角面上所有顶点法向量的函数。

之后我们还会在读取材质纹理的地方需要提供纹理坐标的函数。这里先略过。

全部代码可见**[链接](https://github.com/kegalas/oar/blob/main/tutorial/chapter4/src/model.h)**

# model.cpp

现在我们来看看具体要怎么实现

## 构造函数

```cpp
Model::Model(std::string const & dir){
    std::ifstream in;
    in.open(dir,std::ifstream::in);
    if(in.fail()) return;

    std::string line;
    while(!in.eof()){
        std::getline(in,line);
        std::istringstream iss(line);
        char discard;

        if(line.compare(0,2,"v ")==0){
            iss>>discard;
            geo::vec3f v;
            for(int i=0;i<3;i++) iss>>v[i];
            vertices.push_back(v);
        }
        else if(line.compare(0,3,"vt ")==0){
            iss>>discard>>discard;
            geo::vec2f vt;
            for(int i=0;i<2;i++) iss>>vt[i];
            tex_coords.push_back(vt);
        }
        else if (line.compare(0,3,"vn ")==0) {
            iss>>discard>>discard;
            geo::vec3f vn;
            for(int i=0;i<3;i++) iss>>vn[i];
            norms.push_back(vn);
        }
        else if(line.compare(0,2,"f ")==0){
            iss>>discard;
            size_t fv,ft,fn;
            while(iss>>fv>>discard>>ft>>discard>>fn){
                face_vi.push_back(fv-1);
                face_ti.push_back(ft-1);
                face_ni.push_back(fn-1);
            }
        }
    }
    std::cerr<<"v: "<<vertices.size()<<" t: "<<tex_coords.size()<<" n: "<<norms.size()<<"\n";
    std::cerr<<"f: "<<face_ni.size()/3<<"\n";
}
```

这其实没什么好多说的，把之前说过的OBJ文件格式理解一下，去实际看看obj文件里面怎么写的。如果掌握了ifstream，字符串的操作，整个操作就会了。

需要注意的是，obj里面提供的序号是从$1$开始的，而我这里用的是往vector里push_back元素的办法，方便起见把所有序号都减$1$。

## 获取坐标信息的函数

这一部分其实非常容易理解和实现，就把我们读入的数据，按照下标获取，然后返回即可。全部代码可见**[链接](https://github.com/kegalas/oar/blob/main/tutorial/chapter4/src/model.cpp)**

# 使用例子

```cpp
#include "tga_image.h"
#include "raster.h"
#include "model.h"
#include <ctime>
#include <cstdlib>

int const width = 1500;
int const height = 1500;//设置输出图片的长宽

int main(){
    Model model("../obj/african_head.obj");//读取我们的模型
    TGAImage image(width,height,TGAType::rgb);//新建一个图对象
    int nface = model.getFaceSize();//获取三角面的总数
    std::array<geo::vec4f,3> vert;//读取顶点坐标
    std::array<geo::OARColor,3> color;//设置顶点颜色
    std::array<geo::vec2i,3> screen;//把顶点的空间坐标转化到屏幕像素坐标上

    for(int i=0;i<nface;i++){
        model.getTriangle(vert, i);//把序号为i的三角形的顶点全部读入vert
        for(int j=0;j<3;j++){
            color[j] = geo::OARColor(255,255,255,255);//设置颜色为全白
            screen[j] = ras::world2screen(vert[j],width,height);//转换坐标，函数定义见下
        }
        ras::triangle(image,screen,color);//绘制三角形i号
    }
    image.writeToFile("./af.tga");//写入图像文件

    return 0;
}
```

我们使用以上代码来测试我们的效果，具体步骤解释都写在上面的注释里了（模型见**[链接](https://github.com/kegalas/oar/tree/main/tutorial/chapter4/obj)** ）。

```cpp
geo::vec2i ras::world2screen(geo::vec4f v, int width, int height){
    return geo::vec2i(int((v.x+1.f)*width*.5f+.5f), int((v.y+1.f)*height*.5f+.5f));
}
```

我们的坐标转换函数如上（放在这里**[链接](https://github.com/kegalas/oar/blob/main/tutorial/chapter4/src/raster.cpp#L94)** ），这里本来应该在坐标转换的地方讲，但是这里不得不用到，我们就提前讲一下。

我们的物体模型，有他自己的坐标系。这个坐标的范围是$[-1,1]^3$，但我们的显示屏只有两个维度，而且他还只有整数的坐标。为了将模型的坐标映射到显示屏上，我们首先压缩一维，也就是直接不考虑深度（这里是$z$轴），然后把$[-1,1]^2$映射到$\text{width}\times\text{height}$这个坐标上。我们分别对$x,y$进行操作，首先把$x$整体加$1$，就得到$[0,2]$，再除以$2$，就得到$[0,1]$，再乘以$\text{width}$，再取四舍五入到整数就得到了显示屏上的横坐标，同理就得到了纵坐标。

更进一步的知识等到坐标转换的地方再讲吧！

结果如下：

![1.png](1.png)

虽然我们没有看清他的面容（因为我们把每个三角形都设置成全白了），但是至少整个形状正确了。稍后我们将介绍光照，来让他的面容展现出来。

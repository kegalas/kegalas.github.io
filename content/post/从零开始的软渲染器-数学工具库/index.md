---
title: 干货 | 从零开始的软渲染器 数学工具库
date: 2022-10-28T15:34:04+08:00
draft: false
tags:
  - 图形学
  - 渲染
  - 数学
  - 干货
categories: 图形学
mathjax: true
markup: goldmark
image: cover.jpg
---

<u>**[导航页面](../干货-从零开始的软渲染器-导航/)**</u>

图形学中最基础的数学涉及到线性代数，我们需要自己先写一个能处理简单的向量、矩阵运算的库。前置知识：大学本科水平的线性代数。

# geometry.h

## 向量的定义

首先构造一个一般化的向量类

```cpp
template<class T, size_t dim>
class Vec{
public:
    Vec(){
        for(size_t i=0;i<dim;i++){
            data_[i] = T();
        }
    }

    Vec(Vec<T,dim> const & v){
        for(size_t i=0;i<dim;i++){
            data_[i] = v[i];
        }
    }

    T& operator[](const size_t i){
        assert(i<dim && i>=0);
        return data_[i];
    }

    const T& operator[](const size_t i) const {
        assert(i<dim && i>=0);
        return data_[i];
    }

    Vec<T, dim>& operator=(const Vec<T,dim> vec){
        if(this==&vec){
            return *this;
        }

        for(size_t i=0;i<dim;i++){
            this->data_[i] = vec[i];
        }
        
        return *this;
    }

private:
    T data_[dim];
};
```

首先`template<class T, size_t dim>`指的是让它的向量元素的数据类型为`T`，然后有`dim`维

`Vec()`这个默认构造函数不难理解，`Vec(Vec<T,dim> const & v)`是一个复制构造函数，而相应的`Vec<T, dim>& operator=(const Vec<T,dim> vec)`是一个复制赋值运算符。

`T& operator[]`是为了方便取向量的某一个元素，并且同时提供了const的版本，这是一个经常可以见到的设计。并且我们检查数组越界。

下面对其进行模板特殊化

```cpp
template<class T>
class Vec<T,2>{
public:
    union{
        struct {T x,y;};
        struct {T s,t;};
        struct {T u,v;};
        T raw[2];
    };

    Vec<T,2>():x(),y(){}

    Vec<T,2>(T x_, T y_):x(x_),y(y_){}

    Vec<T,2>(Vec<T,2> const & v):x(v.x),y(v.y){}

    template<class U> Vec<T,2>(Vec<U,2> const &);
    template<class U> Vec<T,2>(Vec<U,3> const &);
    template<class U> Vec<T,2>(Vec<U,4> const &);

    T& operator[](const size_t i){
        assert(i<2 && i>=0);
        return raw[i];
    }

    const T& operator[](const size_t i) const{
        assert(i<2 && i>=0);
        return raw[i];
    }

    Vec<T, 2>& operator=(const Vec<T,2>& vec){
        if(this==&vec){
            return *this;
        }
        for(size_t i=0;i<2;i++){
            this->raw[i] = vec[i];
        }
        
        return *this;
    }

};
```

如上，我们定义了一个二维的向量，这个叫做模板特殊化，当我们声明`Vec<float,2>`时，会首先调用最特殊的那个，在此处会调用`Vec<T,2>`，而不是更一般的`Vec<T,dim>`

另外需要注意的是，模板特殊化并不是继承，特殊的模板不会继承一般的模板的成员。所以我们需要重新地定义整个`Vec<T,2>`

```cpp
union{
    struct {T x,y;};
    struct {T s,t;};
    struct {T u,v;};
    T raw[2];
};
```

这是一个联合，但实际上只用了两个sizeof(T)的内存，其中x、s、u和raw[0]表示的都是第一维，而y、t、v、raw[1]表示的是第二维

```cpp
Vec<T,2>():x(),y(){}

Vec<T,2>(T x_, T y_):x(x_),y(y_){}

Vec<T,2>(Vec<T,2> const & v):x(v.x),y(v.y){}
```

这是三个构造函数，有时我们可能会写成

```cpp
Vec<T,2>(T x_, T y_){
    x = x_;
    y = y_;
}
```

这效果是相同的，但是如果T是一个类，而不是一个int、float这样的类型，那会导致性能问题。

此时会先调用x的默认构造函数，然后再将x_赋值给x，调用复制赋值运算符，这是两步。而我们用成员初始化列表就直接调用了赋值构造函数，只有一步。

虽然你可能会想为什么会有人给向量除了int和float之外的类型，我觉得你最好不要推断用户的想法。

```cpp
template<class U> Vec<T,2>(Vec<U,2> const &);
template<class U> Vec<T,2>(Vec<U,3> const &);
template<class U> Vec<T,2>(Vec<U,4> const &);
```

这是我们之后要让float类型和int类型互相转化而提前声明的模板。

对三维和四维的向量基本相同，目前我们要学的图形学不会用到更高维的向量了。

下面开始定义向量的各种运算

```cpp
template<class T, size_t dim>
Vec<T, dim> operator+(Vec<T, dim> const & lhs, Vec<T, dim> const & rhs){
    Vec<T,dim> ret;
    for(size_t i=0;i<dim;i++){
        ret[i] = lhs[i] + rhs[i];
    }
    return ret;
}
```

代码不难理解，我们选择在全局定义这个运算符，而不是在类内部定义成员函数。这是为了更好的泛用性。以及缩短码量。

其他的加减乘除类似，不过我们这里的乘法和除法指的是各个元素对应的乘除，并且我们还要定义向量和的数乘，数除。数除还应该注意左右操作数。

然后再定义点乘和叉乘

```cpp
template<class T, size_t dim>
T dot(Vec<T, dim> const & lhs, Vec<T, dim> const & rhs){
    T ret = T();

    for(int i=0;i<dim;i++){
        ret += lhs[i] * rhs[i];
    }

    return ret;
}

template<class T>
Vec<T, 3> cross(Vec<T, 3> const & lhs, Vec<T, 3> const & rhs){
    Vec<T, 3> ret;

    ret.x = lhs.y*rhs.z - lhs.z*rhs.y;
    ret.y = lhs.z*rhs.x - lhs.x*rhs.z;
    ret.z = lhs.x*rhs.y - lhs.y*rhs.x;

    return ret;
}
```

不把\*设计为点乘，主要是考虑到OpenGL也是用dot表示点乘，而\*表示元素对应相乘。

代码并不困难，数学也是高中知识，值得注意的是，叉乘只有三维向量才有定义。

```cpp
template<class T, size_t dim>
T norm(Vec<T, dim> const & vec){
    T ret = dot(vec,vec);
    ret = std::sqrt(ret);
    return ret;
}
```

定义向量的2-范数，也就是$\sqrt{x_1^2+x_2^2+\cdots}$，这可以方便我们计算长度。1-范数和无穷范数暂时用不上。

```cpp
template<class T, size_t dim>
void normalize(Vec<T, dim> & vec){
    T vnorm = (static_cast<T> (1))/norm(vec);
    for(size_t i=0;i<dim;i++){
        vec[i] *= vnorm;
    }
}

template<class T, size_t dim>
Vec<T, dim> normalized(Vec<T, dim> const & vec){
    Vec<T, dim> ret;
    T vnorm = (static_cast<T> (1))/norm(vec);
    for(size_t i=0;i<dim;i++){
        ret[i] = vec[i] * vnorm;
    }
    return ret;
}
```

分别定义了，将一个向量标准化，和返回一个向量的标准化。后者是可以对const变量使用的。

值得注意的一点是，我们使用vnorm这个变量，是因为除法比乘法慢，我们通过预先计算倒数，来加速这个过程。

```cpp
template<class T, size_t dim>
std::ostream& operator<<(std::ostream& out, const Vec<T, dim>& vec){
    for(size_t i=0;i<dim;i++){
        out<<vec[i]<<" ";
    }
    return out;
}
```

我们重载了输出流，方便直接打印出来调试。

```cpp
Vec<int,4> toOARColor(int const & v);
Vec<int,4> toOARColor(float const & v);
Vec<int,4> toOARColor(Vec<int,3> const & v);
Vec<int,4> toOARColor(Vec<float,3> const & v);
Vec<int,4> toOARColor(Vec<int,4> const & v);
Vec<int,4> toOARColor(Vec<float,4> const & v);
```

在计算机图形学中，颜色也可以表示为一个向量，为此我们声明几个函数将其他的向量转变为RGB或RGBA颜色值。

这里其中前两个是转换灰度值，中间两个是转换RGB，最后两个是转换RGBA。具体的实现后面会介绍到。

## 矩阵的定义

```cpp
template<class T, size_t nRow, size_t nCol>
class Mat{
private:
    Vec<T, nCol> rowVec[nRow];

public:
    Mat<T, nRow, nCol>(){
        size_t minDim = std::min(nRow, nCol);
        for(size_t i=0 ; i<minDim ; i++){
            rowVec[i][i] = static_cast<T> (1);
        }
    }

    Mat<T, nRow, nCol>(T const value){
        size_t minDim = std::min(nRow, nCol);
        for(size_t i=0 ; i<minDim ; i++){
            rowVec[i][i] = value;
        }
    }

    Mat<T, nRow, nCol>(Mat<T, nRow, nCol> const & m){
        for(size_t i=0 ; i<nRow ; i++){
            rowVec[i] = m[i];
        }
    }

    Vec<T, nCol>& operator[](size_t const i){
        assert(i>=0 && i<nRow);
        return rowVec[i];
    }

    const Vec<T, nCol>& operator[](size_t const i) const {
        assert(i>=0 && i<nRow);
        return rowVec[i];
    }

    Mat<T, nRow, nCol>& operator=(Mat<T, nRow, nCol> const & mat){
        if(this==&mat){
            return *this;
        }
        for(size_t i=0 ; i<nRow ; i++){
            this->rowVec[i] = mat[i];
        }
        return *this;
    }

    Vec<T, nRow> getColVec(size_t const index) const {
        assert(index>=0 && index<nCol);
        Vec<T, nRow> ret;
        for(size_t i=0 ; i<nRow ; i++){
            ret[i] = rowVec[i][index];
        }
        return ret;
    }

    void setColVec(size_t const index, Vec<T, nRow> const & vec){
        assert(index>=0 && index<nCol);
        for(size_t i=0 ; i<nRow ; i++){
            rowVec[i][index] = vec[i];
        }
    }
};
```

```cpp
Vec<T, nCol> rowVec[nRow]
```

这一句，意味着我们采取了行先序，我们将一个矩阵看成了n个行向量。意味着mat[1][2]代表着第1行的第2个元素。行先序和matlab一致，也和glm库一致。

我们的默认构造函数默认地构造了一个单位矩阵，即主对角线上的元素都是1，而Mat<T, nRow, nCol>(T const value)这个构造函数将主对角线上的元素设置为value。

```cpp
Mat<T, nRow, nCol>& operator=(Mat<T, nRow, nCol> const & mat){
    if(this==&mat){
        return *this;
    }
    for(size_t i=0 ; i<nRow ; i++){
        this->rowVec[i] = mat[i];
    }
    return *this;
}
```

另外我们提供了复制构造函数以及复制赋值运算符，后者是很有必要的，否则默认的赋值运算，将会把rowVec这个指针指向右值中对应的地址，此时两个矩阵实际上用的是同一份数据，为了避免这个，我们有必要重新设计复制赋值运算符。

```cpp
Vec<T, nRow> getColVec(size_t const index) const {
    assert(index>=0 && index<nCol);
    Vec<T, nRow> ret;
    for(size_t i=0 ; i<nRow ; i++){
        ret[i] = rowVec[i][index];
    }
    return ret;
}

void setColVec(size_t const index, Vec<T, nRow> const & vec){
    assert(index>=0 && index<nCol);
    for(size_t i=0 ; i<nRow ; i++){
        rowVec[i][index] = vec[i];
    }
}
```

我们也提供了获取和修改列向量的函数。

接下来同样是提供全局的运算符重载。

矩阵的相加、相减、数乘、数除和向量的代码基本一致。但同样为了和OpenGL着色器语言保持一致，我们矩阵乘以向量和向量相乘用的是*符号。

```cpp
template<class T, size_t nRow, size_t nCol>
Vec<T, nRow> operator*(Mat<T, nRow, nCol> const & lhs, Vec<T, nCol> const & rhs){
    Vec<T, nRow> ret;
    for(size_t i=0 ; i<nRow ; i++){
        ret[i] = dot(lhs[i] , rhs);
    }
    return ret;
}

template<class T, size_t nRow, size_t nCol, size_t sameDim>
Mat<T, nRow, nCol> operator*(Mat<T, nRow, sameDim> const & lhs, Mat<T, sameDim, nCol> const & rhs){
    Mat<T, nRow, nCol> ret;
    for(size_t i=0 ; i<nRow ; i++){
        for(size_t j=0 ; j<nCol ; j++){
            ret[i][j] = dot(lhs[i] , rhs.getColVec(j));
        }
    }
    return ret;
}
```

这里我们默认向量都是列向量，所以矩阵*向量要求向量的维数和矩阵的列数相同。* 在图形学中，我们大部分时候其实只会把矩阵当成左操作数，向量当成右操作数，所以我们就暂时不考虑相反情况的代码了。

而矩阵乘以矩阵要求前者的列数等于后者的行数。

用模板可以很好地实现上述内容。

```cpp
template<class T, size_t nRow, size_t nCol>
Mat<T, nCol, nRow> transpose(Mat<T, nRow, nCol> const & mat){
    Mat<T, nCol, nRow> ret;
    for(size_t i=0 ; i<nRow ; i++){
        ret.setColVec(i, mat[i]);
    }
    return ret;
}
```

矩阵转置很简单，将原来的行向量设置给新的列向量即可。

```cpp
template<class T, size_t dim>
Mat<T, dim-1, dim-1> getMinor(Mat<T, dim, dim> const & mat, size_t x, size_t y){
    assert(dim>0);
    Mat<T, dim-1, dim-1> ret;
    for(size_t i=0;i<dim-1;i++){
        for(size_t j=0;j<dim-1;j++){
            ret[i][j] = mat[i<x?i:i+1][j<y?j:j+1];
        }
    }
    return ret;
}
```

这是在计算一个矩阵，去除某个元素的所在行和所在列得到的子矩阵。

```cpp
template<class T, size_t dim>
T det(Mat<T, dim, dim> const & mat){
    assert(dim>0);

    T ret = T();

    for(size_t i=0;i<dim;i++){  
        ret += cofactor(mat, 0, i) * mat[0][i];
    }

    return ret;
}

template<class T>
T det(Mat<T,2,2> const & mat){
    return mat[0][0]*mat[1][1] - mat[0][1]*mat[1][0];
}

template<class T>
T det(Mat<T,1,1> const & mat){
    return mat[0][0];
}

template<class T>
T det(Mat<T,0,0> const & mat){
    return static_cast<T>(0);
}

template<class T, size_t dim>
T cofactor(Mat<T, dim, dim> const & mat, size_t x, size_t y){
    return det(getMinor(mat,x,y)) * static_cast<T>((x+y)%2?-1:1);
}
```

这是计算行列式与代数余子式的代码。

计算大于2阶的行列式，需要用到代数余子式的方法，可以参考线性代数笔记。

值得注意的是，det中我们每次计算都会减少dim一阶，但是你并不能使用`if(dim==2);if(dim==1);if(dim<=0)`这样的语句来在模板内部定义特例，这是无法编译通过的。你需要做的是将模板特殊化，这样才能通过编译。

```cpp
template<class T, size_t dim>
Mat<T,dim,dim> adjugate(Mat<T, dim, dim> const & mat){
    Mat<T, dim, dim> ret;
    for(size_t i=0;i<dim;i++){
        for(size_t j=0;j<dim;j++){
            ret[j][i] = cofactor(mat,i,j);
        }
    }
    return ret;
}

template<class T, size_t dim>
Mat<T,dim,dim> inverse(Mat<T, dim, dim> const & mat){
    Mat<T, dim, dim> ret;
    ret = adjugate(mat)/det(mat);
    return ret;
}
```

这是计算伴随矩阵和逆矩阵的代码，有一点值得注意，就是伴随矩阵的下标相当于进行了转置。

```cpp
typedef Vec<float,2> vec2f;
typedef Vec<float,3> vec3f;
typedef Vec<float,4> vec4f;
typedef Vec<int,2> vec2i;
typedef Vec<int,3> vec3i;
typedef Vec<int,4> vec4i;
typedef Vec<int,4> OARColor;

typedef Mat<float, 4, 4> mat4f;
typedef Mat<int, 4, 4> mat3f;
```

最后声明一下变量的别名方便使用。

另外这些全部都是在geo命名空间中，方便辨别。

着重讲一下为什么我们的颜色信息（OARColor）被设置成4维的int向量。

相信大家都听过RGB颜色，在这上面多加一维透明度，就是RGBA，基本上可以达到我们所有想要的效果了。而RGBA四个属性的取值范围分别都是$[0,255]$的整数，所以用四维int向量就理所当然了。

你可能会问为什么不拆成黑白、RGB、RGBA三种颜色。确实，这可能会更省内存，但是现在电脑内存一点也不稀缺，并且我们的微型渲染器很难有多大开销，统一处理可能更简单一些。

完整的代码在<u>**[这里](https://github.com/kegalas/oar/blob/main/tutorial/chapter1/geometry.h)**</u>

# geometry.cpp

这个文件里大部分都是涉及向量类型转换的。唯一需要注意的是四舍五入。以三维为例

```cpp
template<> template<>
geo::Vec<int,3>::Vec(geo::Vec<float, 2> const & v, float const z_){
    x = static_cast<int>(v.x + 0.5f);
    y = static_cast<int>(v.y + 0.5f);
    z = static_cast<int>(z_ + 0.5f);
}
```

这段代码将二维的float向量转为三维的int向量，+0.5f即是四舍五入。

然后就是转化为RGBA颜色格式，为了简单起见，我们假设所有的颜色都用RGBA表示，所以我们需要将RGB和灰度图像转化为RGBA，然后在写入图像时再具体分析写入什么颜色数据。举例将float的RGB和int的灰度转换为RGBA：

（注：RGBA其实每一个颜色的权重也可以表示为$[0,1]$之间的实数（这样在处理光照的时候容易些），但我们的图像写出去的是离散的值，所以要将$[0,1]$映射到$[0,255]$上的实数。注意四舍五入）

```cpp
geo::Vec<int,4> geo::toOARColor(geo::Vec<float,3> const & v){
    OARColor ret;
    for(int i=0;i<3;i++){
        ret[i] = static_cast<int>(v[i]*255.f+0.5f);
        if (ret[i]<0) ret[i] = 0;
        else if(ret[i]>255) ret[i] = 255;
    }
    ret[3] = 255;

    return ret;
}

geo::Vec<int,4> geo::toOARColor(int const & v){
    OARColor ret;
    for(int i=0;i<3;i++){
        if(v<0) ret[i] = 0;
        else if(v>255) ret[i] = 255;
        else ret[i] = v;
    }
    ret[3] = 255;

    return ret;
}
```

其将连续的[0,1]映射到离散的[0,255]（或者本来就是离散的），并且超出的部分映射到边界上，同时也计算了四舍五入。

完整的代码在<u>**[这里](https://github.com/kegalas/oar/blob/main/tutorial/chapter1/geometry.cpp)**</u>

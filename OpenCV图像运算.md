这篇文章将要介绍OpenCV图像运算的相关内容。图像就是普通的矩阵，因此可以进行加、减、乘、除等操作，所以也就可以用多种方式组合图像。



## 基本的图像运算函数

使用 `cv::addWeighted` 实现图像加权和即图像叠加
​    cv::addWeighted(image1,0.7,image2,0.9,0.,result);



`cv::add`是图像运算的一个基础公式，下面给出图像计算的函数方法及其对应的像素处理版：

```cpp
// c[i]= a[i]+b[i];
cv::add(imageA, imageB, resultC);
// c[i]= a[i]+k;
cv::add(imageA, cv::Scalar(k), resultC);
// c[i]= k1*a[i]+k2*b[i]+k3;
cv::addWeighted(imageA, k1, imageB, k2, k3, resultC);
// c[i]= k*a[i]+b[i];
cv::scaleAdd(imageA, k, imageB, resultC);
```



有些函数还可以指定一个掩码：

```cpp
// 如果(mask[i]) c[i]= a[i]+b[i];
cv::add(imageA, imageB, resultC, mask);
```



使用掩码后，操作就只会在掩码值非空的像素上执行（掩码必须是单通道的）。`cv::subtract`、 `cv::absdiff`、 `cv::multiply` 和 `cv::divide` 等函数也同样支持掩码的使用。



位运算符（对像素的二进制数值进行按位运算）:

- `cv::bitwise_and`
- `cv::bitwise_or`
- `cv::bitwise_xor` 
- `cv::bitwise_not`



还有使用单个输入图像的运算符，它们是 `cv::sqrt`、 `cv::pow`、 `cv::abs`、 `cv::cuberoot`、`cv::exp` 和 `cv::log`。



`cv::min` 和 `cv::max` 运算符也非常实用，它们能找到每个元素中最大或最小的像素值。、





注意：

- 在所有场合都要使用 `cv::saturate_cast` 函数，以确保结果在预定的像素值范围之内（避免上溢或下溢）。
- 这些图像必定有相同的大小和类型（如果与输入图像的大小不匹配，输出图像会重新分配）。
- 由于运算是逐个元素进行的，因此可以把其中的一个输入图像用作输出图像。



## 重载图像运算符

OpenCV 的大多数运算函数都有对应的重载运算符，因此调用 `cv::addWeighted`的语句也可以写成：
    result = 0.7* image1 + 0.9 * image2;
这种代码更加紧凑也更容易阅读。这两种计算加权和的方法是等效的。特别指出，这两种方法都会调 `cv::saturate_cast` 函数，所以无需担心结果超出范围的情况。



而事实上，大部分 C++运算符都已被重载，其中包括位运算符`&`、 `|`、`^` 、 `~`和函数 `min`、 `max`、 `abs`。



同时，比较运算符`<`、`<=`、 `==`、 `!=`、`>`和`>=`也已被重载，它们返回一个 8 位的二值图像。此外还有矩阵乘法 `m1*m2`（其中 m1 和 m2 都是 cv::Mat 实例）、矩阵求逆 `m1.inv()`、变位 `m1.t()`、行列式 `m1.determinant()`、求范数 `v1.norm()`、叉乘 `v1.cross(v2)`、点乘 `v1.dot(v2)`，等等。



在《图片处理：减色》那篇博客中，我们介绍了一个减色函数，它使用循环来扫描图像的像素并对像素进行运算操作。



在这里，我们也可以使用针对输入图像的运算符简单地重写这个函数：
​    image = (image & cv::Scalar(mask, mask, mask)) + cv::Scalar(div / 2, div / 2, div / 2);



由于被操作的是彩色图像，因此使用了 `cv::Scalar`形成向量。



## 分割图像通道

我们有时需要分别处理图像中的不同通道，例如只对图像中的一个通道执行某个操作。这当然可以通过图像扫描循环实现，但也可以使用 `cv::split` 函数，将图像的三个通道分别复制到三个 `cv::Mat` 实例中。假设我们要把一张图只加到蓝色通道中，可以这样实现：

```cpp
// 创建三幅图像的向量
std::vector<cv::Mat> planes;
// 将一个三通道图像分割为三个单通道图像
cv::split(image1, planes);
// 加到蓝色通道上
planes[0] += image2;
// 将三个单通道图像合并为一个三通道图像
cv::merge(planes, result);
```

这里的 `cv::merge` 函数执行反向操作，即用三个单通道图像创建一个彩色图像。
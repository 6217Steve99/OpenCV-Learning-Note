这篇文章我们来一起看看如何通过移动像素修改图像的外观。这个过程不会修改像素值，而是把每个像素的位置重新映射到新的位置。



这一方法不但可以用来创建图像特效，也可以修正因镜片等原因导致的图像扭曲。



要使用 OpenCV 实现图像重映射就需要使用其中的 `remap` 函数。



具体来说，我们首先需要定义在重映射处理中使用的映射参数，然后把映射参数应用到输入图像。而定义映射参数的方式将决定产生的效果。在这里我们定义一个转换函数，在图像上创建波浪形效果：



```cpp
// 重映射图像，创建波浪形效果
void wave(const cv::Mat &image, cv::Mat &result)
{
    // 映射参数
    cv::Mat srcX(image.rows, image.cols, CV_32F);
    cv::Mat srcY(image.rows, image.cols, CV_32F);
    // 创建映射参数
    for (int i = 0; i < image.rows; i++)
    {
        for (int j = 0; j < image.cols; j++)
        {
            // (i,j)像素的新位置
            srcX.at<float>(i, j) = j; // 保持在同一列
            // 原来在第 i 行的像素，现在根据一个正弦曲线移动
            srcY.at<float>(i, j) = i + 5 * sin(j / 10.0);
        }
    }
    // 应用映射参数
    cv::remap(image,             // 源图像
              result,            // 目标图像
              srcX,              // x 映射
              srcY,              // y 映射
              cv::INTER_LINEAR); // 填补方法
}
```



下面我们来一点一点介绍每行代码的作用，从而加深对于OpenCV实现重映射的理解。



重映射是通过修改像素的位置，生成一个新版本的图像。而为了构建新图像，我们必须要知道目标图像中每个像素的原始位置。因此，我们需要的映射函数应该能根据像素的新位置得到像素的原始位置。这个转换过程描述了如何把新图像的像素映射回原始图像，因此称为反向映射。在 OpenCV中，可以用两个映射参数来说明反向映射：一个针对 x 坐标，另一个针对 y 坐标。它们都用浮点数型的 `cv::Mat` 实例来表示：

```cpp
// 映射参数
cv::Mat srcX(image.rows, image.cols, CV_32F); // x 方向
cv::Mat srcY(image.rows, image.cols, CV_32F); // y 方向
```





这些矩阵的大小决定了目标图像的大小。用下面的代码可以从原始图像获得目标图像中 `(i,j)` 像素的值：

srcX.at<float>(i, j), srcY.at<float>(i, j)




最后，我们只需调用 OpenCV 的 remap 函数，即可生成结果图像：

```cpp
// 应用映射参数
cv::remap(image,             // 源图像
          result,            // 目标图像
          srcX,              // x 方向映射
          srcY,              // y 方向映射
          cv::INTER_LINEAR); // 插值法
```



这里我们不难发现，这两个映射参数包含的值其实是浮点数。也就是说，目标图像中的像素可以映射回一个非整数的值（即处在两个像素之间的位置），这也使得我们定义映射参数时能够更加的随意。而在这个重映射例子中，我们用了一个 `sin` 函数进行转换，但这也导致必须在真实像素之间插入虚拟像素的值。而 `remap` 函数的最后一个参数就是用来表示选择了哪种像素插值方法，这里采用线性插值。



补充，这里再给出一个图像水平翻转效果的映射参数：

```cpp
// 创建映射参数
for (int i = 0; i < image.rows; i++)
{
    for (int j = 0; j < image.cols; j++)
    {
        // 水平翻转
        srcX.at<float>(i, j) = image.cols - j - 1;
        srcY.at<float>(i, j) = i;
    }
}
```


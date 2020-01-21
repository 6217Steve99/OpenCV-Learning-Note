`cv::grabCut` 函数只需要输入一幅图像，并对一些像素做上“属于背景”或“属于前景”的标记，根据这个局部标记，算法将计算出整幅图像的前景和背景的分割线。

一种指定输入图像局部前景/背景标签的方法是定义一个包含前景物体的矩形：

```c++
// 定义一个带边框的矩形
// 矩形外部的像素会被标记为背景
cv::Rect rectangle(5,70,260,120);
```

这段代码定义了图像中的一个区域。

![Image with rectangle](E:\Codes\Opencv\Opencv模板\Opencv模板\Opencv模板\Image with rectangle.jpg)

矩形之外的像素都会被标记为背景。调用 `cv::grabCut` 时，除了需要输入图像和分割后的图像，还需要定义两个矩阵，用于存放算法构建的模型，代码如下所示：

```c++
cv::Mat result; // 分割结果（四种可能的值）
cv::Mat bgModel,fgModel; // 模型（内部使用）
// GrabCut 分割算法
cv::grabCut(image, // 输入图像
		result, // 分割结果
		rectangle, // 包含前景的矩形
		bgModel,fgModel, // 模型
		5, // 迭代次数
		cv::GC_INIT_WITH_RECT); // 使用矩形
```

我们在函数的中用 `cv::GC_INIT_WITH_RECT` 标志作为最后一个参数，表示将使用带边框的矩形模型。输入或输出的分割图像可以是以下四个值之一。

- `cv::GC_BGD`：这个值表示明确属于背景的像素（例如本例中矩形之外的像素）。
- `cv::GC_FGD`：这个值表示明确属于前景的像素（本例中没有这种像素）。
- `cv::GC_PR_BGD`：这个值表示可能属于背景的像素。
- `cv::GC_PR_FGD`：这个值表示可能属于前景的像素（即本例中矩形之内像素的初始值）。

通过 `cv::compare` 函数提取值为 `cv::GC_PR_FGD` 的像素，可得到包含分割信息的二值图像，实现代码为：

```c++
// 取得标记为“可能属于前景”的像素
cv::compare(result,cv::GC_PR_FGD,result,cv::CMP_EQ);
// 生成输出图像
cv::Mat foreground(image.size(),CV_8UC3,cv::Scalar(255,255,255));
image.copyTo(foreground, result); // 不复制背景像素
```



其中 `cv::compare` 的使用方法如下：

```c++
cv::compare()
	bool cv::compare(
	cv::InputArray src1, // 输入数组1
	cv::InputArray src2, // 输入数组2
	cv::OutputArray dst, // 输出数组
	int cmpop // 比较操作子
);
```

| Value of cmp_op |   Comparison   |
| :-------------: | :------------: |
|   cv::CMP_EQ    | src1i == src2i |
|   cv::CMP_GT    | src1i > src2i  |
|   cv::CMP_GE    | src1i >= src2i |
|   cv::CMP_LT    | src1i < src2i  |
|   cv::CMP_LE    | src1i <= src2i |
|   cv::CMP_NE    | src1i != src2i |



要提取全部前景像素，即值为 `cv::GC_PR_FGD` 或 `cv::GC_FGD` 的像素，也可以检查第一位的值，代码如下所示：

```c++
// 用“按位与”运算检查第一位
result= result&1; // 如果是前景像素，结果为 1
```



这是因为这几个常量被定义的值为 1 和 3，而另外两个（`cv::GC_BGD` 和 `cv::GC_PR_BGD`）被定义为 0 和 2。



得到的图像如下所示。

![Foreground object](E:\Codes\Opencv\Opencv模板\Opencv模板\Opencv模板\Foreground object.jpg)





在上面的例子中，只需要指定一个包含前景物体（城堡）的矩形， GrabCut 算法就能提取出它。此外，还可以把输入图像中的几个特定像素赋值为 `cv::GC_BGD` 和`cv::GC_FGD`，以掩码图像的形式提供这些值，作为`cv::grabCut` 函数的第二个参数。同时要把输入模式标志指定为 `GC_INIT_WITH_MASK`。获得这些输入标签的方法有很多种，例如可以提示用户在图像中交互式地标记一些元素。当然，将这两种输入模式结合使用也未尝不可。

利用输入信息， GrabCut 算法通过以下步骤进行背景/前景分割。首先，把所有未标记的像素临时标为前景（`cv::GC_PR_FGD`）。基于当前的分类情况，算法把像素划分为多个颜色相似的组（即 K 个背景组和 K 个前景组）。下一步是通过引入前景和背景像素之间的边缘，确定背景/前景的分割，这将通过一个优化过程来实现。在此过程中，将试图连接具有相似标记的像素，并且避免边缘出现在强度相对均匀的区域。使用 Graph Cuts 算法可以高效地解决这个优化问题，它寻找最优解决方案的方法是：把问题表示成一幅连通的图形，然后在图形上进行切割，以形成最优的
形态。分割完成后，像素会有新的标记。然后重复这个分组过程，找到新的最优分割方案，如此反复。因此，GrabCut 算法是一个逐步改进分割结果的迭代过程。根据场景的复杂程度，找到最佳方案所需的迭代次数各不相同（如果情况简单，迭代一次就足够了）。

这解释了函数中用来表示迭代次数的参数。结合代码看，原意应该是：先把参数传递给函数，函数返回时会修改参数的值。因此，如果希望通过执行额外的迭代过程来改进分割结果，可以在调用函数时重复使用上次运行的模型
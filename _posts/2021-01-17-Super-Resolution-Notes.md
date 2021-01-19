---
title: Dictionary-based Super Resolution Methods Notes
date: 2021-01-19 23:20
---
# Dictionary-based Super Resolution Methods Notes

## 1. 稀疏字典学习超分辨率的基本原理

超分辨率的主要方法有插值方法 (interpolation methods)、多帧合成的方法 (multi-frame methods)和基于学习的方法 (learning-based methods)。主要关注基于字典学习的超分辨率方法 (Dictionary methods)，起初的字典学习的方法算法复杂度很高，为了克服这些问题，有邻域嵌入超分辨率方法 (Neighbor embedding, NE) [3]，基于稀疏编码的方法 (Sparse coding methods) [1]，以及 [2] 中提出的锚定邻域回归方法 (Anchored Neighbor Regression, ANR)。

### 1.1 基于稀疏编码的方法

关于稀疏信号表示的研究表明，高分辨率的信号在一定条件下可以从它的低维映射精确的恢复出来，具体到图像超分辨率问题中就是高分辨率图像在一个过完备字典下存在一个稀疏表示，可以通过这个稀疏表示来恢复出原始高分辨率图像。Yang 的方法并不直接去寻找高分辨率图像的稀疏表示，而是通过联合字典学习一个字典对 ($\boldsymbol{D_l}, \boldsymbol{D_h})$ 。求解低分辨率的输入图像块在低分辨率字典 $\boldsymbol{D_l}$ 下的稀疏表示 $\alpha$ ，根据目标高分辨率图像块在高分辨率字典 $\boldsymbol{D_h}$ 下具有相同的稀疏表示 $\alpha$ 来重构出高分辨率图像。

可以通过以下最优化问题来根据输入的低分辨率图像块 $\boldsymbol{y}$ 和求解稀疏表示 $\alpha$ ，其中 $F$ 表示特征变换矩阵：

$$
\min ||\alpha||_0 \quad s.t. \quad||F\boldsymbol{D_l}\alpha-Fy||_2^2\leq\epsilon
$$

由于 $\ell^0$ 范数的最优化问题是一个 NP-hard 问题，可以使用 $\ell^1$ 范数的最优化问题代替求解：

$$
\min ||\alpha||_1 \quad s.t. \quad||F\boldsymbol{D_l}\alpha-Fy||_2^2\leq\epsilon
$$

### 1.2 锚定邻域回归方法

Timofte 在 [2] 中提出了锚定邻域回归方法 (ANR) 和全局回归 (GR) 方法，GR 方法是 ANR 方法的极端情况。

GR 算法相对稀疏编码方法，首先把最小二乘回归中稀疏表示的 $\ell^1$ 范数的约束重构成 $\ell^2$ 范数最小二乘回归问题，这样就可以使用脊回归 (Ridge Regression) 来得到一个封闭形式的解。此外 GR 算法中低分辨率图像特征空间中的 K 近邻 (KNN) 矩阵对应的就是稀疏编码算法中的低分辨率字典 $\boldsymbol{D_l}$ 。据此即可求出从低分辨率图像映射到高分辨率图像的映射矩阵。

GR 算法中的矩阵是预先算好的，并不会根据输入中的特征来调整，因此它的速度较快但是灵活性较差。ANR 相比于 GR就以提高计算量为代价来获得更好的灵活性。ANR 中首先对字典中的每一个原子 (atom) 通过KNN分组成一个个邻域，这样就可以为每个字典原子计算一个单独的投影矩阵，使用 GR 方法中同样的算法，只是把字典改成字典中的原子即可。

## 2. 算法的基本流程

实现通过稀疏字典学习超分辨率主要有以下几个步骤：

- 图像的预处理（转换颜色空间、切块、特征提取等操作）；
- 通过训练集学习用于稀疏表示的字典对；
- 求解最优化问题得到输入低分辨率图像块在字典下的稀疏表示；
- 利用此稀疏表示重构出高分辨率图像块；
- 对结果的后续处理（合成图像块等），得到最终的结果；

### 2.1 图像预处理

#### 变换图像颜色空间

通常我们得到的是 RGB 颜色空间下的彩色图像，由于人眼对于高频的亮度成分 (luminance component) 变化的敏感度要比高频颜色成分 (color component) 变化的敏感度高得多。因此需要通过将 RGB 图像变换到 YCbCr 颜色空间来分离图像的亮度成分和颜色成分，其中 YCbCr 中 Y 通道提供图片的亮度变换信息（基本上是原图像的灰度图），而 Cb 和 Cr 通道分别为蓝色和红色的浓度偏移量成分，提供图片的颜色成分。

这样在后续处理时我们就可以只对图像的 Y 通道进行较为复杂的基于字典学习超分辨率重构，而对于 Cb 和 Cr 通道仅仅进行较为简单的双三次插值 (Bicubic) 重构即可得到很好的效果。该操作直接通过 Matlab 函数 `rgb2ycbcr` 即可实现。

#### 图像切块 (Image Patch)

对于去噪、超分辨率等图像处理方法，在更小的尺寸上处理具有更好的效果，图片在局部相关性更强，更容易得到一些微小的特征，得到更加精细的细节信息。同时图像分块后得到的都是相同尺寸（如3×3或5×5）的图像块，便于编程统一。在这两篇文章中对图像分块的同时也让相邻的图像块有几个像素的重叠（overlap）部分，能得到更好的效果，避免相邻的图像块在处理后出现不连续的情况。

在后续的重构过程中，我们都是直接使用低分辨率的图像块去重构出高分辨率图像中对应位置上的图像块，最后再将图像块拼接在一起得到最终结果。

#### 图像特征提取

图像的特征提取主要是提取图像中一些关键的点、边缘或者特定的形状，即图像中的一些高频的细节信息。具体在图像超分辨率重构问题上，提取低分辨率图像中的不同特征可以提高预测精度，得到更加准确的高分辨率图像。

在进行特征提取之前，一般要对数据进行规范化处理，通过减去图像块的均值并除以标准差，使得图像的均值为 0 ，方差为 1。

提取图像特征的最简单的方法之一可以将图像通过一个高通滤波器，来提取图像的边缘信息。也可以通过一组高斯滤波器进行特征提取。

在 Yang 的算法中使用了图像的一阶和二阶梯度作为图像特征，而 Radu Timeofte 的算法中在图像一阶和二阶梯度的基础上又使用主成分分析 (PCA) 的方法对特征进行降维处理，在几乎不损失图像能量的条件下大幅降低特征的维数。

### 2.2 学习字典的方法

稀疏表示重构算法需要输入所谓的字典 $D$，计算出输入图像在该字典上的稀疏表示$\alpha$。

字典可以仅仅通过输入图像进行学习，这样学得的字典叫“内部字典”(internal dictionary)，这是根据图像切块后图像块中的冗余信息学得的。字典也可以通过输出以外多种图像进行学习，叫做“外部字典”(external dictionary)。字典可以通过随机采样得到，也可以通过对训练集学习得到。通过学习得到的字典在得到与随机采样字典相似效果时具有更小的字典尺寸，在进行超分辨率时也就具有更快地速度。

#### 单一字典学习 (Single Dictionary Training)：

从训练集学习到一个过完备字典的方法是求解以下这个最优化问题：

$$
\boldsymbol{D} = \arg \min_{\boldsymbol{D},Z}||X-\boldsymbol{D}Z||_2^2+\lambda||Z||_1
\\s.t. ||D||_2^2\leq1,i=1,2,\ldots,K
$$

其中$X=\{x_1,x_2,\ldots,x_t\}$ 为训练集， $\boldsymbol{D}$ 即为要学到的字典，$Z$ 为训练集中的样本在字典下的稀疏表示组成的向量。求解这个最优化问题可以通过变量交替优化的策略来实现。首先，将字典 $\boldsymbol{D}$ 初始化为一个高斯随机矩阵，并固定住字典，根据训练集和初始字典求出稀疏表示向量 $Z$ 的处值；然后固定住 $Z$ ，更新字典 $\boldsymbol{D}$ ；不断迭代更新字典 $\boldsymbol{D}$ 和 $Z$ ，最终可以求得字典  $\boldsymbol{D}$ 。常用的求解方法有基于逐列更新策略的 KSVD方法。

#### 联合字典学习 (Joint Dictionary Training)：

根据算法的基本假设：低分辨率图像和其对应的高分辨率图像在各自的字典 ($\boldsymbol{D_l}, \boldsymbol{D_h})$ 下具有相同的稀疏表示，为了学到这样的字典，需要在学习时同时输入低分辨率图像和高分辨率图像，同时训练两个字典。

$$
\boldsymbol{D_h} = \arg \min_{\boldsymbol{D_h},Z}||X^h-\boldsymbol{D_h}Z||_2^2+\lambda||Z||_1
\\\boldsymbol{D_l} = \arg \min_{\boldsymbol{D_l},Z}||X^l-\boldsymbol{D_l}Z||_2^2+\lambda||Z||_1
$$

结合这两个最优化问题为：

$$
\min_{\boldsymbol{D_h},\boldsymbol{D_l},Z}\frac{1}{N}||X^h-\boldsymbol{D_h}Z||_2^2+\frac{1}{M}||X^l-\boldsymbol{D_l}Z||_2^2+\lambda(\frac{1}{N}+\frac{1}{M})||Z||_1
$$

其中 $M,N$ 分别是低分辨率图像和高分辨率图像向量形式的维数，上式能够简写成：

$$
\min_{\boldsymbol{D_h},\boldsymbol{D_l},Z}||X_c-\boldsymbol{D_c}Z||_2^2+\lambda(\frac{1}{N}+\frac{1}{M})||Z||_1
\\X_c = [\begin{smallmatrix}\frac{1}{N}X^h\\\frac{1}{M}X^l \end{smallmatrix}]\quad \boldsymbol{D}_c = [\begin{smallmatrix}\frac{1}{N}\boldsymbol{D}_h\\\frac{1}{M}\boldsymbol{D}_l \end{smallmatrix}]
$$

这样即可使用上述的单字典学习的方法来学习字典对。

## 参考文献

- [1] Yang, Jianchao, et al. "Image super-resolution via sparse representation." IEEE transactions on image processing 19.11 (2010): 2861-2873.
- [2] Timofte, Radu, Vincent De Smet, and Luc Van Gool. "Anchored neighborhood regression for fast example-based super-resolution." Proceedings of the IEEE international conference on computer vision. 2013.
- [3] Chang, Hong, Dit-Yan Yeung, and Yimin Xiong. "Super-resolution through neighbor embedding." *Proceedings of the 2004 IEEE Computer Society Conference on Computer Vision and Pattern Recognition, 2004. CVPR 2004.*. Vol. 1. IEEE, 2004.

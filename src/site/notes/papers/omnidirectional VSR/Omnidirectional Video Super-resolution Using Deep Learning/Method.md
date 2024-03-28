---
{"dg-publish":true,"permalink":"/papers/omnidirectional VSR/Omnidirectional Video Super-resolution Using Deep Learning/Method/"}
---

# Architecture
![Pasted image 20240302151410.png](/img/user/asserts/Pasted%20image%2020240302151410.png)
在时间步$t$ 中，以滑动窗口的方式输入为$F_{t-1}、F_t、F_{t+1}$三个连续的低分辨率视频帧。
循环网络中上次迭代的结果$\text{HR}_{t-1}、\text{Hidden State}$作为输入

1. $F_{t-1}、F_t、F_{t+1}$经过`360° Feature Extractor`模块获取到$G_{\text{local}}$ 
2. $F_t、\text{HR}_{t-1}、\text{Hidden State}$作为`Recurrent Fusion`的输入，得到$G_{\text{global}}$
3. $G_{text{local}}、G_{\text{local}}$作为输入到`Dual-Duct Residual Blocks`中，得到隐藏状态$\text{Hidden State}$和两组特征$Feature_1、Feature_2$
4. $Feature_1、Feature_2$作为经过像素洗牌运算，对360°特征图进行深度-空间转换
5. 将得到的 ×4 特征图作为残差，逐元素添加到双线性插值的目标帧中，最终生成本页底部所示的 ×4 超分辨等方位帧 HR


# 360° Feature Extractor


- **当物体在连续帧之间移动时，同一物体内不同区域的失真也可能变化，使得进行光流估计具有挑战性**

![Pasted image 20240312121439.png](/img/user/asserts/Pasted%20image%2020240312121439.png)
![Pasted image 20240325161217.png](/img/user/Pasted%20image%2020240325161217.png)
## 1. Co-joint Feature Extraction

直接从每个等距投影（ERP）帧中提取的**初始二维特征**会根据在局部邻域中**所有三帧的联合特征**进行精化，这使得可以提取互相关的局部特征。
![Pasted image 20240325171559.png](/img/user/Pasted%20image%2020240325171559.png)
对于每对相邻帧（$F_{t−1}$ 和 $F_{t+1}$），相应特征会根据目标帧（$F_t$）的特征进一步精化。
![Pasted image 20240325171608.png](/img/user/Pasted%20image%2020240325171608.png)
最终提取的两组特征然后融合以生成由式（1）中的FinalFeaturet表示的单组局部特征。
![Pasted image 20240325171752.png](/img/user/Pasted%20image%2020240325171752.png)
其中$Conv 3×3$是具有3×3滤波器大小的卷积操作，‘$\bigodot$’表示沿特征深度的串联操作，‘$\bigoplus$’表示逐元素加法操作，$ReLU$表示修正线性单元。

## 2. Spatial Attention followed by Channel Attention

在第二阶段，为了应对ERP帧中存在的畸变而引起的空间差异，使用**空间注意机制**，为扭曲的ERP帧内的不同空间区域分配不同的注意力。

![Pasted image 20240325175709.png](/img/user/Pasted%20image%2020240325175709.png)

$\bigotimes$表示逐元素乘法操作，$σ$表示`sigmoid`激活函数

$$
\text{sigmoid}(\cdot) = \frac{1}{1+e^{-(\cdot)}}
$$
> sigmoid 可以将实数映射到 0-1 之间

# Recurrent Global Fusion

该模块以上一帧的输出$\text{HR}_{t-1}$、隐藏状态$h_{t-1}$和目标帧$F_t$作为输入，其中$\text{HR}_{t-1}$需要进行空间到深度转换（Space-to-Depth Transformation），来获取一组融合特征。

![Pasted image 20240312122704.png](/img/user/asserts/Pasted%20image%2020240312122704.png)

![Pasted image 20240325180556.png](/img/user/Pasted%20image%2020240325180556.png)
> ↓ represents a pixel-unshuffle operation.

# Dual-Duct Refinement

![Pasted image 20240326102441.png](/img/user/Pasted%20image%2020240326102441.png)
> 图7为作者提出的一个双导多信息交换残差块


该模块的输入为上一层的输出$G_{\text{global}}、G_{\text{local}}$

通过一个残差卷积网络进行细化。该网络由十个残差块组成，每个块包含一个双通道，用于同时学习ERP帧之间的全局和局部关系。

![Pasted image 20240326102549.png](/img/user/Pasted%20image%2020240326102549.png)

# Upsampling

在残差块中的细化操作之后，来自局部和全局通道的两个结果，即$LF_t$和$GF_t$，分别经过3×3卷积操作，然后进行**深度到空间的转换**，将×4特征深度转换为空间数据.
![Pasted image 20240326103113.png](/img/user/Pasted%20image%2020240326103113.png)
> 对于深度到空间的转换，S3PO模型使用像素洗牌操作。
> pixel shuffle
> 在深度学习中，Pixel shuffle通常作为一种上采样技术被集成到神经网络中，用于增加特征图的尺寸。通过Pixel shuffle，神经网络可以学习到如何有效地将低分辨率的特征图转换为高分辨率的特征图，从而提高图像重建和超分辨率的效果。

这两个输出沿深度连接，并通过两个3×3卷积层，得到×4超分辨率残差。

超分辨率残差与双线性插值的LR帧逐元素相加，用于给定$F_t$的最终HR重建（$F_t × 4$）。
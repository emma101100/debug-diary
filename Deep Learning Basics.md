# 深度学习基础学习笔记

## 线性模型：神经网络的起点
- **定义**：
  线性模型是神经网络的基本计算单元，通过公式 \( y = Wx + b \) 将输入 \( x \) 映射到输出 \( y \)。
  - \( x \): 输入向量（如 MNIST 图像展平为 784 维）。
  - \( W \): 权重矩阵（如 \( 10 \times 784 \)，每行对应一个类别的权重）。
  - \( b \): 偏置向量（如 10 维，每类一个偏置）。
  - \( y \): 输出向量（如 10 维得分，称为 logits）。
- **计算过程**：
  - 输入 \( x \) 与权重 \( W \) 做矩阵乘法，得到 \( Wx \)。
  - 加上偏置 \( b \)，输出 \( y \)。
  - 例如：一张 28×28 图像展平为 \( x = [0.1, 0.2, ..., 0.7] \)，通过 \( W \) 和 \( b \)，得到 \( y = [3.5, 1.2, ..., 0.8] \)，表示 10 个类别的得分。
- **作用**：
  - 线性模型通过权重组合输入特征，生成初步表示。
  - 局限：只能学习线性关系（如直线分隔），无法处理复杂模式（如图像的非线性分布）。
- **类比 OpenCV**：
  - 类似卷积核对图像像素加权求和（如 Sobel 边缘检测），但 OpenCV 的核是手动设计的，而线性模型的 \( W \) 和 \( b \) 通过数据训练自动学习。
- **细节**：
  - 权重 \( W \) 的每一行代表一个类别的“模板”，与输入的内积衡量匹配程度。
  - 偏置 \( b \) 调整输出基线，防止模型强制通过原点。
- **联系其他知识点**：
  - 输出 \( y \) 通常作为 **激活函数**（隐藏层）或 **Softmax**（输出层）的输入。
  - 权重和偏置的优化依赖 **交叉熵损失**、**反向传播**和 **梯度下降**。
  - 在 CNN 中，卷积操作可以看作局部线性模型，权重共享减少参数量。

---

## Softmax 分类器：从得分到概率
- **定义**：
  Softmax 分类器将线性模型的输出（logits）转为概率分布，用于多分类任务。
  - 公式：\( P(y=i) = \frac{e^{z_i}}{\sum_{j=1}^n e^{z_j}} \)
    - \( z_i \): 第 \( i \) 个类别的得分（如 \( z = Wx + b \)）。
    - 输出：概率值在 0-1 之间，总和为 1。
- **计算过程**：
  - 输入得分 \( z = [3.5, 1.2, 0.8] \)。
  - 计算指数：\( e^{3.5} \approx 33.115 \), \( e^{1.2} \approx 3.320 \), \( e^{0.8} \approx 2.225 \)。
  - 归一化：总和 \( 33.115 + 3.320 + 2.225 = 38.660 \)，概率为 \( [0.856, 0.086, 0.058] \)。
  - 结果：第 1 类概率最高，模型倾向预测为该类。
- **作用**：
  - 将“生硬”的得分转化为直观的概率，方便分类决策（如判断图像是“猫”还是“狗”）。
  - 指数运算放大高分差异，增强区分度。
- **类比 OpenCV**：
  - 类似像素值归一化（如除以 255 得到 0-1），但 Softmax 通过指数运算突出高分，类似增强对比度。
- **细节**：
  - Softmax 常用于输出层，确保概率总和为 1。
  - 对数值稳定性敏感（如大值溢出），实现时常减去最大值 \( z_i - \max(z) \)。
- **联系其他知识点**：
  - 输入 \( z \) 来自 **线性模型**。
  - 输出概率用于计算 **交叉熵损失**，评估模型表现。
  - 在 CNN 输出层，Softmax 整合全连接层特征，生成分类概率。

---

## 交叉熵损失：量化预测误差
- **定义**：
  交叉熵损失衡量 Softmax 输出概率与真实标签的差距，是分类任务的标准评估指标。
  - 公式：\( L = -\sum_{i=1}^n y_{\text{true},i} \cdot \log(P(y=i)) \)
    - \( y_{\text{true}} \): 真实标签（如 \( [1, 0, 0] \) 表示第 1 类）。
    - \( P(y=i) \): Softmax 预测概率。
- **计算过程**：
  - 例：真实标签 \( [1, 0, 0] \)，预测概率 \( [0.856, 0.086, 0.058] \)。
  - 损失：\( L = -\log(0.856) \approx 0.155 \)。
  - 若预测错误，如 \( [0.1, 0.6, 0.3] \)，损失为 \( -\log(0.1) \approx 2.303 \)，表明模型表现差。
- **作用**：
  - 损失值越小，预测越接近真实，指导模型优化。
  - 提供误差信号，告诉模型“错在哪里”。
- **类比 OpenCV**：
  - 类似比较两张图像的像素差（如均方误差 MSE），但交叉熵比较概率分布，适合分类任务。
- **细节**：
  - 仅对正确类别的概率取对数，鼓励正确类得分最大化。
  - 常与 Softmax 结合，组成 Softmax-CrossEntropy 损失，便于梯度计算。
- **联系其他知识点**：
  - 使用 **Softmax** 的概率输出。
  - 通过 **反向传播** 计算梯度，传递给 **梯度下降**。
  - 在 CNN 和 RNN 中，交叉熵损失同样用于分类任务。

---

## 梯度下降：优化模型的引擎
- **定义**：
  梯度下降通过计算损失对参数（\( W \), \( b \)）的梯度，沿梯度反方向更新参数，减小损失。
  - 更新公式：
    - \( W_{\text{new}} = W_{\text{old}} - \eta \cdot \frac{\partial L}{\partial W} \)
    - \( b_{\text{new}} = b_{\text{old}} - \eta \cdot \frac{\partial L}{\partial b} \)
    - \( \eta \): 学习率，控制更新幅度。
- **工作原理**：
  - 梯度 \( \frac{\partial L}{\partial W} \) 指向损失增加方向，更新走反方向（下坡）。
  - 学习率 \( \eta \) 平衡速度与稳定性：过大可能跳过最优解，过小收敛慢。
- **变种**：
  - **批量梯度下降（Batch GD）**：用全数据集计算梯度，稳定但计算量大。
  - **随机梯度下降（SGD）**：每次用一个样本，噪声大但更新快。
  - **小批量梯度下降（Mini-batch GD）**：折中方案，常用。
- **例子**：
  - 若 \( \frac{\partial L}{\partial w_{1,1}} = 0.3 \)，学习率 \( \eta = 0.01 \)，则 \( w_{1,1}^{\text{new}} = w_{1,1}^{\text{old}} - 0.01 \cdot 0.3 \)。
- **类比 OpenCV**：
  - 类似手动调整滤波器参数（如高斯核大小）时的试错，梯度下降用数学自动化优化。
- **细节**：
  - 学习率需调优，常用自适应方法（如 Adam 优化器）。
  - 梯度消失/爆炸可能影响深层网络，需正则化或残差连接。
- **联系其他知识点**：
  - 依赖 **交叉熵损失** 提供误差。
  - 梯度由 **反向传播** 计算。
  - 在 **神经网络** 和 **CNN** 中逐层优化参数。

---

## 反向传播：梯度计算的核心
- **定义**：
  反向传播（Backpropagation）从损失函数开始，基于链式法则逆向计算每个参数的梯度，指导优化。
- **详细流程**：
  1. **前向传播**：
     - 输入 \( x \) 通过线性模型 \( z = Wx + b \)。
     - 经激活函数或 Softmax，得到预测概率。
     - 计算交叉熵损失 \( L \)。
  2. **反向传播**：
     - 从损失 \( L \) 开始，计算输出层梯度 \( \frac{\partial L}{\partial z} \)。
     - 逐层向后，计算权重梯度 \( \frac{\partial L}{\partial W} = \frac{\partial L}{\partial z} \cdot \frac{\partial z}{\partial W} \)。
     - 同理计算偏置梯度 \( \frac{\partial L}{\partial b} \)。
  3. 用梯度下降更新参数。
- **例子**：
  - 预测概率 \( [0.6, 0.4] \)，真实标签 \( [1, 0] \)。
  - 损失梯度：\( \frac{\partial L}{\partial z} = [0.6 - 1, 0.4 - 0] = [-0.4, 0.4] \)。
  - 进一步推导到 \( W \), \( b \)，调整权重让正确类概率更高。
- **类比 OpenCV**：
  - 类似调试程序，从错误结果倒查代码，找出每个步骤的影响。
  - 反向传播自动化这一过程，精确定位权重问题。
- **细节**：
  - 链式法则确保梯度逐层传递，计算复杂度与网络深度相关。
  - 激活函数导数（如 ReLU 的 0 或 1）影响梯度传递。
- **联系其他知识点**：
  - 连接 **交叉熵损失** 和 **梯度下降**。
  - 在 **神经网络架构** 中逐层工作。
  - 在 CNN 中，反向传播计算卷积核梯度。

---

## 激活函数：非线性的关键
- **定义**：
  激活函数对线性输出 \( z = Wx + b \) 施加非线性变换 \( a = f(z) \)，增强模型学习复杂模式的能力。
- **为什么需要**：
  - 线性模型只能学直线关系，多层线性变换仍为线性。
  - 激活函数引入非线性，让网络学习复杂模式（如图像的曲线分布）。
- **常见类型**：
  - **ReLU**：\( f(z) = \max(0, z) \)，隐藏层常用，简单高效。
  - **Sigmoid**：\( \sigma(z) = \frac{1}{1 + e^{-z}} \)，输出 0-1，适合二分类。
  - **Tanh**：\( \tanh(z) = \frac{e^z - e^{-z}}{e^z + e^{-z}} \)，输出 -1 到 1，适合隐藏层。
  - **Softmax**：输出概率，适合多分类输出层。
- **例子**：
  - 输入 \( z = [-0.5, 1.2, 0.3] \)。
  - ReLU 输出：\( [0, 1.2, 0.3] \)，负值置零。
  - Sigmoid 输出：\( [0.378, 0.769, 0.574] \)，压缩到 0-1。
- **类比 OpenCV**：
  - 类似阈值处理（如二值化），ReLU 像“只保留亮像素”，但更灵活。
  - Sigmoid 像平滑过渡，保留强度信息。
- **细节**：
  - ReLU 避免梯度消失，但可能导致“死神经元”（负值恒为 0）。
  - Sigmoid 压缩输出，梯度小，深层易消失。
- **联系其他知识点**：
  - 应用于 **线性模型** 输出。
  - 导数用于 **反向传播** 计算梯度。
  - 在 **神经网络** 和 **CNN** 的隐藏层增强表达。

---

## 神经网络架构：从输入到输出
- **定义**：
  神经网络由多层节点组成，包括输入层、隐藏层和输出层，通过前向传播计算预测，反向传播优化参数。
- **结构**：
  - **输入层**：接收原始数据（如 784 维图像向量）。
  - **隐藏层**：多层线性模型 + 激活函数，提取特征。
  - **输出层**：生成结果（如 Softmax 概率）。
- **工作流程**：
  - **前向传播**：数据从输入到输出，逐层计算。
  - **反向传播**：误差从输出到输入，更新参数。
- **例子**：
  - 输入：28×28 图像（784 维）。
  - 隐藏层 1：128 神经元（线性 + ReLU）。
  - 隐藏层 2：64 神经元（线性 + ReLU）。
  - 输出层：10 神经元（Softmax），输出 \( [0.9, 0.05, ..., 0.01] \)，预测“0”。
- **类比 OpenCV**：
  - 类似图像处理流水线（滤波 → 边缘检测 → 特征提取），但神经网络自动学习每一步。
- **细节**：
  - 隐藏层越多，模型越深，表达能力越强，但计算复杂。
  - 过拟合风险需正则化（如 Dropout）。
- **联系其他知识点**：
  - 核心组件：**线性模型**、**激活函数**、**Softmax**。
  - 训练机制：**交叉熵损失**、**反向传播**、**梯度下降**。
  - CNN 和 RNN 是其特化形式。

---

## 卷积神经网络（CNN）：图像处理的利器
- **定义**：
  CNN 是一种针对图像等结构化数据的神经网络，通过卷积操作提取局部特征，适合分类、检测等任务。
- **结构**：
  1. **输入层**：接收图像（如 224×224×3 的 RGB 数据）。
  2. **卷积层**：用卷积核提取特征（如边缘、纹理）。
  3. **激活函数层**：引入非线性（如 ReLU）。
  4. **池化层**：缩小特征图，保留关键信息。
  5. **全连接层**：整合全局特征，映射到分类。
  6. **输出层**：生成概率（如 Softmax）。
- **卷积计算**：
  - 卷积核（如 3×3）在输入上滑动，计算点积，生成特征图。
  - 公式：\( \text{Output}_{i,j} = \sum_{m,n} \text{Input}_{i+m,j+n} \cdot \text{Kernel}_{m,n} + b \)。
  - 参数共享：同一核权重复用，减少参数量。
- **池化**：
  - 最大池化：选窗口内最大值，突出强特征。
  - 平均池化：取平均值，平滑特征。
- **例子**：
  - 输入：32×32×3 图像。
  - 卷积层：32 个 3×3 核，输出 30×30×32 特征图。
  - ReLU：激活特征。
  - 池化层：2×2 最大池化，输出 15×15×32。
  - 全连接层：展平后映射到 10 类，Softmax 输出概率。
- **类比 OpenCV**：
  - 卷积类似滤波（如高斯模糊），池化类似下采样，但 CNN 自动学习核和参数。
- **细节**：
  - 填充（padding）控制输出尺寸，步幅（stride）影响下采样。
  - 感受野随层加深扩大，深层捕获高级特征。
- **联系其他知识点**：
  - 卷积层基于 **线性模型**，局部计算。
  - **激活函数** 和 **池化** 增强表达和效率。
  - 依赖 **高带宽显存** 加速计算。
  - **反向传播** 优化卷积核。

---

## 池化和下采样：降维与鲁棒性
- **定义**：
  - **池化**：通过固定窗口（如 2×2）操作缩小特征图，保留关键信息。
  - **下采样**：广义降维方法，池化是主要形式。
- **池化类型**：
  - **最大池化**：选最大值，突出强特征（如边缘）。
  - **平均池化**：取平均值，平滑特征。
- **计算过程**：
  - 窗口在特征图上滑动（步幅控制间隔）。
  - 例：4×4 特征图，2×2 最大池化（步幅 2），输出 2×2。
- **逐步池化**：
  - 多次小窗口池化（如 2×2），而非一次大窗口（如 8×8）。
  - 原因：
    1. 保留细节，逐步浓缩信息。
    2. 匹配 CNN 层次，从低级到高级特征。
    3. 控制感受野增长，早期关注局部，深层看全局。
    4. 平衡计算与性能。
- **作用**：
  - 减少计算量和内存需求。
  - 扩大感受野，增强对偏移、噪声的鲁棒性。
- **类比 OpenCV**：
  - 类似图像缩放或降采样（如金字塔），但池化选择性保留强信号。
- **细节**：
  - 池化无参数，降低过拟合风险。
  - 步幅大于 1 时，池化等效下采样。
- **联系其他知识点**：
  - 配合 **卷积层**，优化 **CNN 架构**。
  - 依赖 **高带宽显存** 加速。
  - **反向传播** 计算池化梯度。

---

## 高带宽显存与缓存：硬件加速
- **定义**：
  - **高带宽显存（VRAM）**：GPU 专用内存，存储输入、权重、特征图。
  - **缓存**：GPU 内部高速内存，靠近计算核心，存储常用数据。
- **作用**：
  - **显存**：高带宽传输大块数据（如图像、特征图），减少核心等待。
  - **缓存**：复用卷积核权重或局部像素，减少显存访问。
  - 协同作用：核心持续计算，数据延迟最小化。
- **例子**：
  - 卷积计算频繁读写 3×3 核权重，缓存存储核数据，显存传输整张特征图。
- **类比 OpenCV**：
  - 类似优化图像处理（如并行滤波），显存和缓存提升数据吞吐量。
- **细节**：
  - 显存带宽远超 CPU DRAM（如 GDDR6 达 600 GB/s）。
  - 缓存容量小，需高效利用局部性（如卷积的滑动窗口）。
- **联系其他知识点**：
  - 加速 **卷积计算** 和 **池化**。
  - 支持 **CNN 层次结构** 的多层运算。
  - **参数共享** 减少权重存储，配合缓存提效。

---

## 参数共享：卷积效率的关键
- **定义**：
  参数共享指卷积核权重在整个输入上复用，不随位置变化。
- **原理**：
  - 一个 3×3×C_{\text{in}} 核有固定权重（如 27 个参数）。
  - 滑动时，权重不变，仅输入区域变化。
  - 输出特征图每个点共享核参数。
- **对比全连接**：
  - 全连接层（如线性模型）为每个输出分配独立权重，参数量随输入尺寸暴增。
  - 卷积层参数仅与核大小和通道数相关，如 \( K \times K \times C_{\text{in}} \times C_{\text{out}} \)。
- **例子**：
  - 32×32×3 输入，3×3×3×64 卷积核，参数量为 \( 9 \times 3 \times 64 = 1728 \)。
  - 全连接层同等规模需 \( 32 \times 32 \times 3 \times 64 \approx 196608 \) 参数。
- **作用**：
  - 减少参数量，降低内存和计算需求。
  - 假设特征空间一致性，增强泛化能力。
- **类比 OpenCV**：
  - 类似同一滤波器（如 Sobel 核）应用于整张图像，卷积核自动学习。
- **联系其他知识点**：
  - 优化 **卷积计算**，减少 **显存** 需求。
  - 支持 **CNN 层次结构** 深层堆叠。
  - **反向传播** 更新共享权重。

---

## 经典 CNN 架构：从 AlexNet 到 ResNet
- **AlexNet**：
  - **结构**：8 层（5 卷积 + 3 全连接），大核（11×11、5×5）。
  - **特点**：ReLU 加速训练，最大池化降维，Dropout 防过拟合。
  - **作用**：证明 CNN 优于传统方法，奠定分类基础。
- **VGG**：
  - **结构**：16/19 层，全用 3×3 核，深层规整。
  - **特点**：小核堆叠效果媲美大核，参数少，适合特征提取。
  - **作用**：强化深层设计，广泛用于迁移学习。
- **ResNet**：
  - **结构**：50/101/152 层，残差连接（输出 = 输入 + 卷积）。
  - **特点**：解决深层退化，Batch Normalization 稳定训练，全局平均池化替代全连接。
  - **作用**：突破深度瓶颈，适合复杂任务。
- **类比 OpenCV**：
  - 类似优化图像处理流水线，AlexNet 开路，VGG 细化，ResNet 智能跳跃。
- **联系其他知识点**：
  - 基于 **CNN 架构**，集成 **卷积**、**池化**、**激活函数**。
  - 依赖 **预训练** 和 **高带宽显存**。
  - **反向传播** 优化深层参数。

---

## 循环神经网络（RNN）：序列建模专家
- **定义**：
  RNN 通过循环结构处理序列数据，隐藏状态记录历史信息。
- **结构**：
  - 输入：序列（如单词列表）。
  - 隐藏状态：每步更新，公式 \( h_t = f(W_h h_{t-1} + W_x x_t + b) \)。
  - 输出：每步或末尾输出（如分类）。
- **变种**：
  - **LSTM**：引入遗忘门、输入门，解决长序列遗忘。
  - **GRU**：简化 LSTM，效率更高。
- **例子**：
  - 输入文本“我爱学习”，RNN 逐词处理，预测下一词“深度”。
- **作用**：
  - 捕捉时间或顺序依赖，适合文本、语音、时间序列。
- **类比 OpenCV**：
  - 类似视频帧间分析，RNN 关注序列前后关系。
- **细节**：
  - 顺序计算不可并行，效率低于 CNN。
  - 梯度消失限制长序列建模。
- **联系其他知识点**：
  - 类似 **神经网络**，特化序列。
  - 可结合 **预训练** 初始化。
  - **注意力机制** 改进长序列处理。

---

## 预训练与迁移学习
- **定义**：
  预训练在大数据集上训练模型，生成通用参数，微调适配特定任务。
- **过程**：
  - **预训练**：用 ImageNet（图像）或语料库（文本）学习特征。
  - **微调**：用小数据集调整参数，适配任务（如情感分析）。
- **例子**：
  - ResNet 在 ImageNet 预训练，微调用于医学图像分类。
- **作用**：
  - 加速收敛，减少训练时间。
  - 提升性能，尤其在小数据集上。
- **类比 OpenCV**：
  - 类似用预训练 Haar 特征加速目标检测。
- **细节**：
  - 需大算力支持，冻结部分层可减少微调成本。
  - 泛化能力强，但可能引入偏差。
- **联系其他知识点**：
  - 用于 **CNN**（如 ResNet）、**RNN**。
  - 启发 **注意力机制** 的预训练（如 BERT）。
  - 依赖 **高带宽显存** 支持大数据训练。

---

## 注意力机制：动态聚焦
- **定义**：
  注意力机制通过为序列元素分配权重，动态聚焦关键信息，增强建模能力。
- **Self-Attention**：
  - 每个元素与序列中所有元素交互，生成上下文相关表示。
  - 公式：\( \text{Attention}(Q, K, V) = \text{softmax}(\frac{QK^T}{\sqrt{d_k}})V \)。
- **QKV（Query, Key, Value）**：
  - **Query**：查询向量，计算关注点。
  - **Key**：键向量，衡量相关性。
  - **Value**：值向量，提供特征。
  - 计算：Q 与 K 内积生成权重，权重加权 V 得到输出。
- **例子**：
  - 句子“我爱学习深度学习”，Self-Attention 让“深度”更关注“学习”。
- **作用**：
  - 解决 RNN 长序列遗忘，支持并行计算。
  - 动态建模上下文，适应多义性。
- **类比 OpenCV**：
  - 类似图像中关注显著区域（如人脸），注意力动态选择关键特征。
- **细节**：
  - 计算复杂度高（\( O(n^2) \)，n 为序列长度）。
  - 多头注意力（Multi-Head）捕获多维关系。
- **联系其他知识点**：
  - 改进 **RNN**，核心于 **Transformer**。
  - 取代 **Word2Vec** 静态表示。
  - 依赖 **预训练** 提升性能。

---

## Word2Vec 不足与 Self-Attention
- **Word2Vec**：
  - **定义**：预训练词向量，将词映射为固定向量（如 300 维）。
  - **不足**：
    - 静态表示：词向量不随上下文变化（如“bank”在“河岸”和“银行”相同）。
    - 无法捕捉多义性。
    - 缺乏序列交互。
- **Self-Attention 改进**：
  - 动态生成上下文相关表示。
  - 全局交互，每个词考虑其他词。
  - 例：“bank”在“river bank”中偏向“岸”，在“bank account”中偏向“银行”。
- **类比 OpenCV**：
  - Word2Vec 像固定特征描述子，Self-Attention 像动态特征匹配。
- **联系其他知识点**：
  - Word2Vec 是早期 **预训练**。
  - Self-Attention 用于现代 **Transformer**，通过 **QKV** 实现。

---

## 总结与知识点联系

### 核心知识点

1. **线性模型**：
   - 基础计算单元，公式 \( y = Wx + b \)，生成得分。
   - 局限：仅限线性，需激活函数增强。
   - 例：MNIST 图像到 10 类得分。
2. **Softmax 分类器**：
   - 将得分转为概率，公式 \( P(y=i) = \frac{e^{z_i}}{\sum_j e^{z_j}} \)。
   - 作用：直观分类，放大高分。
   - 例：得分 \( [3.5, 1.2, 0.8] \) 转为 \( [0.856, 0.086, 0.058] \)。
3. **交叉熵损失**：
   - 衡量预测与真实差距，公式 \( L = -\sum_i y_{\text{true},i} \cdot \log(P(y=i)) \)。
   - 驱动优化，损失小表示预测准。
   - 例：正确概率 0.856，损失 0.155。
4. **梯度下降**：
   - 更新参数，公式 \( W_{\text{new}} = W_{\text{old}} - \eta \cdot \frac{\partial L}{\partial W} \)。
   - 变种：批量、随机、小批量。
   - 例：梯度 0.3，学习率 0.01，更新权重。
5. **反向传播**：
   - 从损失逆向计算梯度，基于链式法则。
   - 流程：前向计算损失 → 反向推导梯度 → 更新参数。
   - 例：概率误差 [-0.4, 0.4] 推到权重。
6. **激活函数**：
   - 引入非线性，如 ReLU \( \max(0, z) \)、Sigmoid \( \frac{1}{1 + e^{-z}} \)。
   - 增强复杂模式学习。
   - 例：ReLU 将 [-0.5, 1.2] 转为 [0, 1.2]。
7. **神经网络架构**：
   - 输入 → 隐藏层（线性 + 激活）→ 输出层（Softmax）。
   - 前向预测，反向优化。
   - 例：MNIST 网络输出“0”概率 0.9。
8. **卷积神经网络（CNN）**：
   - 针对图像，卷积提取局部特征，池化降维。
   - 结构：卷积 → ReLU → 池化 → 全连接 → Softmax。
   - 例：32×32 图像到 10 类概率。
9. **池化与下采样**：
   - 池化缩小特征图，最大池化突出特征。
   - 逐步池化保留细节，匹配层次递进。
   - 例：4×4 特征图池化为 2×2。
10. **高带宽显存与缓存**：
    - 显存快速传输，缓存复用局部数据。
    - 加速卷积、池化。
    - 例：缓存存储 3×3 核权重。
11. **参数共享**：
    - 卷积核权重复用，减少参数。
    - 对比全连接，效率高。
    - 例：3×3×3×64 核仅 1728 参数。
12. **经典 CNN 架构**：
    - AlexNet：大核 + ReLU，8 层。
    - VGG：小核深层，16/19 层。
    - ResNet：残差连接，50+ 层。
13. **循环神经网络（RNN）**：
    - 循环结构，捕捉序列依赖。
    - 变种：LSTM、GRU。
    - 例：预测“我爱学习”后为“深度”。
14. **预训练**：
    - 大数据集预学参数，微调适配任务。
    - 加速收敛，提升性能。
    - 例：ResNet 微调医学图像。
15. **注意力机制**：
    - 动态权重，聚焦关键信息。
    - Self-Attention 通过 QKV 建模上下文。
    - 例：“bank”随语境动态表示。
16. **Word2Vec 不足**：
    - 静态向量，难适多义。
    - Self-Attention 动态改进。
    - 例：“bank”在不同句子不同表示。

### 知识点联系
- **基础计算**：
  - **线性模型** 是神经网络、CNN、RNN 的核心，生成初步特征。
  - **激活函数**（如 ReLU）为线性输出加非线性，增强表达。
  - **Softmax** 将输出转为概率，服务分类任务。
- **训练闭环**：
  - **交叉熵损失** 量化误差，评估 **Softmax** 概率。
  - **反向传播** 从损失计算梯度，连接输出到权重。
  - **梯度下降** 用梯度更新参数，形成优化循环。
  - 整个流程（前向 → 损失 → 反向 → 更新）贯穿 **神经网络**。
- **图像优化（CNN）**：
  - **卷积计算** 提取局部特征，类似 **线性模型** 的局部版本。
  - **参数共享** 减少权重，降低 **显存** 需求。
  - **池化** 降维增鲁棒，逐步设计匹配 **层次结构**。
  - **高带宽显存** 和 **缓存** 加速卷积和池化。
  - **经典架构**（AlexNet → VGG → ResNet）优化结构，集成 **预训练**。
- **序列建模（RNN 与注意力）**：
  - **RNN** 捕捉序列依赖，类似 **神经网络** 的动态版本。
  - **预训练** 为 RNN 提供初始参数，类似 CNN 的迁移学习。
  - **注意力机制** 改进 RNN，动态聚焦，**Self-Attention** 通过 **QKV** 实现上下文建模。
  - **Word2Vec** 的静态表示被 Self-Attention 的动态表示取代。
- **硬件与效率**：
  - **高带宽显存** 和 **缓存** 支持 CNN 和 RNN 的大数据计算。
  - **参数共享**（CNN）与 **预训练**（CNN、RNN）优化计算效率。
- **整体逻辑**：
  - **数据流**：输入 → 特征提取（线性/卷积 + 激活/池化）→ 输出（Softmax/全连接）。
  - **优化流**：预测 → 损失（交叉熵）→ 梯度（反向传播）→ 更新（梯度下降）。
  - **架构演进**：
    - 神经网络：通用框架。
    - CNN：图像特化，卷积 + 池化。
    - RNN：序列特化，循环结构。
    - 注意力机制：动态建模，Transformer 核心。
  - **硬件支持**：显存、缓存加速训练。
  - **预训练**：贯穿 CNN、RNN、Transformer，提升性能。

### 总结
这份笔记从线性模型到神经网络、CNN、RNN，再到注意力机制，系统梳理了深度学习的核心组件和逻辑链条。**线性模型** 和 **激活函数** 构建基础计算，**Softmax** 和 **交叉熵损失** 驱动分类任务，**反向传播** 和 **梯度下降** 优化参数，形成神经网络的核心流程。**CNN** 通过 **卷积**、**池化** 和 **参数共享** 高效处理图像，**高带宽显存** 和 **缓存** 提供硬件支持，**经典架构**（如 ResNet）进一步突破性能瓶颈。**RNN** 针对序列数据，**预训练** 加速收敛，**注意力机制**（尤其是 **Self-Attention** 和 **QKV**）动态建模，克服 **Word2Vec** 的局限，开启 Transformer 时代。所有知识点围绕“特征提取 → 输出预测 → 误差优化”的逻辑紧密相连，硬件和预训练贯穿始终，构成深度学习的完整体系。

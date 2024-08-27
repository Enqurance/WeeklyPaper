# August 2024

#### [To Believe or Not to Believe Your LLM](https://arxiv.org/abs/2406.02543)

##### Yasin Abbasi Yadkori, Ilja Kuzborskij, et al.

本文提出了一种基于信息论度量的LLM幻觉检测方法，该方法能够分析模型输出时的认知不确定性，从而判断模型是否处于幻觉状态。本文方法的核心思想是，一个完美的模型不存在认知不确定性，而只存在数据不确定时，使用模型进行多次推理应当是独立同分布的。用随机变量表示每一次推理的结果，那么我们可以用一个联合概率分布表示多次推理的结果。根据链式法则，联合概率分布可以写作：
$$
P(X_1, X_2, ... , X_n) = \prod_{i=1}^{n}P(X_i|X_1,..X_{i-1})
$$
同时我们注意到，LLM可以按顺序接受输入，因此我们可以将多次推理整合到上下文中，模拟链式法则。理想的情况下，由于输出是独立同分布的，每一次推理不应受到上下文的影响。例如，询问英国的首都是哪里，不论上下文如何，模型应当总是回答“伦敦”。实际上，在加入多次“英国的首都是巴黎”的上下文时，模型偶尔会受到影响输出“巴黎”。因此，我们可以通过在语言模型推理上应用类似链式法则的提示词，来确定模型的认知不确定性，从而判断模型是否处于幻觉状态。在此基础上，本文提出了一个基于信息论度量的变量，用于在多次推理下估计认知不确定性。

本文与Semantic Entropy、P(True)等方法进行了对比实验，但是本文只提供了在Gemini 1.0上的实验结果。

#### [Semantic Entropy Probes: Robust and Cheap Hallucination Detection in LLMs](https://arxiv.org/abs/2406.15927)

##### Jannik Kossen, Jiatong Han, et al.

本文提出了一种基于语义熵的探针预测方法SEPs，可以快速、有效地识别LLM的幻觉行为。总体上来说，SEPs是在LLMs的隐藏层上训练的线性探针，也可以将SEPs看作一个二元线性回归模型。相较于Farquhar等人的语义熵方法，SEPs仅需要在模型的隐藏层上进行多次采样即可，而不需要多次推理。

SEPs的训练是基于一组二元组的，每一个元组的形式为$(h_{l}^{p}(x), H_{SE}(x))$ ，x表示输入，其中$h_{l}^{p}(x)$ 表示位置为p的Token在l层的隐藏层，$H_{SE}(x)$ 则表示对应的语义熵。

如何构建这些二元组训练数据？首先对于一个询问$x$ ，进行一次推理并且存储隐藏层，随后在隐藏层上进行$N=10$ 次采样，即可根据Farquhar等人提出的方法计算语义熵。

随后，由于我们要训练的是一个二分类模型，因此我们需要对输出进行二元化。二元化的方法启发自回归树：

![image-20240823152553033](https://raw.githubusercontent.com/Enqurance/Figures/main/202408231526089.png)

在本文中，所使用的隐藏层是输入最后一个Token后的隐藏层，以及输出前的第二个隐藏层。

[Semantic Density: Uncertainty Quantification in Semantic Space for Large Language Models](https://arxiv.org/abs/2405.13845)

##### Xin Qiu, Risto Miikkulainen

本研究指出两个关键的问题：

- 目前许多方法都仅仅在分类任务上适用，而在NLG任务上存在较多限制
- 许多方法只关注词和文本的不确定性，而忽略了语义空间的不确定性

目前，一个公开认为有效的方法是Semantic Entropy，然而这个方法有两个问题：

- 该方法仅对一个prompt计算不确定性，而不是针对每一个response，因此无法得知某一条response是否可信
- 该方法仅考虑生成回复（Response）语义上的等价性，而没有考虑更细粒度下的语义区别

为了填补这些研究的空白，本研究提出了一个新的不确定性度量框架，即语义密度（Semantic Density），该框架可以从语义密度的角度重建输出的概率分布。该框架有以下的优点：

1. 无需对LLM进行训练和微调
2. 对于任务类型没有限制，可以在各种生成式任务上适用
3. 返回面向Response的结果，而不是面向Prompt的结果
4. 考虑了Response之间细语义颗粒度的差异

Methodology

给定一个输入提示词$\vec{x}$与一个输出序列$\vec{y} = [y_{1},…,y_{L}]$，本研究的目标是为输出序列$\vec{y}$计算一个不确定性度量，该度量与序列$\vec{y}$的真实性存在单调关系。本研究所提出的度量值是和Response相关的。

本文定义了一个语义空间，并且使用范数来衡量$\vec{y}$与$\vec{x}$的语义相似程度，若为0则代表两者语义相同，$\frac{\sqrt{2}}{2}$ 则代表两者语义无关，1则代表两者语义相同。同时，该空间中也定义了语义的距离，更小的范数代表更近的语义距离。

对于语义空间中范数的计算，本研究使用Natural Language Inference(NLI)实现。

![image-20240827100112792](https://raw.githubusercontent.com/Enqurance/Figures/main/202408271001913.png)

SD估计

针对$\vec{x}$的输出$\vec{y}_{*}$，本文使用多次采样估计语义密度。对于每一个$\vec{y}_{i}$，可以从模型中获得Token的概率，进而只需要对$\vec{y}_{i}$采样一次，随后用链式法则求解SD中的条件概率即可。在实现过程中，还有一些其他的细节可参考原文。

![image-20240827100119903](https://raw.githubusercontent.com/Enqurance/Figures/main/202408271001948.png)
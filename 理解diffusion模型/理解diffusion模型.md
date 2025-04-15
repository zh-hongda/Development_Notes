# 扩散模型

- [扩散模型](#扩散模型)
  - [引言：Generative Models](#引言generative-models)
  - [证据下界（Evidence Lower Bound）](#证据下界evidence-lower-bound)
    - [推导 ELBO（证据下界）](#推导-elbo证据下界)
  - [**变分自编码器**](#变分自编码器)
  - [**层级变分自编码器**](#层级变分自编码器)



## 引言：Generative Models

给定从目标分布中观测到的样本 $ x $，生成模型的目标是学习其对应的真实数据分布 $p(x)$。一旦模型学习完成，我们就可以随时从近似模型中生成新的样本。

当前有几种常见的生成模型的研究方向，我们在此仅进行简要介绍。

1. 生成对抗网络（Generative Adversarial Networks, GANs）），通过对抗的方式学习复杂分布的采样过程。
2. ==基于似然likelihood-based）的模型==，这类方法旨在学习一个模型，使其能够为观察到的数据样本分配较高的似然值。这类模型包括自回归模型（autoregressive models）、归一化流（normalizing flows）以及变分自编码器（Variational Autoencoders, VAEs）。
3. 基于能量的建模（energy-based modeling），其目标是通过一个具有灵活性的能量函数来学习分布，然后对其进行归一化。与此密切相关的是基于分数的生成模型（score-based generative models），这类模型并非直接学习能量函数本身，而是通过神经网络来学习能量模型的分数（即梯度）。

在本文中，我们分析扩散模型（diffusion models）的定义及其工作原理，扩散模型既可以从基于似然的角度进行解释，也可以从基于分数的角度进行理解。

某些情况下，我们可以认为我们观察到的数据是由一个相关的、未被观察到的==隐变量（latent variable）==生成的，这个隐变量可以用随机变量 $z$ 表示。表达这一思想的最佳直觉来源于柏拉图的《洞穴寓言》。在寓言中，一群人终生被锁链束缚在洞穴中，他们只能看到投射在他们面前墙上的二维影子，而这些影子是由火焰前未被看到的三维物体生成的。对于这些人来说，他们所观察到的一切实际上是由他们永远无法看到的高维抽象概念决定的。

类似地，我们观测到的样本可以被解释为高维分布的三维投影或实例化，就像洞穴中的人们观察到的其实是三维物体的二维投影一样。尽管洞穴中的人们永远无法看到（甚至完全理解）隐藏的物体，但他们仍然对其进行猜测；以类似的方式，我们也可以近似估计我们观察到数据的潜在表示。

虽然柏拉图的寓言阐释了隐变量的思想，即可能无法直接观察到的表示决定了观察结果，但在生成建模中，我们通常寻求学习低维的潜在表示，而不是高维的。这是因为在没有强先验的情况下，试图学习比观察数据更高维的表示是徒劳的。另一方面，学习低维潜在变量也可以被视为一种压缩形式。虽然降低了潜在变量的维度，但它们仍然能够捕捉到数据中重要的特征或模式。例如：这些低维表示可能包含一些抽象的语义信息，颜色、形状、大小等，能够帮助我们理解数据的本质结构。



## 证据下界（Evidence Lower Bound）

>==似然函数==：在统计学中广泛应用，尤其是在模型参数估计中。例如： 最大似然估计（Maximum Likelihood Estimation, MLE）通过最大化似然函数来找到最优的模型参数。
>
>似然函数是一个关于模型参数的函数，表示在给定数据的条件下，模型参数使数据出现的概率大小。它的数学表达式通常如下：
>
>$$
>L(\theta | x) = P(x | \theta)
>$$
>
>其中：  $x$ 是观察到的数据（样本）， $\theta$ 是模型的参数。$P(x | \theta)$ 是在模型参数为 $\theta$ 时，数据 $x$ 出现的概率。
>
>注意，**似然函数是关于参数 $\theta$ 的函数**，而不是关于数据 $x$ 的函数。换句话说，似然函数的是评估不同的参数值（$\theta$）在给定数据情况下的优劣

从数学上来看，我们可以将观测到的数据和隐变量建模为联合分布 $p(x, z)$。回忆生成模型的一种方法，称为“基于似然”（likelihood-based），其目标是学习一个模型以最大化所有观察数据 $x$ 的似然 $p(x)$。

我们可以通过两种方式操作这个联合分布来恢复观测数据的似然 $p(x)$。我们可以显式地对潜在变量 $z$ 进行边缘化处理：

$$
p(x) = \int p(x, z) dz
$$

或者，我们可以使用概率的链式法则

$$
p(x) = \frac{p(x, z)}{p(z|x)}
\tag{2} \label{eq2}
$$

直接计算并最大化似然 $p(x)$ 是困难的，因为需要对公式 (1) 中对所有潜在变量 $z$ 进行积分，对于复杂模型（往往对应高维空间）来说是不可行的，或者需要知道公式 $\eqref{eq2}$ 中的隐变量编码器 $p(z|x)$。然而，通过使用这两个公式，我们可以推导出一个术语，称为 **证据下界（Evidence Lower Bound, ELBO）**，顾名思义，它是证据的一个下界。

在这种情况下，证据被量化为观察数据的对数似然。然后，最大化 ELBO 成为优化隐变量模型的代理目标；在最优情况下，当 ELBO 被强大地参数化并完美优化时，它与证据完全等价。ELBO 的公式为：

$$
\mathbb{E}_{q_\phi(z|x)} \left[ \log \frac{p(x, z)}{q_\phi(z|x)} \right]
\tag{3} \label{eq3}
$$
为了使与证据的关系更加明确，我们可以用数学表达如下：
$$
\log p(x) \geq \mathbb{E}_{q_\phi(z|x)} \left[ \log \frac{p(x, z)}{q_\phi(z|x)} \right] 
\tag{4} \label{eq4}
$$
这里，$q_\phi(z|x)$ 是一个灵活的近似变分分布，带有需要优化的参数 $\phi$，它可以被看作是一个参数化的模型，用于估计给定观测值 $x$ 的真实分布；换句话说，它试图逼近真实后验分布 $p(z|x)$。如我们将在探索变分自编码器时看到的，通过调整参数 $\phi$ 来最大化 ELBO（证据下界），我们可以获得一些组件，这些组件可以用于建模真实数据分布并从中采样，从而学习一个生成模型。目前，让我们深入探讨为什么 ELBO 是我们希望最大化的目标。



### 推导 ELBO（证据下界）

> ==Jensen不等式==，对于一个随机变量 $X$ 和一个凸函数 $f$，Jensen不等式的形式如下：
> $$
> f(\mathbb{E}[X]) \leq \mathbb{E}[f(X)]
> $$
> 其中，$ \mathbb{E}[X] $ 表示随机变量 $ X $ 的期望值。
>
> ==对数函数是凹函数==，所以有
> $$
> \log(\mathbb{E}[X]) \geq \mathbb{E}[\log(X)]
> $$
>
> ==KL散度（Kullback-Leibler Divergence）==，是衡量两个概率分布之间差异的一种度量方式。它是非对称的，通常用来比较一个分布（ 近似分布）与另一个分布（真实分布）的相似程度。KL散度定义如下：
> $$
> D_{KL}(q(z) \| p(z)) = \mathbb{E}_{q(z)} \left[ \log \frac{q(z)}{p(z)} \right]
> $$
>
> 或者写成积分形式：
>
> $$
> D_{KL}(q(z) \| p(z)) = \int q(z) \log \frac{q(z)}{p(z)} dz
> $$
>
> 其中： $ q(z) $ 是近似分布（通常是我们构造的分布，用来近似真实分布）。 $ p(z) $ 是真实分布（目标分布）。 $ \mathbb{E}_{q(z)} $ 表示对 $ q(z) $ 的期望。


我们从公式$\eqref{eq1}$开始推导 ELBO：

$$
\begin{align*}
\log p(x) &= \log \int p(x, z) \, dz \\
&= \log \int \frac{p(x, z) q_\phi(z \mid x)}{q_\phi(z \mid x)} \, dz \\
&= \log \mathbb{E}_{q_\phi(z \mid x)} \left[ \frac{p(x, z)}{q_\phi(z \mid x)} \right] \\
&\geq \mathbb{E}_{q_\phi(z \mid x)} \left[ \log \frac{p(x, z)}{q_\phi(z \mid x)} \right] \quad \text{(应用 Jensen 不等式)}
\end{align*}
\tag{5} \label{eq5}
$$

在这个推导过程中，通过应用 Jensen 不等式，我们直接得到了下界。然而，这并没有提供太多关于底层发生的实际情况的有用信息；至关重要的是，这种证明并没有直观地解释为什么 ELBO 实际上是证据的下界，因为 Jensen 不等式只是简单地给出了结果。此外，仅仅知道 ELBO 是证据的下界并不能真正告诉我们为什么我们希望将其最大化作为目标。

为了更好地理解证据与 ELBO 之间的关系，让我们使用公式$\eqref{eq2}$ 进行另一个推导。

$$
\begin{align}
\log p(x) &= \log p(x) \int q_\phi(z|x) dz \\
&= \int q_\phi(z|x) (\log p(x)) dz \\
&= \mathbb{E}_{q_\phi(z|x)} [\log p(x)] \\
&= \mathbb{E}_{q_\phi(z|x)} \left[ \log \frac{p(x, z)}{p(z|x)} \right] \\
&= \mathbb{E}_{q_\phi(z|x)} \left[ \log \frac{p(x, z) q_\phi(z|x)}{p(z|x) q_\phi(z|x)} \right] \\
&= \mathbb{E}_{q_\phi(z|x)} \left[ \log \frac{p(x, z)}{q_\phi(z|x)} \right] + \mathbb{E}_{q_\phi(z|x)} \left[ \log \frac{q_\phi(z|x)}{p(z|x)} \right] \\
&= \mathbb{E}_{q_\phi(z|x)} \left[ \log \frac{p(x, z)}{q_\phi(z|x)} \right] + D_{KL}(q_\phi(z|x) \| p(z|x)) \tag{6.1} \label{eq6.1}\\
&\geq \mathbb{E}_{q_\phi(z|x)} \left[ \log \frac{p(x, z)}{q_\phi(z|x)} \right] 
\end{align}
\tag{6} \label{eq6}
$$

通过以上推导，我们可以观察到证据$log(p(x))$等于ELBO加上近似后验$q_\phi(z|x)$ 和真实后验$p(z|x)$之间的KL散度。KL散度项通过Jensen不等式在公式$\eqref{eq5}$中被移除。

理解这一项是理解ELBO与证据之间关系的关键，同时也是为什么优化ELBO是一个合理目标的原因。我们现在已经明白为什么证据下界（ELBO）确实是一个下界：证据与ELBO之间的差异是一个严格非负的KL散度项，因此ELBO的值永远不会超过证据的值。

我们进一步探讨为何需要最大化ELBO。加入隐变量概念后，我们的目标是希望通过观测数据学习能够描述这些数据的潜在结构。换言之，我们旨在优化变分后验分布$q_{\phi}(z|x) $，使其尽可能匹配真实后验分布$p(z|x) $，这通过最小化两者的KL散度（理想情况下为零）来实现。

不幸的是，我们无法直接最小化这个KL散度，因为我们无法获得真实分布$p(z|x)$的数据。然而，请注意，在公式$\eqref{eq6.1}$的左侧，证据项$\log p(x)$是通过对联合分布$p(x, z)$中的所有潜变量$z$进行边际化计算的，证据项与参数$\phi$无关，对于参数$\phi$来说证据项$\log p(x)$可以被看做一个常数。

由于 ELBO 和 KL散度项的总和是一个常数，任何关于$\phi$的 ELBO 项的最大化都会同时引发KL散度的最小化。因此，ELBO可以被视为学习真实潜在后验分布的代理；我们越是优化 ELBO，我们的近似后验分布就越接近真实后验分布。



## **变分自编码器**

在变分自编码器（Variational Autoencoder, VAE）的默认公式中[^1]，我们直接最大化证据下界（ELBO）。这种方法被称为“变分”，因为我们在一组由参数 $\phi$ 表示的潜在后验分布中，优化出最佳的 $q_{\boldsymbol{\phi}}(\boldsymbol{z} \mid \boldsymbol{x})$。它被称为“自编码器”，是因为这种方法与传统的自编码器模型类似，其中输入数据在经历一个中间的瓶颈表示步骤后被训练来预测自身。为了明确这种联系，我们进一步剖析 ELBO 项：

$$
\begin{array}{rlrl}
\mathbb{E}_{q_\phi(\boldsymbol{z} \mid \boldsymbol{x})}\left[\log \frac{p(\boldsymbol{x}, \boldsymbol{z})}{q_{\boldsymbol{\phi}}(\boldsymbol{z} \mid \boldsymbol{x})}\right] & =\mathbb{E}_{q_\phi(\boldsymbol{z} \mid \boldsymbol{x})}\left[\log \frac{p_{\boldsymbol{\theta}}(\boldsymbol{x} \mid \boldsymbol{z}) p(\boldsymbol{z})}{q_{\boldsymbol{\phi}}(\boldsymbol{z} \mid \boldsymbol{x})}\right] \\
& =\mathbb{E}_{q_\phi(\boldsymbol{z} \mid \boldsymbol{x})}\left[\log p_{\boldsymbol{\theta}}(\boldsymbol{x} \mid \boldsymbol{z})\right]+\mathbb{E}_{q_\phi(\boldsymbol{z} \mid \boldsymbol{x})}\left[\log \frac{p(\boldsymbol{z})}{q_{\boldsymbol{\phi}}(\boldsymbol{z} \mid \boldsymbol{x})}\right]\\
& =\underbrace{\mathbb{E}_{q_\phi(\boldsymbol{z} \mid \boldsymbol{x})}\left[\log p_{\boldsymbol{\theta}}(\boldsymbol{x} \mid \boldsymbol{z})\right]}_{\text {重构项 }}-\underbrace{D_{\mathrm{KL}}\left(q_{\boldsymbol{\phi}}(\boldsymbol{z} \mid \boldsymbol{x}) \| p(\boldsymbol{z})\right)}_{\text {先验匹配项 }}
\end{array}
\tag{7} \label{eq7}
$$

在这种情况下，我们学习将输入转换为一个中间分布 $q_\phi(\boldsymbol{z} \mid \boldsymbol{x})$，也即隐变量的概率分布，这个模型可以被视为编码器；。同时，我们学习一个确定性函数 $p_{\boldsymbol{\theta}}(\boldsymbol{x} \mid \boldsymbol{z})$，将给定的隐向量 $\boldsymbol{z}$ 转换为观测值 $\boldsymbol{x}$，这个函数可以被看做解码器。

公式 $\eqref{eq7}$ 中的两项分别具有直观的解释：

1. 第一项衡量了解码器从变分分布中进行重构的可能性；这确保了学习到的分布能够有效地建模潜变量，使得原始数据可以从中再生成。
2. 第二项衡量了学习到的变分分布与潜变量的先验分布之间的相似程度。最小化该项可以促使编码器真正学习一个分布。最大化 ELBO 等价于最大化第一项并最小化第二项。

变分自编码器（VAE）的一个显著特征是，证据下界（ELBO）通过参数 $\phi$ 和 $\theta$ 的联合优化来实现。VAE的编码器通常被设计为对角协方差的多元高斯分布，而先验分布通常被选为标准多元高斯分布：

$$
\begin{aligned}
q_{\boldsymbol{\phi}}(\boldsymbol{z} \mid \boldsymbol{x}) & =\mathcal{N}\left(\boldsymbol{z} ; \boldsymbol{\mu}_{\boldsymbol{\phi}}(\boldsymbol{x}), \boldsymbol{\sigma}_\phi^2(\boldsymbol{x}) \mathbf{I}\right) \\
p(\boldsymbol{z}) & =\mathcal{N}(\boldsymbol{z} ; \mathbf{0}, \mathbf{I})
\end{aligned}
\tag{8} \label{eq8}
$$

在这种情况下，ELBO的KL散度项可以解析计算，而重构项可以通过蒙特卡洛估计进行近似。目标函数可以被重新写为：

$$
\underset{\boldsymbol{\phi}, \boldsymbol{\theta}}{\arg \max } \mathbb{E}_{q_{\boldsymbol{\phi}}(\boldsymbol{z} \mid \boldsymbol{x})}\left[\log p_{\boldsymbol{\theta}}(\boldsymbol{x} \mid \boldsymbol{z})\right]-D_{\mathrm{KL}}\left(q_{\boldsymbol{\phi}}(\boldsymbol{z} \mid \boldsymbol{x}) \| p(\boldsymbol{z})\right) \approx \underset{\boldsymbol{\phi}, \boldsymbol{\theta}}{\arg \max } \sum_{l=1}^L \log p_{\boldsymbol{\theta}}\left(\boldsymbol{x} \mid \boldsymbol{z}^{(l)}\right)-D_{\mathrm{KL}}\left(q_{\boldsymbol{\phi}}(\boldsymbol{z} \mid \boldsymbol{x}) \| p(\boldsymbol{z})\right)
\tag{9} \label{eq9}
$$

其中，隐变量 $\left\{\boldsymbol{z}^{(l)}\right\}_{l=1}^L$ 是从 $q_{\boldsymbol{\phi}}(\boldsymbol{z} \mid \boldsymbol{x})$ 中对数据集中的每个观测值 $\boldsymbol{x}$ 进行采样得到的。然而，在默认设置中会出现一个问题：我们的损失函数计算所依赖的每个 $\boldsymbol{z}^{(l)}$ 都是通过随机采样程序生成的，而随机采样过程通常是不可微的。幸运的是，当 $q_{\boldsymbol{\phi}}(\boldsymbol{z} \mid \boldsymbol{x})$ 被设计为某些分布（包括多元高斯分布）时，可以通过重参数化技巧解决这个问题。

重参数化技巧将随机变量重写为噪声变量的确定性函数；这使得非随机项可以通过梯度下降进行优化。例如，从一个均值为 $\mu$、方差为 $\sigma^2$ 的正态分布 $x \sim \mathcal{N}\left(x ; \mu, \sigma^2\right)$ 中采样的样本可以被重写为：

$$
x=\mu+\sigma \epsilon \quad \text { 其中 } \epsilon \sim \mathcal{N}(\epsilon ; 0, \mathrm{I})
\tag{10} \label{eq10}
$$

换句话说，任意高斯分布可以被解释为一个标准高斯分布（其中 $\epsilon$ 是一个样本），通过加法将其均值从零移动到目标均值 $\mu$，并通过目标方差 $\sigma^2$ 进行伸缩。因此，通过重参数化技巧，可以通过从标准高斯分布采样，将结果按目标标准差进行缩放，并通过目标均值进行平移，来实现从任意高斯分布采样。

在VAE中，每个 $\boldsymbol{z}$ 因此被表示为输入 $\boldsymbol{x}$ 和辅助噪声变量 $\boldsymbol{\epsilon}$ 的确定性函数：

$$
z=\mu_\phi(\boldsymbol{x})+\sigma_\phi(\boldsymbol{x}) \odot \boldsymbol{\epsilon} \quad \text { 其中 } \boldsymbol{\epsilon} \sim \mathcal{N}(\boldsymbol{\epsilon} ; \mathbf{0}, \mathbf{I})
\tag{11} \label{eq11}
$$

其中 $\odot$ 表示逐元素乘法。在这种重参数化版本的 $\boldsymbol{z}$ 下，可以根据需要计算相对于 $\phi$ 的梯度，从而优化 $\mu_\phi$ 和 $\sigma_\phi$。因此，VAE通过重参数化技巧和蒙特卡洛估计来联合优化 $\phi$ 和 $\boldsymbol{\theta}$ 的ELBO。

在训练完成后，可以通过直接从潜变量空间 $p(\boldsymbol{z})$ 中采样并将其输入解码器来生成新数据。当 $\boldsymbol{z}$ 的维度小于输入 $\boldsymbol{x}$ 的维度时，变分自编码器会特别有趣，因为此时我们可能学习到紧凑且有用的表示。此外，当潜变量空间具有语义意义时，可以在将潜向量传递给解码器之前对其进行编辑，以更精确地控制生成的数据。

[^1]:Kingma, Diederik P., and Max Welling. "Auto-encoding variational bayes." 20 Dec. 2013,



## **层级变分自编码器**

层级变分自编码器（Hierarchical Variational Autoencoder, HVAE）[^2] [^3]是对变分自编码器（VAE）的一种推广，它将隐变量扩展到多层级结构。在这种框架下，隐变量本身被视为由更高层次、更抽象的潜变量生成的。直观地讲，就像我们将三维观测物体视为由更高层次的抽象隐变量生成一样，柏拉图“洞穴寓言”中的人将三维物体视为生成其二维观测的隐变量。因此，从柏拉图洞穴居民的视角来看，他们的观测可以被视为由深度为两层（或更多层）的隐变量层级模型生成的。

在具有 $T$ 层的通用 HVAE 中，每个潜变量都可以依赖于所有先前的潜变量。然而，在本文中，我们关注一种特殊情况，称之为马尔可夫层级变分自编码器（Markovian HVAE, MHVAE）。在马尔科夫层级变分自编码器（MHVAE）中，生成过程是一个马尔科夫链。具体来说，层次结构中的每一次向下转移都遵循马尔科夫性质，即每个潜变量$z_t$ 的解码仅依赖于其上一层的潜变量$z_{t+1}$。从直观和视觉角度来看，这种模型可以简单地理解为将多个变分自编码器（VAE）逐层堆叠在一起，如图2所示。因此，这种模型也可以称为递归VAE（Recursive VAE）。在数学上，我们可以用以下公式表示马尔科夫HVAE的联合分布和后验分布：：

$$
\begin{aligned}
    p\left(\boldsymbol{x}, \boldsymbol{z}_{1: T}\right) & =p\left(\boldsymbol{z}_T\right) p_{\boldsymbol{\theta}}\left(\boldsymbol{x} \mid \boldsymbol{z}_1\right) \prod_{t=2}^T p_{\boldsymbol{\theta}}\left(\boldsymbol{z}_{t-1} \mid \boldsymbol{z}_t\right)
\end{aligned}
\tag{12} \label{eq12}
$$
$$
\begin{aligned}
    q_{\boldsymbol{\phi}}\left(\boldsymbol{z}_{1: T} \mid \boldsymbol{x}\right) & =q_{\boldsymbol{\phi}}\left(\boldsymbol{z}_1 \mid \boldsymbol{x}\right) \prod_{t=2}^T q_{\boldsymbol{\phi}}\left(\boldsymbol{z}_t \mid \boldsymbol{z}_{t-1}\right)
\end{aligned}
\tag{13} \label{eq13}
$$

然后，我们可以将证据下界（ELBO）轻松扩展为：
$$
\begin{aligned}
\log p(\boldsymbol{x}) & =\log \int p\left(\boldsymbol{x}, \boldsymbol{z}_{1: T}\right) d \boldsymbol{z}_{1: T} \\
& =\log \int \frac{p\left(\boldsymbol{x}, \boldsymbol{z}_{1: T}\right) q_{\boldsymbol{\phi}}\left(\boldsymbol{z}_{1: T} \mid \boldsymbol{x}\right)}{q_{\boldsymbol{\phi}}\left(\boldsymbol{z}_{1: T} \mid \boldsymbol{x}\right)} d \boldsymbol{z}_{1: T} \\
& =\log \mathbb{E}_{q_{\boldsymbol{\phi}}\left(\boldsymbol{z}_{1: T} \mid \boldsymbol{x}\right)}\left[\frac{p\left(\boldsymbol{x}, \boldsymbol{z}_{1: T}\right)}{q_{\boldsymbol{\phi}}\left(\boldsymbol{z}_{1: T} \mid \boldsymbol{x}\right)}\right] \\
& \geq \mathbb{E}_{q_{\boldsymbol{\phi}}\left(\boldsymbol{z}_{1: T} \mid \boldsymbol{x}\right)}\left[\log \frac{p\left(\boldsymbol{x}, \boldsymbol{z}_{1: T}\right)}{q_{\boldsymbol{\phi}}\left(\boldsymbol{z}_{1: T} \mid \boldsymbol{x}\right)}\right]
\end{aligned}
\tag{14} \label{eq14}
$$
接下来，我们可以将联合分布公式 $\eqref{eq12}$和后验分布公式 $\eqref{eq13}$代入公式 $\eqref{eq14}$，从而得到一种替代形式：
$$
\mathbb{E}_{q_{\boldsymbol{\phi}}\left(\boldsymbol{z}_{1: T} \mid \boldsymbol{x}\right)}\left[\log \frac{p\left(\boldsymbol{x}, \boldsymbol{z}_{1: T}\right)}{q_{\boldsymbol{\phi}}\left(\boldsymbol{z}_{1: T} \mid \boldsymbol{x}\right)}\right]=\mathbb{E}_{q_{\boldsymbol{\phi}}\left(\boldsymbol{z}_{1: T} \mid \boldsymbol{x}\right)}\left[\log \frac{p\left(\boldsymbol{z}_T\right) p_{\boldsymbol{\theta}}\left(\boldsymbol{x} \mid \boldsymbol{z}_1\right) \prod_{t=2}^T p_{\boldsymbol{\theta}}\left(\boldsymbol{z}_{t-1} \mid \boldsymbol{z}_t\right)}{q_{\boldsymbol{\phi}}\left(\boldsymbol{z}_1 \mid \boldsymbol{x}\right) \prod_{t=2}^T q_{\boldsymbol{\phi}}\left(\boldsymbol{z}_t \mid \boldsymbol{z}_{t-1}\right)}\right]
\tag{15} \label{eq15}
$$
正如我们将在下文所展示的，当我们研究变分扩散模型时，该目标可以进一步分解为具有可解释性的组成部分。

[^2]: Kingma, Durk P., et al. "Improved variational inference with inverse autoregressive flow." *Advances in neural information processing systems* 29 (2016).
[^3]: Sønderby, Casper Kaae, et al. "Ladder variational autoencoders." *Advances in neural information processing systems* 29 (2016).



文章翻译并略微修改自文章：Understanding Diffusion Models: A Unified Perspective

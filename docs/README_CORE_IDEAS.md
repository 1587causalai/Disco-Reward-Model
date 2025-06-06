# Disco Reward Model 论文核心思想

reward score 是一个随机变量, 并且随机性来自于不同的个体.  举个例子, 苹果好吃还是香蕉好吃, 不同的人有不同的答案, 对于这样没有标准答案的问题, reponse "苹果" 的奖励和 reponse "香蕉" 的奖励是不同的, 因此他应该具备非常大的方差. 即使评分差异非常大, 加大方差, 我们 disco-Reward 可以让其没有偏好. 

> 方案完全变了, 基于 abduction/action 的 cauchy 回归奖励模型. 


## 1. 研究背景与动机

### 1.1 当前 RLHF 奖励模型面临的挑战

在大型语言模型（LLMs）的对齐（Alignment）研究中，传统的从人类反馈中学习强化（RLHF）方法虽然取得了显著成果，但仍面临几个关键挑战：

- **奖励误定义（Reward Misspecification）**: 传统奖励模型输出一个标量值，但这种简化往往无法捕捉人类偏好的全部复杂性，导致模型优化方向与真实目标产生偏差。
- **奖励操纵（Reward Hacking）**: 当模型过度优化一个有缺陷的奖励信号时，可能会产生表面上高分但实际不符合人类期望的行为。
- **循环偏好问题**: 阿罗不可能定理（Arrow's Impossibility Theorem）指出，在群体决策中可能出现非传递性偏好循环（如 a > b > c > a），这是确定性标量奖励模型难以表达的。

### 1.2 阿罗不可能定理与人类偏好的复杂性

阿罗不可能定理揭示了一个基本限制：不存在能够同时满足一系列合理性条件的理想投票或决策系统。在人类集体决策中，经常会出现循环偏好现象，即群体整体可能同时呈现：
- 选项 a 优于选项 b
- 选项 b 优于选项 c
- 选项 c 却又优于选项 a

这种现象在个体层面看是悖论性的（单个理性个体的偏好应当满足传递性），但在群体层面确是常见的。传统的确定性奖励模型隐含地假设存在一个能对所有行为进行全序排列的"真实"奖励函数，这与人类决策的实际复杂性不符。

## 2. Disco Reward Model 核心思想

### 2.1 奖励作为随机变量

Disco Reward Model 的核心创新在于将奖励建模为**随机变量**，而非传统的确定性标量值。具体来说：

- 对于给定的提示（prompt）x 和回答（response）y，模型输出的不是单一奖励值，而是一个奖励**分布**，通常以分布参数表示（如均值 μ 和方差 σ²）。
- 这种随机性设计能够自然地捕捉：
  1. **评价者间的异质性**：不同人对同一回答的评价可能不同
  2. **情境依赖性**：同一回答在不同上下文中的价值可能不同
  3. **内在不确定性**：有些评价本身就具有模糊性
  4. **群体非传递性偏好**：可以在模型层面反映群体决策中可能出现的循环偏好

### 2.2 数学模型与演进的思考

最初，在 Disco Reward Model 的探索阶段，我们采用了一些简化的启发式假设，以便于模型的初步构建和验证：

1.  **最初的启发式：正态分布假设**：我们曾假设奖励 $R(x, y)$ 服从正态分布，即
    $R(x, y; \psi) \sim \mathcal{N}(\mu_{\psi}(x, y), \sigma^2_{\psi}(x, y))$
    其中 $\mu_{\psi}$ 和 $\sigma^2_{\psi}$ 是由神经网络 $f_{\psi}$ 预测的均值和方差参数。这个选择主要基于正态分布在统计建模中的普遍性及其数学上的便利性。

然而，深入思考人类偏好的复杂性，特别是观察到偏好数据中可能存在的"重尾现象"——即少数极端强烈的正面或负面评价——我们开始重新审视正态分布的适用性。正态分布的尾部衰减较快，可能难以充分捕捉这类罕见但重要的极端偏好信号。这些极端信号可能恰恰是理解模型潜在风险（如"对齐越狱"）的关键。

2.  **当前的思考：转向柯西分布 (Cauchy Distribution)**：基于上述考虑，我们现在提出一个新的核心假设：奖励 $R(x, y)$ 服从**柯西分布**。
    $R(x, y; \psi) \sim \text{Cauchy}(x_{0, \psi}(x, y), \gamma_{\psi}(x, y))$

    其中：
    *   $x_{0, \psi}(x, y)$ 是由神经网络 $f_{\psi}$ 预测的**位置参数 (location parameter)**，它描述了分布的中心趋势（类似于中位数）。
    *   $\gamma_{\psi}(x, y)$ (其中 $\gamma > 0$) 是由神经网络 $f_{\psi}$ 预测的**尺度参数 (scale parameter)**，它描述了分布的离散程度或宽度。

    柯西分布以其"重尾"特性著称，这意味着它比正态分布更容易产生远离中心位置的极端值。我们认为这一特性使其更适合捕捉人类偏好中固有的、有时甚至是剧烈的不确定性和多样性。值得注意的是，柯西分布的数学期望（均值）和方差是未定义的，这一特性深刻地影响了我们后续对偏好概率和优化目标的设计。

3.  **条件独立性假设**：与原模型类似，我们暂时保留给定提示 $x$，不同完成 $y_1$ 和 $y_2$ 的奖励随机变量是条件独立的假设。未来工作中可以探索放宽此假设。

基于这些新的假设，特别是奖励服从柯西分布，我们可以重新计算偏好概率 $P(y_w \succ y_l | x)$。
已知若 $R_w \sim \text{Cauchy}(x_{0w}, \gamma_w)$ 且 $R_l \sim \text{Cauchy}(x_{0l}, \gamma_l)$ 且两者独立，则它们的差 $Z = R_w - R_l$ 也服从柯西分布：
$Z \sim \text{Cauchy}(x_{0w} - x_{0l}, \gamma_w + \gamma_l)$
令 $x_0 = x_{0w} - x_{0l}$ 和 $\gamma = \gamma_w + \gamma_l$。
偏好概率 $P(y_w \succ y_l | x) = P(Z > 0)$ 可以通过柯西分布的累积分布函数 (CDF) 计算。柯西分布 $C(x_0, \gamma)$ 的 CDF 为 $F(z; x_0, \gamma) = \frac{1}{\pi} \arctan\left(\frac{z - x_0}{\gamma}\right) + \frac{1}{2}$。
因此：
$P(y_w \succ y_l | x) = 1 - F(0; x_0, \gamma) = 1 - \left( \frac{1}{\pi} \arctan\left(\frac{-(x_{0w} - x_{0l})}{\gamma_w + \gamma_l}\right) + \frac{1}{2} \right)$
$P(y_w \succ y_l | x) = \frac{1}{2} - \frac{1}{\pi} \arctan\left(\frac{x_{0l} - x_{0w}}{\gamma_w + \gamma_l}\right) = \frac{1}{2} + \frac{1}{\pi} \arctan\left(\frac{x_{0w} - x_{0l}}{\gamma_w + \gamma_l}\right)$

重要的是，我们明确认识到这些假设（包括柯西分布和条件独立性）仍然是实际问题的一种简化。例如，真实的奖励分布可能更为复杂，或者不同回复间的奖励可能存在更复杂的依赖关系。然而，我们相信转向柯西分布是捕捉人类偏好特性的一个更精确和有力的方向。这一理论框架的转变将对后续的训练目标、Disco-DPO 算法设计以及模型的理论优势分析产生深远影响，我们将在后续章节中详细探讨。

### 2.3 训练目标

奖励模型通过最大化观测偏好数据的似然来训练。其损失函数定义为：

$\mathcal{L}_{RM}(\psi; D) = -\mathbb{E}_{(x, y_w, y_l) \sim D}[\log P_{\psi}(y_w \succ y_l | x)]$

其中，$P_{\psi}(y_w \succ y_l | x)$ 是在给定模型参数 $\psi$ 和输入 $x$ 的条件下，回答 $y_w$ 优于 $y_l$ 的概率。根据我们在 2.2 节中推导的，当奖励 $R(x,y)$ 服从柯西分布时，该概率由以下公式给出：

$P_{\psi}(y_w \succ y_l | x) = \frac{1}{2} + \frac{1}{\pi} \arctan\left(\frac{x_{0,\psi}(x, y_w) - x_{0,\psi}(x, y_l)}{\gamma_{\psi}(x, y_w) + \gamma_{\psi}(x, y_l)}\right)$

这一训练过程旨在优化神经网络 $f_{\psi}$ 的参数，使其预测的位置参数 $x_0$ 和尺度参数 $\gamma$ 能够最佳地拟合观测到的成对偏好数据 $D$。

## 3. Disco-DPO：将随机奖励思想融入直接偏好优化

### 3.1 DPO 背景

直接偏好优化（Direct Preference Optimization, DPO）通过将偏好数据直接用于优化策略网络 $\pi_{\theta}$，避免了显式训练奖励模型再进行强化学习的必要。标准 DPO 基于 Bradley-Terry 模型，隐含地假设奖励是确定性的，并与策略对数比率相关。

### 3.2 Disco-DPO 创新：从正态分布到柯西分布的演进

Disco-DPO 的核心思想是将随机奖励的概念直接融入直接偏好优化（DPO）的框架中。其具体实现随着我们对奖励分布模型的理解而演进。

#### 3.2.1 最初的探索：基于正态分布的 Disco-DPO

在我们最初假设奖励 $R(x,y)$ 服从正态分布 $\mathcal{N}(\mu, \sigma^2)$ 时，Disco-DPO 的设计如下：

1.  **偏好概率模型 (正态)**：基于两个正态分布奖励之差的概率，使用标准正态分布的累积分布函数 $\Phi$：
    $P_{\theta}(y_w \succ y_l | x) = \Phi\left(\frac{\mu_{\theta}(x, y_w) - \mu_{\theta}(x, y_l)}{\sqrt{\sigma^2_{\theta}(x, y_w) + \sigma^2_{\theta}(x, y_l)}}\right)$

2.  **均值与策略链接 (正态)**：奖励的均值 $\mu_{\theta}(x, y)$ 与策略 $\pi_{\theta}$ 和参考策略 $\pi_{ref}$ 的对数概率比率相关联：
    $\mu_{\theta}(x, y) = \beta \log\frac{\pi_{\theta}(y|x)}{\pi_{ref}(y|x)}$

3.  **方差参数化 (正态)**：奖励的方差 $\sigma^2_{\theta}(x, y)$ 由策略网络 $\pi_{\theta}$ 直接预测。

4.  **损失函数 (正态)**：对应的 Disco-DPO 损失函数为：
    $\mathcal{L}_{Disco-DPO}^{Normal}(\pi_{\theta}; \pi_{ref}) = -\mathbb{E}_{(x, y_w, y_l) \sim D}\left[\log \Phi\left(\frac{\mu_{\theta}(x, y_w) - \mu_{\theta}(x, y_l)}{\sqrt{\sigma^2_{\theta}(x, y_w) + \sigma^2_{\theta}(x, y_l)}}\right)\right]$

然而，正如 2.2 节所述，正态分布在捕捉奖励的重尾特性和处理潜在的数值稳定性方面存在局限性。这促使我们将核心奖励模型转向柯西分布，并相应地重新设计 Disco-DPO。

#### 3.2.2 当前的方案：基于柯西分布的 Disco-DPO

当我们将奖励模型更新为柯西分布 $R(x,y) \sim \text{Cauchy}(x_0, \gamma)$ 后，Disco-DPO 的各个组成部分也随之调整：

1.  **替换偏好概率模型 (柯西)**：我们采用基于两个独立柯西分布之差的偏好概率（详见 2.2 节推导）：
    $P_{\theta}(y_w \succ y_l | x) = \frac{1}{2} + \frac{1}{\pi} \arctan\left(\frac{x_{0,\theta}(x, y_w) - x_{0,\theta}(x, y_l)}{\gamma_{\theta}(x, y_w) + \gamma_{\theta}(x, y_l)}\right)$
    其中，$x_{0,\theta}$ 和 $\gamma_{\theta}$ 是由当前策略网络 $\pi_{\theta}$ 参数化的位置和尺度参数。这种基于 $\arctan$ 的形式有望在处理极端参数时提供更好的数值稳定性。

2.  **位置参数 ($x_0$) 与策略链接 (柯西)**：柯西分布的**位置参数 $x_{0,\theta}(x, y)$** （而非均值）与策略的对数概率比率关联：
    $x_{0,\theta}(x, y) = \beta \log\frac{\pi_{\theta}(y|x)}{\pi_{ref}(y|x)} + \text{const}$
    其中 $\beta$ 是一个超参数。常数项 `const` 或让 $x_0$ 的一部分直接由网络预测，可以增加模型的表达能力。

3.  **尺度参数 ($\gamma$) 参数化 (柯西)**：柯西分布的**尺度参数 $\gamma_{\theta}(x, y)$** 反映了奖励信号的不确定性或分散程度。该参数由策略网络 $\pi_{\theta}$ 直接预测，并强制 $\gamma_{\theta}(x, y) > 0$。

4.  **Disco-DPO 损失函数 (柯西)**：新的 Disco-DPO 损失函数旨在最大化基于柯西分布的偏好概率的对数似然：
    $\mathcal{L}_{Disco-DPO}^{Cauchy}(\pi_{\theta}; \pi_{ref}) = -\mathbb{E}_{(x, y_w, y_l) \sim D}\left[\log \left( \frac{1}{2} + \frac{1}{\pi} \arctan\left(\frac{x_{0,\theta}(x, y_w) - x_{0,\theta}(x, y_l)}{\gamma_{\theta}(x, y_w) + \gamma_{\theta}(x, y_l)}\right) \right)\right]$
    其中 $x_{0,\theta}(x, y_w)$ 和 $x_{0,\theta}(x, y_l)$ 根据上述策略链接计算，而 $\gamma_{\theta}(x, y_w)$ 和 $\gamma_{\theta}(x, y_l)$ 由策略网络直接预测。

通过最小化此损失函数，策略网络 $\pi_{\theta}$ 被优化以直接拟合观测到的人类偏好数据。基于柯西分布的 Disco-DPO 旨在更好地处理偏好数据中的重尾现象，并增强训练的鲁棒性。

## 4. 理论意义与潜在优势

### 4.1 更贴近人类决策的复杂性

Disco Reward Model 的随机性设计更接近人类决策的本质：

- **承认不确定性**：明确认识到奖励评估中的固有不确定性和变异性
- **兼容循环偏好**：能够在理论上容纳阿罗不可能定理揭示的循环偏好现象
- **反映群体异质性**：可以自然表达不同人群或情境下的偏好差异

### 4.2 潜在的实际优势

- **抵抗奖励操纵**：通过考虑奖励的不确定性（方差），模型可能更谨慎地优化高方差区域，减少过拟合有缺陷奖励信号的风险
- **校准置信度**：模型可利用奖励方差信息来调整其输出的置信度
- **处理模糊查询**：对于模糊或有多种合理答案的查询，随机奖励模型可能表现更好

### 4.3 局限性、挑战与未来方向 (基于柯西分布的新视角)

尽管我们相信转向柯西分布来建模奖励 $R(x,y)$ 是一个重要的进步，能够更好地捕捉人类偏好的重尾特性并提升数值稳定性，但这一新的理论框架也带来了自身的局限性、挑战以及更广阔的未来研究方向。

**当前的局限性与挑战：**

1.  **单峰柯西分布的简化**：虽然柯西分布的重尾特性优于正态分布，但真实的奖励分布可能更为复杂，例如可能存在多峰（multimodal）现象（即一个回应可能引发几种截然不同但内部一致的评价群体）。当前的单峰柯西分布假设可能无法完全捕捉此类复杂结构。此外，虽然条件独立性假设在数学上带来了便利，但不同候选回应之间的奖励评价在现实中可能存在关联性，尤其当它们针对同一复杂或模糊的提示时。

2.  **位置参数 $x_0$ 与策略链接的启发性**：将柯西分布的位置参数 $x_0$ 与策略的对数概率比率进行链接，延续了DPO的核心思想。然而，这仍然是一种启发式的设计。$x_0$ 作为分布的中心（中位数），其与最优策略的关系可能比原先正态分布的均值 $\mu$ 更为复杂，尤其是在柯西分布均值未定义的情况下。这种链接方式的最优性和理论保障有待进一步探究。

3.  **尺度参数 $\gamma$ 的诠释与利用**：尺度参数 $\gamma$ 提供了关于偏好分散程度或不确定性的重要信息。当前模型直接从网络预测 $\gamma$，但如何更精确地诠释 $\gamma$ 的变化（例如，它反映的是数据噪声、内在模糊性，还是群体间的真实分歧？），以及如何在后续的策略优化中更有效地利用 $\gamma$ 值（除了在偏好概率计算中），是值得深入研究的问题。

4.  **参数估计与优化稳定性**：虽然基于 $\arctan$ 的偏好概率有助于提升数值稳定性，但柯西分布的参数估计（特别是当数据稀疏或存在极端异常点时）本身可能比正态分布更具挑战性。确保训练过程的鲁棒性和高效性，以及对超参数（如 $\beta$）的敏感性分析，仍然是重要的实践考量。

**未来可能的研究方向：**

1.  **更丰富的奖励分布模型**：
    *   **混合柯西分布 (Mixture of Cauchy Distributions)**：为了处理潜在的多峰偏好，可以探索使用柯西分布的混合模型来建模奖励，这能提供更大的灵活性。
    *   **其他重尾分布**：系统比较柯西分布与其他重尾分布（如学生t分布的低自由度变体）在建模奖励时的表现。
    *   **条件依赖建模**：放松奖励间的条件独立性假设，例如通过引入协方差结构或使用更复杂的图模型来捕捉 $R(y_w)$ 和 $R(y_l)$ 之间的依赖关系。

2.  **深化对 $x_0$ 和 $\gamma$ 的理解与应用**：
    *   **动态调整 $\beta$ 和 $\gamma$ 的学习**：研究更自适应的方法来学习或调整 $\beta$（在 $x_0$ 的策略链接中）和 $\gamma$，使其能更好地反映不同情境下的偏好结构和不确定性水平。
    *   **$\gamma$ 指导的探索 (Gamma-guided Exploration)**：在强化学习或后续微调阶段，探索如何利用预测的尺度参数 $\gamma$ 来指导模型的探索策略，例如，对 $\gamma$ 值较大的区域进行更谨慎的优化或更积极的探索。

3.  **柯西分布下DPO理论的深化**：对基于柯西分布的 Disco-DPO（$\mathcal{L}_{Disco-DPO}^{Cauchy}$）进行更深入的理论分析，包括其收敛性质、对策略梯度的影响，以及它如何具体地帮助缓解对齐过程中的越狱或奖励操纵问题。

4.  **大规模实证研究与应用场景拓展**：
    *   在更多样化、更具挑战性的人类偏好数据集上进行广泛的实证评估，特别关注那些已知包含极端或高度分歧反馈的场景。
    *   探索将基于柯西分布的奖励模型应用于更广泛的对齐任务，例如多轮对话、长文本生成或多模态输出的对齐。

5.  **与鲁棒统计的联系**：进一步挖掘柯西分布在奖励建模中与鲁棒统计思想的联系。柯西分布对异常值的低敏感度可能使其成为一种天然的鲁棒奖励建模方法，这方面的理论和实践意义值得探索。

通过解决这些挑战并探索这些未来方向，我们期望能进一步提升 Disco Reward Model 的性能和适用性，为构建更安全、更可靠、更符合人类复杂偏好的 AI 系统贡献力量。

## 5. 结论

Disco Reward Model 通过将奖励建模为随机变量，打开了一个探索大模型对齐的新视角。通过拥抱人类偏好和决策固有的随机性和复杂性，这一方法有望开发出更符合人类价值观、能够处理现实世界偏好复杂性的 AI 系统。虽然目前的实现基于一些启发式假设，但这一理论框架为未来研究提供了丰富的可能性。 
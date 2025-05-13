# Disco-Reward-Model

本项目旨在研究、实现和评估一种新颖的奖励模型训练方法——**Disco Reward Model (DiscoRM)**，通过将奖励建模为概率分布（包含均值和方差），以期更准确地捕捉人类偏好的复杂性与不确定性，从而改进大型语言模型（LLM）的对齐过程（如 RLHF）。


> 方案完全变了, 基于 abduction/action 的 cauchy 回归奖励模型. 



## 项目核心概念（摘要）

DiscoRM 的核心思想是将奖励视为**随机变量**，而非传统 RLHF 中的确定性标量。这主要基于以下考虑：

1.  **人类偏好的异质性与不确定性**: 不同人对同一反馈可能有不同评价，单一标量难以表达这种分歧。
2.  **奖励信号的内在模糊性**: 某些任务的评价标准本身就存在模糊地带。
3.  **理论启发 (阿罗不可能定理)**: 群体决策中可能出现非传递性偏好，随机性建模提供了更灵活的表达框架。

通过预测奖励的**均值 (μ)** 和**方差 (σ²)**，DiscoRM 理论上能够：

*   更准确地**表达模型对预测的不确定性**。
*   在优化过程中**对高不确定性区域更鲁棒**，可能缓解奖励误定义和操纵问题。
*   提供**更丰富的信号**用于改进偏好优化算法 (如 Disco-DPO)。

欲了解更深入的理论阐述和优势分析，请参考我们的 [**理论文档网站**](./docs/index.html)。

## 项目结构与入口

本项目主要包含以下几个部分，各有侧重：

1.  **[理论文档 (Theory Docs)](./docs/index.html)** (位于 `./docs/`)
    *   **目标受众**: 对 DiscoRM **核心思想、动机、数学模型、理论优势** 感兴趣的研究人员和广大读者。
    *   **内容**: 详细阐述 DiscoRM 的理论基础。
    *   **形式**: 基于 Docsify 的独立文档网站。

2.  **[代码实现 (Code Implementation)](./DiscoRM-LLaMA-Factory/)** (位于 `./DiscoRM-LLaMA-Factory/`)
    *   **目标受众**: 需要使用、修改或理解 DiscoRM **代码实现**的工程师和开发者。
    *   **内容**: 基于 LLaMA Factory 修改的代码库，包含 DiscoRM 模型、损失函数和训练流程的实现。
    *   **入口**: `DiscoRM-LLaMA-Factory/README.md` (包含原始 LLaMA Factory 说明和 DiscoRM 特定指引)。

3.  **[代码实现文档 (Code Docs)](./DiscoRM-LLaMA-Factory/docs/index.html)** (位于 `./DiscoRM-LLaMA-Factory/docs/`)
    *   **目标受众**: 使用代码库的工程师和开发者。
    *   **内容**: 聚焦于**代码层面的细节**：模型架构 (`NormalHead`)、损失函数实现、Trainer 适配、配置参数、运行指南等。
    *   **形式**: 基于 Docsify 的独立文档网站。

4.  **[论文 (LaTeX Source)](./Disco-Reward-Model-Latex/)** (位于 `./Disco-Reward-Model-Latex/`)
    *   **目标受众**: 学术界读者。
    *   **内容**: 项目的正式学术论文 LaTeX 源文件，包含最严谨的理论推导和实验结果。

5.  **辅助材料** (位于 `./docs/`)
    *   `./docs/Brainstorm/`: 项目构思和探索阶段的想法记录。
    *   `./docs/background_knowledge/`: 相关研究论文和背景资料。

## 主要特点总结

*   创新的 DiscoRM 方法：将奖励建模为正态分布 (均值+方差)。
*   潜在优势：更好地捕捉偏好复杂性，增强优化鲁棒性。
*   基于 LLaMA Factory 实现，易于扩展和使用。
*   提供独立的理论文档站和代码实现文档站。
*   包含配套论文。

## 如何开始？

*   **了解理论**: 请访问 [理论文档网站](./docs/index.html)。
*   **使用代码**: 请查阅 [代码库 (`DiscoRM-LLaMA-Factory/`)](./DiscoRM-LLaMA-Factory/) 及其 [代码实现文档](./DiscoRM-LLaMA-Factory/docs/index.html)。
*   **阅读论文**: 请查看 [论文库 (`Disco-Reward-Model-Latex/`)](./Disco-Reward-Model-Latex/)。 
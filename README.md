# Disco-Reward-Model

基于 Disco Loss 的奖励模型训练框架，用于增强大型语言模型的 RLHF 流程。

## 项目结构

- **DiscoRM-LLaMA-Factory/**: 核心代码库，基于 LLaMA-Factory 框架的修改版本
  - `docs/`: 详细的设计文档和实现细节
    - `DiscoRM_Loss_and_Trainer_Design.md`: Disco Loss 和训练器设计
    - `reward_model_training_overview.md`: 奖励模型训练概述
    - `discorm_base_model_design.md`: DiscoRM 基础模型设计

- **Disco-Reward-Model-Latex/**: 项目论文和学术报告的 LaTeX 文件

- **Brainstorm/**: 项目构思和探索阶段的想法记录

- **background_knowledge/**: 相关研究论文和背景资料

- **README_CORE_IDEAS.md**: DiscoRM 方法的核心理念和创新点

- **THEORETICAL_EVALUATION.md**: DiscoRM 方法的理论分析和评估

## 主要特点

- 创新的 Disco Loss 函数设计，提升奖励模型训练效果
- 灵活的模型架构，支持将现有 LLM 转换为奖励模型
- 与 LLaMA-Factory 框架的无缝集成
- 完善的理论支撑和实验评估

## 使用方法

详细使用说明请参考 `DiscoRM-LLaMA-Factory/docs/` 目录下的文档。 
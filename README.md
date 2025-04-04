# Disco Reward Model

overleaf: https://www.overleaf.com/project/65de312e3d37beae9b7afed6



- 最简单的代码实现 disco-DPO with LLamaFactory



## 核心贡献

reward score 是一个随机变量, 并且随机性来自于不同的个体.  举个例子, 苹果好吃还是香蕉好吃, 不同的人有不同的答案, 对于这样没有标准答案的问题, reponse "苹果" 的奖励和 reponse "香蕉" 的奖励是不同的, 因此他应该具备非常大的方差. 即使评分差异非常大, 加大方差, 我们 disco-Reward 可以让其没有偏好.  
---
title : 'LLM Agent(2) - Memory'
permalink: /posts/2024/03/llm-agent-2/
date : 2024-03-05
tags:
    - Agent
    - LLM
---
## 组件二：记忆
### 记忆的种类
这里的记忆是指负责**获取，存储，保存和检索信息的进程**。如果把代理中的记忆和人类记忆相比较的话：

- 负责嵌入表示的记忆就相当于人类的感官记忆，负责原始输入的处理，包括文字、图片或其他模态的数据。
- 上下文学习的记忆就相当于人类的短期记忆，被上下文窗口的长度限制。
- 外部向量存储就相当于人类的长期记忆，代理可以随时查询。

最大内积搜索（MIPS）是一种用于查询外部向量存储的方法，当前常见的方法是使用 ANN 算法，查找 k 个最接近的点，在一点性能损失的前提下，获得很大的速度提升。比较常见的 ANN 算法有下列几个：

- LSH，Locality-Sensitive Hashing
- [ANNOY](https://github.com/spotify/annoy) (Approximate Nearest Neighbors Oh Yeah)
- [HNSW](https://arxiv.org/abs/1603.09320) (Hierarchical Navigable Small World)
- [FAISS](https://github.com/facebookresearch/faiss) (Facebook AI Similarity Search)
- [ScaNN](https://github.com/google-research/google-research/tree/master/scann) (Scalable Nearest Neighbors)

### Reference
Weng, Lilian. (Jun 2023). LLM-powered Autonomous Agents”. Lil’Log. https://lilianweng.github.io/posts/2023-06-23-agent/.
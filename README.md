> **Note**<br>
> 本文档译自Meta AI Research维护的[Faiss官方Wiki](https://github.com/facebookresearch/faiss/wiki)，结合中文技术文档规范进行本地化适配。

# Faiss

Faiss 是一个用于高效相似性搜索和密集向量聚类的库。它包含可在任意规模向量集中进行搜索的算法，包括那些可能无法完全载入内存的超大规模数据集。该库同时提供评估和参数调优的支持代码，采用 C++ 编写并完整支持 Python 2/3 封装，部分核心算法提供 GPU 实现。项目由 [Meta AI Research](https://research.facebook.com/) 主导开发，并得到外部[贡献者](https://github.com/facebookresearch/faiss/graphs/contributors)支持。

## 目录

- [相似性搜索原理](./similarity_search.md)
- [Faiss 研究基础](./research_foundations.md)
- [开始](./getting-started.md)

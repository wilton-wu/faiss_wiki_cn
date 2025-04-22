> **Note**<br>
> 本文档译自Meta AI Research维护的[Faiss官方Wiki](https://github.com/facebookresearch/faiss/wiki)，结合中文技术文档规范进行本地化适配。<br>
> **最后更新**：2025-04-22

# Faiss

Faiss 是一个用于高效相似性搜索和密集向量聚类的库。它包含可在任意规模向量集中进行搜索的算法，包括那些可能无法完全载入内存的超大规模数据集。该库同时提供评估和参数调优的支持代码，采用 C++ 编写并完整支持 Python 2/3 封装，部分核心算法提供 GPU 实现。项目由 [Meta AI Research](https://research.facebook.com/) 主导开发，并得到外部[贡献者](https://github.com/facebookresearch/faiss/graphs/contributors)支持。

# 相似性搜索原理

给定 d 维空间中的向量集合 $\\{ x_1,..., x_n \\}$，Faiss 在内存中构建数据结构（即索引）。构建完成后，当输入新的 d 维向量 x 时，系统将高效执行以下运算：

$$
i = \mathrm{argmin}_i \lVert x - x_i \rVert
$$

其中 $\lVert.\rVert$ 表示欧氏距离（L2 范数）。

在 Faiss 术语中：
- 该数据结构称为 _索引（index）_，具有添加向量的 _add_ 方法
- 所有向量 $x_i$ 的维度必须一致
- 执行 argmin 运算即为索引的 _search_ 操作

核心功能扩展：
- 支持返回前 k 个最近邻（k-NN）
- 批量处理多个查询向量，显著提升吞吐量
- 支持精度与速度的权衡（如接受 10% 错误率以换取 10 倍速度提升）
- 支持最大内积搜索 $\mathrm{argmax}_i\langle x, x_i \rangle$
- 有限支持其他距离度量（L1、Linf 等）
- 范围搜索（返回半径内的所有近邻）
- 磁盘存储索引
- 二进制向量索引
- 向量 ID 谓词过滤

# Faiss 研究基础

Faiss 整合了二十年来相似性搜索领域的突破性成果，主要包括：

* **倒排文件系统**
  * [Video google: A text retrieval approach to object matching in videos.](http://ieeexplore.ieee.org/abstract/document/1238663/)（Sivic & Zisserman, ICCV 2003）  
  * 实现大规模数据集的非穷举搜索，突破传统暴力搜索的算力瓶颈

* **乘积量化 PQ**
  * [Product quantization for nearest neighbor search](https://hal.inria.fr/inria-00514462v2/document)（Jégou et al., PAMI 2011）
  * 高维向量有损压缩技术，在压缩域实现高精度距离计算

* **三级量化 IVFADC-R** 
  * [Searching in one billion vectors: re-rank with source coding](https://arxiv.org/pdf/1102.3828)（Tavenard et al., ICASSP'11）
  * 多级量化架构实现精度与速度的平衡

* **倒排多索引**  
  * [The inverted multi-index](http://ieeexplore.ieee.org/abstract/document/6248038/)（Babenko & Lempitsky, CVPR 2012）
  * 显著提升倒排索引在高速/低精度场景下的性能

* **优化乘积量化** 
  * [Optimized product quantization](http://ieeexplore.ieee.org/abstract/document/6678503/)（He & al., CVPR 2013）
  * 通过向量空间线性变换增强乘积量化器的索引适配性

* **多义码预过滤**
  * [Polysemous codes](http://link.springer.com/chapter/10.1007/978-3-319-46475-6_48)（Douze & al., ECCV 2016）
  * 在PQ距离计算前增加二进制过滤阶段以提升效率

* **GPU加速框架**
  * [Billion-scale similarity search with GPUs](https://arxiv.org/abs/1702.08734)（Johnson & al., ArXiv 1702.08734）
  * 基于GPU的大规模相似性搜索框架，集成快速k值选择机制

* **HNSW索引**
  * [Efficient and robust approximate nearest neighbor search using Hierarchical Navigable Small World graphs](https://arxiv.org/abs/1603.09320)（Malkov & al., ArXiv 1603.09320）
  * 基于层级导航小世界图的高效近似最近邻搜索

* **SIMD寄存器计算**  
  * [Quicker ADC : Unlocking the Hidden Potential of Product Quantization with SIMD](https://arxiv.org/abs/1812.09162)（André et al., PAMI'19）
  * [Accelerating Large-Scale Inference with Anisotropic Vector Quantization](https://arxiv.org/abs/1908.10396)（Guo & Sun et al., ICML'20）
  * 利用SIMD指令集解锁乘积量化潜在性能，联合应用于各向异性向量量化加速

* **二进制多索引哈希**
  * [Fast Search in Hamming Space with Multi-Index Hashing](http://www.cs.toronto.edu/~norouzi/research/papers/multi_index_hashing.pdf)（Norouzi et al., CVPR’12）
  * 汉明空间快速搜索的创新型多索引哈希方案

* **NSG图索引**
  * [Fast Approximate Nearest Neighbor Search With The Navigating Spreading-out Graph](https://arxiv.org/abs/1707.00143)（Cong Fu et al., VLDB 2019）
  * 导航扩散图索引方法，实现高召回率与低内存占用的平衡

* **局部搜索量化**
  * [Revisiting additive quantization](https://drive.google.com/file/d/1dDuv6fQozLQFS2AJoNNFGTH499QIp_vO/view)（Julieta Martinez et al., ECCV 2018）
  * [LSQ++: Lower running time and higher recall in multi-codebook quantization](https://openaccess.thecvf.com/content_ECCV_2018/html/Julieta_Martinez_LSQ_lower_runtime_ECCV_2018_paper.html)（Julieta Martinez et al., ECCV 2018）
  * LSQ++方法降低运行时开销同时提升多码本量化的召回率

* **残差量化器**
  * [Improved Residual Vector Quantization for High-dimensional Approximate Nearest Neighbor Search](https://arxiv.org/abs/1509.05195)（Shicong Liu et al., AAAI'15）
  * 改进型残差向量量化技术，优化高维近似搜索的内存效率

产品量化领域核心文献（综述论文）：[A Survey of Product Quantization](https://www.jstage.jst.go.jp/article/mta/6/1/6_2/_pdf)（Yusuke Matsui, Yusuke Uchida, Hervé Jégou,
Shin’ichi Satoh, ITE transactions on MTA, 2018）

![PQ_variants_Faiss_annotated.png](https://raw.githubusercontent.com/wiki/facebookresearch/faiss/PQ_variants_Faiss_annotated.png)

图中红色标注为Faiss已实现方法（截至2021-08-19新增：SIMD优化选项与加法量化）

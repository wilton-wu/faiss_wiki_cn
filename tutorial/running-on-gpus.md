# 在GPU上运行Faiss

Faiss能够无缝利用您的NVIDIA GPU。

## 单GPU配置

首先声明GPU资源对象，该对象封装了GPU显存的一部分：

```python
res = faiss.StandardGpuResources()  # 创建标准GPU资源对象
```

### GPU索引构建

通过以下方式将CPU索引转换为GPU索引：

```python
# 创建CPU平面索引
index_flat = faiss.IndexFlatL2(d)
# 转换为GPU索引
gpu_index_flat = faiss.index_cpu_to_gpu(res, 0, index_flat)
```

注意：单个GPU资源对象可被多个索引共享使用，前提是这些索引不会并发执行查询操作。

### 索引使用

GPU索引的使用方式与CPU索引完全一致：

```python
gpu_index_flat.add(xb)         # 向索引添加向量
print(gpu_index_flat.ntotal)

k = 4                          # 设置最近邻数量
D, I = gpu_index_flat.search(xq, k)  # 执行搜索操作
print(I[:5])                   # 输出前5个查询结果的索引
print(I[-5:])                  # 输出最后5个查询结果的索引
```

## 多GPU配置

通过以下方式实现多GPU并行计算：

```python
ngpus = faiss.get_num_gpus()
print("可用GPU数量:", ngpus)

# 创建CPU索引
cpu_index = faiss.IndexFlatL2(d)

# 将索引分配到所有GPU
gpu_index = faiss.index_cpu_to_all_gpus(cpu_index)

gpu_index.add(xb)              # 添加向量到索引
print(gpu_index.ntotal)         # 输出索引总数

k = 4                          # 设置最近邻数量
D, I = gpu_index.search(xq, k)  # 执行并行搜索
print(I[:5])                   # 输出前5个查询结果
print(I[-5:])                  # 输出最后5个查询结果
```

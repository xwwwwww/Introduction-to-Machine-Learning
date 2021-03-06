## 实验二：K-MEANS聚类

> 温佳鑫 2017010335 计84

### 1 概要

本次实验的任务是实现基本的K-MEANS聚类算法，对MNIST做聚类。

### 2 实验结果

实验设置：k=10, max_step=300, 预测采用majority voting

指标：每类的precision, recall, f1，整体的acc

- k=10, 预测采用majority voting

  <img src="pic/image-20210509144859183.png" alt="image-20210509144859183" style="zoom:67%;" />

### 3 可视化

基于t-SNE，采500个点，2维。

<img src="pic/cluster.png" alt="cluster" style="zoom:80%;" />

### 4 分析

#### K的选取

| k值  | acc    |
| ---- | ------ |
| 5    | 0.4281 |
| 6    | 0.4304 |
| 7    | 0.4570 |
| 8    | 0.4737 |
| 9    | 0.5781 |
| 10   | 0.5771 |
| 11   | 0.5920 |
| 12   | 0.6403 |
| 13   | 0.6203 |
| 14   | 0.6611 |
| 15   | 0.6769 |
| 16   | 0.6928 |
| 17   | 0.6736 |
| 18   | 0.6934 |
| 19   | 0.6983 |

这里贴出k=19的详细性能，与k=10对比，可以观察到在各类上的指标都有明显提升。

<img src="pic/image-20210509160046849.png" alt="image-20210509160046849" style="zoom: 67%;" />

#### 预测方式

除了投票外，我还尝试了在k=10的设置下用二分图匹配预测，但看起来效果没有更好（

<img src="pic/image-20210509144913670.png" alt="image-20210509144913670" style="zoom: 67%;" />
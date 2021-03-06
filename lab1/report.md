## 实验一：朴素贝叶斯分类器

>  温佳鑫 2017010335 计84

### 1 概要

本次的任务是垃圾邮件二分类，要求用朴素贝叶斯分类器实现。最终我在按照9:1的比例随机划分的测试集中达到了98.62%的准确率。

### 2 实验

#### 数据预处理

用list of dict维护数据集，以json格式存储，其中每项的格式是

```json
{"label": "spam", "data": [word1, word2, ...]}
```

##### 实现技巧

- 使用python自带的email module，简洁的提取正文和发送者邮箱。

- 只保留utf-8可编码的字符。

- 分词使用nltk的word_tokenize

- 预处理的过程，单进程遍历一次需要3分钟，为此我使用了multiprocess

  (实验室的服务器有56核，差不多3s跑完)

#### 模型

##### 基本原理

<img src="pic/image-20210404161915794.png" alt="image-20210404161915794" style="zoom:50%;" />

因此只需从训练集中统计得到P(y), 和P(xi, y)， 比较y取spam和y取ham的概率，即可得到预测的结果。

##### 实现技巧

- 为了避免连乘导致的精度问题，采用log相加的方式计算
- 为了避免零概率问题，加一个epsilon(本次实验中我取1e-10)
  - 零概率问题的详细讨论见ISSUE 2

### 3 实验结果

选取acc, precision, recall, F1作为评价的自动指标。

- positive对应分类为spam

#### 3.1 5折交叉验证

|      | acc    | precision | recall | F1     |
| ---- | ------ | --------- | ------ | ------ |
| 0    | 0.9837 | 0.9907    | 0.9845 | 0.9876 |
| 1    | 0.9841 | 0.9937    | 0.9821 | 0.9879 |
| 2    | 0.9823 | 0.9911    | 0.9821 | 0.9866 |
| 3    | 0.9794 | 0.9918    | 0.9766 | 0.9841 |
| 4    | 0.9841 | 0.9923    | 0.9836 | 0.9879 |
| avg  | 0.9827 | 0.9919    | 0.9818 | 0.9868 |

#### 3.2 部分训练集

以9:1划分训练集，测试集

选取不同的sample rate, 测试效果

| sample rate | acc    | presicion | recall | F1     |
| ----------- | ------ | --------- | ------ | ------ |
| 0.05        | 0.9450 | 0.9880    | 0.9275 | 0.9568 |
| 0.5         | 0.9773 | 0.9934    | 0.9718 | 0.9825 |
| 1           | 0.9863 | 0.9947    | 0.9843 | 0.9895 |

### 4 ISSUES

#### ISSUE 1: 训练集大小如何影响分类器的表现

随着训练集大小的增大， 参与统计的数据增多，统计得到的参数会更接近真实值，因此分类器的准确率，精准率，召回率和F1值都有明显的上升。

| sample rate | acc    | presicion | recall | F1     |
| ----------- | ------ | --------- | ------ | ------ |
| 0.05        | 0.9450 | 0.9880    | 0.9275 | 0.9568 |
| 0.5         | 0.9773 | 0.9934    | 0.9718 | 0.9825 |
| 1           | 0.9863 | 0.9947    | 0.9843 | 0.9895 |

#### ISSUE 2: 零概率问题

由于测试集中可能见到训练集中没有的数据xi, 则根据训练集统计出P(xi|y)会是0， 根据公式计算，这会导致整体概率归0。

以y为spam为例，“由于训练集的垃圾邮件中没有见过字符xi, 则出现字符xi的邮件是垃圾邮件的概率为0”，这样的推理显然不合理。为此需要给这个P(xi|y)设定一个小值。

可以采用laplace平滑，也可以直接设定成一个epsilon。

在大二的人工智能导论课程中我使用了Laplace平滑，本次我尝试了后者，epsilon设定为1e-10。

#### ISSUE 3: 特殊特征

考虑发件人邮箱这个特征, 因为一般垃圾邮件都会来自于qq邮箱，163邮箱这种普通邮箱，而不是公司邮箱，学校邮箱。

于是我针对这个特征做了如下的ablation study

- 仅使用域名

  > 因为根据上述的思路，多样的用户名不但本身意义不大，反而可能干扰正确的统计。

- 使用整个邮箱账号

- 不适用

|                  | acc    | precision | recall | F1     |
| ---------------- | ------ | --------- | ------ | ------ |
| 仅使用域名       | 0.9863 | 0.9947    | 0.9843 | 0.9895 |
| 使用整个邮箱账号 | 0.9828 | 0.9927    | 0.9811 | 0.9869 |
| 不使用           | 0.9767 | 0.9882    | 0.9762 | 0.9822 |

实验结果也印证了我们的想法。

### 5 遇到的问题

#### 问题一：预处理中的邮件格式、编码问题

预处理是非常重要的环节。

为了最大化的保留有效的邮件内容，我没有选择直接用utf-8读&&ignore掉error。因为大部分邮件头中有注明charset,  于是我先二进制读入，再借助email模块中的`message_from_binary_file`, 完成内容的获取。

除了编码问题外，有一些multipart/alternative类型的样本格式错误，导致模块不能正确的parse，得到每个part的content. 我搜索了相关的问题，但没有找到更好的解法，所以选择直接返回空的body。

#### 问题二：设定随机种子之后结果依然不固定

为了加快预处理速度，我用multiprocess改写了预处理代码。

但几次实验发现结果不一致。我确认了自己对`random.seed`的理解无误后，意识到可能是multiprocess不保序。

虽然可以加入一个id的字段，完成后手动恢复顺序，但感觉也没有必要（

因此如果从预处理开始复现的话，结果会有一定程度的波动

#### 问题三：邮箱特征的利用

最初我只想到了利用整个邮箱账号，随后意识到了用户名会干扰这一特征的问题，于是改正，性能也得到了明显的提升。

### 6 总结与收获

本次实验我实现了朴素贝叶斯分类器，即使这个数据集可能很简单，但传统机器学习算法能达到的效果还是让我有些惊讶的。实验中带给我最大收获的应该是特征的选取和利用部分，这种寻找可解释性特征的思路我认为是有广泛意义的。
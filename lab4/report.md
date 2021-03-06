## Report

### 1 实验设计

##### 模型

roberta-base

##### 任务

五分类

##### 输入

summary 特殊分隔符 reviewText

- 注意看了一下summary, 感觉和打分的关联性还是比较强的，因此用一个特殊的分隔符隔开，拼在了输入中

- 这个数据集很长，而且有些脏， 因此感觉截断的短一些可以加快速度，又不会对效果产生明显影响。实验中我按256截断。

##### 预测

最初我用了最基本的方式，argmax得到one-hot的预测结果。后来向张哲昕同学请教，他采用了一个启发式的规则，对类别按概率加权，得到预测值。这种预测方式能使rmse降低几个百分点。感觉在基础模型比较强的情况下，在rmse这个指标上，one-hot的预测值不能充分体现模型的能力, 因此这样启发式的规则可能会有更好的表现。

##### 集成

用了两个不同的随机种子，跑了两次roberta-base, 做了最基本的bagging。

这步集成能让rmse再降几个百分点。

##### 超参数设置

- max_epoch: 10（设了earlystopping, 按验证集的loss保存最佳结果，实验中基本第一个epoch就收敛了）
- batch size: 64

### 2 实验结果

![image-20210529152309342](pic/image-20210529152309342.png)

### 3 不同模型的效果

| 模型                                             | rmse    |
| ------------------------------------------------ | ------- |
| roberta-base, 未加入特殊分隔符，argmax预测       | 0.62048 |
| roberta-base, 未加入特殊分隔符，启发式预测       | 0.53996 |
| roberta-base, 加入特殊分隔符，启发式预测         | 0.52309 |
| 两个roberta-base集成, 加入特殊分隔符，启发式预测 | 0.50903 |
| nezha-base，加入特殊分隔符，argmax预测           | 0.68961 |
| nezha-base和roberta-base集成, argmax预测         | 0.63437 |

### 4 分析和讨论

从实验结果的表格可以观察到

- 特殊分隔符有效果
- 启发式预测相比于argmax预测有效果
- 我还尝试了引入相对位置编码的nezha-base, 但相比于roberta差距明显（跑完才想起来它预训练的参数是在中文上训的）。
- 集成有一定效果。
  - 但不改变模型本身结构，只改变随机种子，集成带来的增益有限
  - 模型本身结构有差异时，集成带来的增益通常会更大。

分类任务其实是最基本的任务，目前学术界都会在few-shot等其他限制条件下做研究，但其实我觉得这个基本任务还是很有趣的，最近在关于分类器的实验中也发现了一些感觉很神奇的实验现象。希望之后对这个最基本的任务能有更深入的理解。

第一次在Kaggle上提交，感觉很方便，谢谢助教团队:wine_glass:


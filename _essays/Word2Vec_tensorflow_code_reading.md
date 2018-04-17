---
title: "Word2Vec代码阅读"
collection: essays
permalink: /essays/word2vec_tensorflow_code_reading/
author_profile: true
---


From [tensorflow/tutorials](https://github.com/tensorflow)    

## Word2vec_basic.py
[代码链接](https://github.com/tensorflow/tensorflow/blob/master/tensorflow/examples/tutorials/word2vec/word2vec_basic.py)
### maybe_download:
根据路径和文件名下载数据集，返回本地文件名。

### read_data():
读取数据，返回数据中单词的字符串列表。

### build_dataset():
整理数据，建立指定大小的字典，返回值：

* `data`：整理后的数据集，存储文本中每个单词对应的序号的list
* `count`：存储单词出现次数的dict。计算每一个单词在数据集中的出现次数，选取出现次数最多的n-1个单词，其余低频词用UNK代替
* `dictionary`：单词到序号的对应，低频词对应的序号为0
* `reverse_dictionary`:序号到单词的对应

### generate_batch():
为skip-gram模型生成每一批的训练数据，具体过程：

* 建立一个最大长度为`span`（2*`skip_window`+1）双端队列`buffer`作为当前滑动窗口，窗口每次滑动一位
* 每次获取滑动窗口的中心词作为目标单词
* 为每个中心词在其窗口内随机采样`num_skips`次获取上下文
* 相应的滑动窗口，考虑回到数据集开始和结束部分等特殊情况
* 返回目标单词矩阵`batch`和上下文矩阵`labels`，皆为单词序号

### Build model
建立基于tensorflow的skip-gram模型，包括几个部分：

* `inputs`：输入单词序号
* `embeddings`：获得输入单词的表示向量`embed`
* `weights`&`biases`：NCE误差的权重和偏移，可以认为是上下文的向量
* `loss`：获得每一批训练集的误差，基于负采样
* `optimizer`：梯度下降训练
* `norm`: 向量结果归一化
* `init`：初始化所有变量
* `saver`：保存结果

### Train
定义好图之后，进行训练，简化过程：

```
init.run()
for step in xrange(num_steps):
	batch_inputs, batch_labels = generate_batch(batch_size, num_skips, skip_window)
	feed_dict = {train_input: batch_inputs, train_labels: batch_labels}
	_, loss_val = session.run([optimizer, loss], feed_dict=feed_dict)
	average_loss += loss_val
final_embeddings = normalized_embeddings.eval()
```
在源代码中，包括了保存数据、可视化等步骤，这里省略。

********
*2018.04.17*
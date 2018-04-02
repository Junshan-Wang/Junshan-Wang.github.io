---
title: "DeepWalk 代码阅读"
collection: essays
permalink: /essays/deepwalk_gensim_code_reading/
author_profile: true
---


## main.py 
主函数，执行入口

### main( ):
接受参数，执行process

### process(args):
* 加载数据，建图
* 生成游走序列，调用`graph.py`中的`graph.build_deepwalk_corpus()`函数
* Word2vec训练，调用`gensim`模块的`Word2vec`函数或其扩展`skipgram.py`

## graph.py
构建图，`defaultdict`结构的扩展，字典的`(key,value)`为`(node,edge list of node)`

### build\_deepwalk\_corpus():
* `num_paths`：每个结点的游走序列数，即每一轮对所有结点各得到一个游走序列
* `walks`：所有游走序列的列表

### random\_walks():
* `start`：初始结点
* `path_length`：游走序列的长度
* `path`：游走序列的结点列表，从初始结点开始，有一定的随机概率回到初始结点，如果长度达到最大长度或者当前结点没有边，则序列结束

---
才发现该[代码](https://pypi.python.org/pypi/deepwalk)的实现是基于`gensim`而不是`tensorflow`  

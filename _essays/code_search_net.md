---
title: "Code Search Net说明"
collection: essays
permalink: /essays/code_search_net/
author_profile: true
---

### Overview

* Github link：[https://github.com/github/CodeSearchNet](https://github.com/github/CodeSearchNet)
* 提交系统：[https://app.wandb.ai/github/codesearchnet/benchmark](https://app.wandb.ai/github/codesearchnet/benchmark)
* 论文：[https://arxiv.org/abs/1909.09436](https://arxiv.org/abs/1909.09436)

### 环境配置

1. 需要安装docker和Nvidia-Docker，其中docker需要指定19.03版本。在安装的过程中系统的nvidia-driver可能会更新，导致版本不一致问题：
   * `docker run --gpus all nvidia/cuda:10.0-base nvidia-smi`会报错：`E: Couldn't find any package by glob 'nvidia-driver-440.33.01'`
   * `nvdia-smi`也会报错：`Failed to initialize NVML driver/library version mismatch`
   * `cat /proc/driver/nvidia/version`查看显卡驱动所使用的内核版本，`cat /var/log/dpkg.log | grep nvidia`查看电脑驱动，参考[https://blog.csdn.net/qq_40200387/article/details/90341107](****)
   * 尝试了多种方法，最后应该是卸载并重新安装了对应版本的driver后，注意要重启机器`sudo reboot now`🤦‍♀️
2. 安装成功后根据指令执行即可：`script/setup`会下载相关数据集，并构造进入相应的container，`script/console`进入相应的container，该过程需要在每次运行代码前都执行
3. 运行前可以选择是否要登录W&B账号，从而可以上传到系统，查看运行可视化结果，并提交
4. 在docker容器内运行时，不能直接退出，即使用了`nohup`也程序也会终止。可以按`Ctrl+P+Q`退出但不关闭容器，然后再需要重新进入容器时，通过`docker ps -a`查找正在运行的容器id，然后`docker attach id`进入指定容器

### 代码相关

* 代码架构：code encoder + comment encoder + loss
* 流程：
  1. Preprocess
  2. Token embeddings：Neural bag of word (default), Bidirectional RNN, 1D CNN, Self attention
  3. Pooling: mean / max / weighted
  4. Train: (code, query) -> distance metric: cosine / softmax / max-margin
  5. Test: search nearest neighbor

### 结果测试

* 在训练过程中会计算MRR值（一组正例对应999个负例）
* 最终提交的结果，是在给定了99个queries，然后返回这些queries的6种语言上预测结果，计算NDCG值（在整个代码库中查找）
* 运行`python predict.py`后，会生成`predictions.csv`文件，注意到在代码里是通过wandb API上传至服务器，从而可以直接提交。如果是用自己的预测方法，也要用类似的方法上传。
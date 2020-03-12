---
title: "Researches"
permalink: /researches/
author_profile: true
---

## Publications
1. Junshan Wang*, Yilun Jin*, Guojie Song, Xiaojun Ma. EPNE: Evolutionary Pattern Preserving Network Embedding. Accepted by *ECAI 2020*.
2. Junshan Wang, Zhicong Lu, Guojie Song, Yue Fan, Lun Du. [Tag2Vec: Learning Tag Representations in Tag Networks](https://dl.acm.org/citation.cfm?doid=3308558.3313622). Accepted by *WWW 2019*. 
3. Guojie Song, Yuanhao Li, Junshan Wang, Lun Du. [Inferring Explicit and Implicit Social Ties Simultaneously in Mobile Social Network](http://scis.scichina.com/en/2020/149101.pdf). Accepted by *Science China Information Sciences 2020*.
4. Junshan Wang, Guojie Song. [A Deep Spatial-Temporal Ensemble Model for Air Quality Prediction](https://ac.els-cdn.com/S0925231218307859/1-s2.0-S0925231218307859-main.pdf?_tid=f099e8c8-cf4e-4dd5-905c-adfca0ab3871&acdnat=1551863389_55de715a9a5e0f012ccc8ef3e46c7b03). Accepted by *Neurocomputng 2018*. 
5. Lun Du, Yun Wang, Guojie Song, Zhicong Lu, Junshan Wang. [Dynamic Network Embedding: An Extended Approach for Skip-gram based Network Embedding](https://www.ijcai.org/proceedings/2018/0288.pdf). Accepted by *IJCAI 2018*. 


## Researches

### Tag2Vec: Learning Tag Representations in Tag Networks
Network embedding is a method to learn low-dimensional repre- sentation vectors for nodes in complex networks. In real networks, nodes may have multiple tags but existing methods ignore the abundant semantic and hierarchical information of tags. This in- formation is useful to many network applications and usually very stable. In this paper, we propose a tag representation learning model, Tag2Vec, which mixes nodes and tags into a hybrid network. Firstly, for tag networks, we define semantic distance as the proximity between tags and design a novel strategy, parameterized random walk, to generate context with semantic and hierarchical informa- tion of tags adaptively. Then, we propose hyperbolic Skip-gram model to express the complex hierarchical structure better with lower output dimensions. We evaluate our model on the NBER U.S. patent dataset and WordNet dataset. The results show that our model can learn tag representations with rich semantic information and it outperforms other baselines.

### A Deep Spatial-Temporal Ensemble Model for Air Quality Prediction
Air quality has drawn much attention on the recent years because it seriously affects people’s health. Nowadays, people strongly desire air quality prediction, which is a challenging problem as it depends on several complicated factors, such as weather patterns and spatial-temporal dependencies of air quality. In this paper, we design a data-driven approach that utilizes historical air quality and meteorological data to predict air quality in the future. We propose a deep spatial-temporal ensemble(STE) model which is comprised of three components. The first component is an ensemble method to handle different weather patterns. The second one is spatial correlation exploration. The last one is a temporal predictor based on LSTM. We evaluate our model with data from 35 monitoring stations in Beijing, China. The experiments show that each component of our model makes contribution to the improvement in prediction accuracy and the model is superior to baselines.

### Inferring Explicit and Implicit Social Ties Simultaneously in Mobile Social Network
In mobile social network, social tie recognition is a significant task. A great portion of relations are hidden from the interactions in mobile social network. Inferring these implicit social ties can be applied in many aspects like recommender system. But it is challenging because there isn’t any explicit interaction information of these implicit ties in mobile social network. In this paper, we propose a community-based recognition model to infer explicit and implicit social ties simultaneously. Firstly, we observe community features by empirical analysis. It shows that communities with different relation types have different features. Secondly, we propose a communal factor graph to infer social ties. We extract community features, build up a community layer that treats each community as a node. We experiment our model on both implicit and explicit ties and results confirm that using our community features based model can reach a better recognition result.

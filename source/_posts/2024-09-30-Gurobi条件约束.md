---
title: Gurobi条件约束
date: 2024-09-30 13:22:13
tags: 
    - Gurobi
    - 建模
---

前几天刷到一个公众号上的推文，关于CVRP的两索引的建模，其中有一个约束是条件约束。
之前我遇到条件约束都是手工转成线性约束，然后输入gurobi中，一方面是锻炼自己线性化的能力，另一方面是懒的翻手册查找方法。
然后发现Gurobi是真好用，内置了条件约束的添加方法。

条件约束的广义形式见文章末尾的参考来源，这里只是举一个例子, 如下：

$$
x_{ij} = 1 \rightarrow u_i + q_i = u_j, \quad i,j \in A, j \ne 0, i \ne 0
$$

上面的约束表明只有在$x_{ij} = 1$的情况下，$u_i + q_i = u_j$才成立，其中$x,u$是决策变量。
Gurobi添加的约束如下（python）：

```python
m.addConstrs((X[i,j] == 1) >> (Y[i] + demand[j] == Y[j]) for i,j in A if i!=1 and j!= 1)
```
方法不止这一种，上述的是重载的形式，原来的形式应该使用```addGenConstrIndicator```这个方法，详见参考链接。



参考文章：
https://gurobi.com/documentation/current/refman/py_model_agc_indicator.html
---
title: Latex排版优化模型
date: 2024-10-22 21:11:31
tags: Latex
---

在使用Latex写作时，经常遇到数学优化模型的排版问题。在一般情况下，我希望写一个具有最小化\最大化的目标公式，
约束、以及“subject to: ”或者“s.t.: ”这样的约束前缀。
完成这样的任务涉及到一些Latex公式对齐的问题, 经过测试，以下两个实现方式效果最佳。

# 公式独立标号

先放源码：

```latex
\begin{align}
  & \min Cx \label{eq:obj} \\
  \mbox{subject to: } \quad & Ax = b \label{eq:constr} \\
  & x \ge 0
\end{align}
```

编译结果如下：

![Latex](model_align.png)

# 公式共用一个编号（适合多个模型的情况）

```latex
\begin{subequations}
\label{eq:optim}
\begin{align}
    \underset{X}{\text{minimize}}
        & \quad \mathrm{trace}(X)   \label{eq:cost}\\
    \text{subject to} 
        & \quad X_{ij} = M_{ij}\,, \; (i,j) \in \Omega\,, \label{eq:const1}\\
        & \quad X \succeq 0 \,. \label{eq:const2}
\end{align}
\end{subequations}
```

编译结果如下：

![Latex](model_one_num.png)
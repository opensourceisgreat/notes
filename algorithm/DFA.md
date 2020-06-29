### 介绍

DFA，全称 Deterministic Finite Automaton 即确定有穷自动机：从一个状态通过一系列的事件转换到另一个状态，即 state -> event -> state。

- 确定：状态以及引起状态转换的事件都是可确定的，不存在“意外”。
- 有穷：状态以及事件的数量都是可穷举的。

计算机操作系统中的进程状态与切换可以作为 DFA 算法的一种近似理解。

NFA和DFA区别：NFA中当前状态，获取输入后，到达的下一个状态不唯一；DFA 是唯一的

正则表达式可以转换为NFA，NFA可以转换为DFA，参考视频讲解。

### 应用

关键词匹配，正则表达式就是DFS做算法实现的。



### 参考文档

[DFA和FAM区别](https://blog.csdn.net/cpucooler2011/article/details/50347303)

[视频讲解](https://www.bilibili.com/video/BV1zW411t7YE?from=search&seid=14286046442799584196)


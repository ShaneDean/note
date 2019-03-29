# 简介

[参考1](https://blog.csdn.net/wangdong20/article/details/83443250)
[参考2](http://www.csie.ntnu.edu.tw/~u91029/DirectedAcyclicGraph.html)
[参考3](https://www.geeksforgeeks.org/topological-sorting/)
[参考4](https://oi-wiki.org/graph/topo/)
[参考5](https://blog.csdn.net/lisonglisonglisong/article/details/45543451)

拓扑排序英文为Topological sorting，用于解决给一个图的所有节点排序的问题。

可以用大学选课的例子来解释：

![cs_major](https://github.com/ShaneDean/file/blob/master/blog/algorithm/algorithm_toposort_computer_science_major.png?raw=true)

我们要完成一系列任务，单个任务之间可能是依赖关系，比如要完成任务B的前提是完成任务A，每次只能开始能进行的任务。就像完成学校学位所需要完成的课程。拓扑排序就是通关计算机程序来寻找完成这些任务的顺序，还可以延伸到许多不同的问题

假设B任务必须依赖A才能完成，也就是说B一定在A之后完成，即(A,B)表示B依赖于A。

假设我们有任务A B C D E F G

彼此之间的依赖关系为(A,B) (C,D) (A,C) (C,E) (E,G) (F,G) (B,E) (D,F)

可以根据依赖做出如下的图

![abcdefg](https://github.com/ShaneDean/file/blob/master/blog/algorithm/algorithm_toposort_abcdefg.png?raw=true)



现在需要计算机程序来输入这些任务以及依赖关系并计算出可行的完成任务顺序来完成所有的这些程序。

## TODO

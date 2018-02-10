# 数据库

## 使用 mysql 索引都有哪些原则 ?? 索引是什么数据结构 ?? B+tree 和 B tree 什么区别？
索引可分为`聚集索引和非聚集索引`。

先解释一下索引的基本实现原理：
* 聚集索引的叶子结点是数据行，非聚集索引的叶子结点是聚集索引id。聚集索引天然是是唯一索引
* 对`聚集索引进行查询`，可以`快速定位并找到相应的数据行`
* 对非聚集索引进行查询，如果`只需要查询出被索引的字段`，则`不需要重复去查找聚集索引`；如果需要其他字段，则扔需要去查询聚集索引

依照以上的实现原理，可总结的使用原则为：
* 对经常需要作为`唯一 id`，查询出其他数据的列，设为`主键以使用聚集索引`；
* 对经常在 `order by, group by, union, distinct` 中使用的列，可`建立非聚集索引`；
* `避免建立太多索引`，以降低存储和 CUD 的维护成本；
* 如果列太大，可使用前缀建立索引；

索引是什么数据结构？
* B树，二叉搜索树，数据存储在叶子和非叶子结点
* B-树，多叉搜索树，数据存储在叶子和非叶子结点
* B+树，多叉搜索树，`数据存储在叶子结点`

B+ && B- 比 B树 IO 效率要高，一个磁盘块，存储可多路去向，减少整体遍历深度，即 IO 次数少。
![B+树](./B+tree.jpg)

## MySQL 有哪些存储引擎啊？都有哪些区别？要详细！
![innodb-engine](./innodb-engine.jpeg)
![myisam-engine](./myisam-engine.jpeg)
![memory-engine](./memory-engine.jpeg)


引用：
* [@B树，B-树，B+树，B*树](http://www.cnblogs.com/oldhorse/archive/2009/11/16/1604009.html)
* [@The Inno DB Strorage Engine](https://dev.mysql.com/doc/refman/5.6/en/innodb-introduction.html)



1.最接近的三数之和

给你一个长度为 n 的整数数组 nums 和 一个目标值 target。请你从 nums 中选出三个整数，使它们的和与 target 最接近。返回这三个数的和。假定每组输入只存在恰好一个解。

示例 1：

输入：nums = [-1,2,1,-4], target = 1 输出：2 解释：与 target 最接近的和是 2 (-1 + 2 + 1 = 2)。


2.反转链表

给你单链表的头节点 head ，请你反转链表，并返回反转后的链表。

数据库相关

有一张订单表：

CREATE TABLE t_order (

​    id BIGINT PRIMARY KEY AUTO_INCREMENT,

​    user_id BIGINT NOT NULL,

​    money DECIMAL(10,2) NOT NULL,

​    status TINYINT NOT NULL,

​    create_time DATETIME NOT NULL

);

业务查询语句如下：

-- 查询第1001页，每页10条

SELECT *

FROM t_order

WHERE money > 50

ORDER BY create_time

LIMIT 10000, 10;

请分析：

这条 SQL 在大数据量下可能会有什么性能问题？

你会如何优化？

业务编程-用户注册

请使用 Java（Spring Boot 风格） 编写一个 用户注册的 Controller。

要求：

只需要写 Controller 层；

UserService 抽象出来，不需要具体实现；

其他细节（如参数校验、返回值结构、异常处理等）由你自行设计。

针对一个ID来说，Java从Redis中获取，和从MySQL中获取，各个耗时分别是什么？
MySQL底层索引的数据结构是什么？
如何解决MySQL超大分页问题？
MySQL 建立索引的原则
什么样的情况下需要进行分库分表？
为什么量级达到1000万或者数据量大于20G才进行分库分表？
TCP 和 UDP的区别是什么？
Redis zset 数据结构是什么？
TCP 如何保证可靠连接的呢？

作者：杰IT
链接：https://www.nowcoder.com/discuss/816247810759610368
来源：牛客网
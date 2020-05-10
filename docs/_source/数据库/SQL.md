## [182\. 查找重复的电子邮箱](https://leetcode-cn.com/problems/duplicate-emails/)

Difficulty: **简单**

### 题目描述

**SQL架构：**

```sql
Create table If Not Exists Person (Id int, Email varchar(255))
Truncate table Person
insert into Person (Id, Email) values ('1', 'a@b.com')
insert into Person (Id, Email) values ('2', 'c@d.com')
insert into Person (Id, Email) values ('3', 'a@b.com')
```

编写一个 SQL 查询，查找 `Person` 表中所有重复的电子邮箱。

**示例：**

```
+----+---------+
| Id | Email   |
+----+---------+
| 1  | a@b.com |
| 2  | c@d.com |
| 3  | a@b.com |
+----+---------+
```

根据以上输入，你的查询应返回以下结果：

```
+---------+
| Email   |
+---------+
| a@b.com |
+---------+
```

**说明：**所有电子邮箱都是小写字母。

### Solution

Language: **MySQL**

```sql
select Email 
from Person 
group by email 
having count(email) > 1;
```

```sql
select Email from
(
    select Email, count(Email) as num
    from Person 
    group by Email
) as statistic
where num > 1;
```

## 常用的SQL

```sql
-- 它指示数据库只返回不同的值。
SELECT DISTINCT vend_id
FROM Products;

-- LIMIT 5 指示MySQL返回不超过5 行的数据。
-- LIMIT 5 OFFSET 5 指示MySQL 等DBMS 返回从第5 行起的5 行数据。
-- 第一个数字是指从哪儿开始，第二个数字是检索的行数。
SELECT prod_name
FROM Products
LIMIT 5 OFFSET 5;

-- ORDER BY 子句取一个或多个列的名字，据此对输出进行排序。
-- 下面的例子以价格降序来排序产品（最贵的排在最前面）：
SELECT prod_name
FROM Products
ORDER BY prod_price DESC, prod_name;

-- 不匹配检查。列出所有不是供应商DLL01 制造的产品：
SELECT vend_id, prod_name
FROM Products
WHERE vend_id != 'DLL01';

SELECT vend_id, prod_name
FROM Products
WHERE vend_id <> 'DLL01';

-- 范围值检查
SELECT prod_name, prod_price
FROM Products
WHERE prod_price >= 5 AND prod_price <= 10;

-- OR操作符
SELECT prod_name, prod_price
FROM Products
WHERE vend_id = 'DLL01' OR vend_id = ‘BRS01’;

-- 空值检查
SELECT prod_name
FROM Products
WHERE prod_price IS NULL;

-- IN 操作符 （IN 操作符完成了与OR 相同的功能）
SELECT prod_name, prod_price
FROM Products
WHERE vend_id IN ( 'DLL01', 'BRS01' )
ORDER BY prod_name;

-- NOT 操作符  否定其后所跟的任何条件
SELECT prod_name
FROM Products
WHERE NOT vend_id = 'DLL01'
ORDER BY prod_name;

-- 与前面一条等效
SELECT prod_name
FROM Products
WHERE vend_id <> 'DLL01'
ORDER BY prod_name;

-- 百分号（%）通配符  %表示任何字符出现任意次数。
-- 例如，为了找出所有以词Fish 起头的产品
SELECT prod_id, prod_name
FROM Products
WHERE prod_name LIKE 'Fish%';

SELECT prod_id, prod_name
FROM Products
WHERE prod_name LIKE '%bean bag%';

SELECT prod_name
FROM Products
WHERE prod_name LIKE 'F%y';

-- 下划线（_）通配符  但它只匹配单个字符，而不是多个字符。
SELECT prod_id, prod_name
FROM Products
WHERE prod_name LIKE '__ inch teddy bear';

-- 拼接字段，例如结果为 Bear Emporium (USA )
SELECT Concat(vend_name, ' (', vend_country, ')')
FROM Vendors
ORDER BY vend_name;

-- 平均值
SELECT AVG(prod_price) AS avg_price
FROM Products;

-- 统计个数
SELECT COUNT(*) AS num_cust
FROM Customers;

SELECT COUNT(cust_email) AS num_cust
FROM Customers;

-- 最大值
SELECT MAX(prod_price) AS max_price
FROM Products;

-- 最小值
SELECT MIN(prod_price) AS min_price
FROM Products;

-- 和
SELECT SUM(quantity) AS items_ordered
FROM OrderItems
WHERE order_num = 20005;

SELECT SUM(item_price * quantity) AS total_price
FROM OrderItems
WHERE order_num = 20005;

-- 创建分组
SELECT vend_id, COUNT(*) AS num_prods
FROM Products
GROUP BY vend_id;

/* 输出
vend_id  num_prods
-------  ---------
BRS01       3
DLL01       4
FNG01       2
*/

-- 过滤分组
SELECT cust_id, COUNT(*) AS orders
FROM Orders
GROUP BY cust_id
HAVING COUNT(*) >= 2;

SELECT vend_id, COUNT(*) AS num_prods
FROM Products
WHERE prod_price >= 4
GROUP BY vend_id
HAVING COUNT(*) >= 2;

SELECT order_num, COUNT(*) AS items
FROM OrderItems
GROUP BY order_num
HAVING COUNT(*) >= 3
ORDER BY items, order_num;

-- 利用子查询进行过滤。这里注意，where 『order_num』in (select 『order_num』 ····)
SELECT cust_id
FROM Orders
WHERE order_num IN (SELECT order_num
                    FROM OrderItems
                    WHERE prod_id = 'RGAN01');

SELECT cust_name, cust_contact
FROM Customers
WHERE cust_id IN (SELECT cust_id
                FROM Orders
                WHERE order_num IN (SELECT order_num
                                    FROM OrderItems
                                    WHERE prod_id = 'RGAN01'));

-- 计算结果作为子查询
SELECT cust_name,
        cust_state,
        (SELECT COUNT(*)
            ROM Orders
            WHERE Orders.cust_id = Customers.cust_id) AS orders
FROM Customers
ORDER BY cust_name;

-- 创建连结
-- ！！千万不要这么写，这样子写的笛卡尔积很恐怖
SELECT vend_name, prod_name, prod_price
FROM Vendors, Products
WHERE Vendors.vend_id = Products.vend_id;


SELECT Customers.cust_id, Orders.order_num
FROM Customers LEFT OUTER JOIN Orders
ON Customers.cust_id = Orders.cust_id;

SELECT Customers.cust_id, Orders.order_num
FROM Customers RIGHT OUTER JOIN Orders
ON Orders.cust_id = Customers.cust_id;

SELECT Customers.cust_id,
        COUNT(Orders.order_num) AS num_ord
FROM Customers INNER JOIN Orders
ON Customers.cust_id = Orders.cust_id
GROUP BY Customers.cust_id;

-- 使用union
SELECT cust_name, cust_contact, cust_email
FROM Customers
WHERE cust_state IN ('IL','IN','MI')
UNION
SELECT cust_name, cust_contact, cust_email
FROM Customers
WHERE cust_name = 'Fun4All';

-- 插入全部数据
CREATE TABLE CustCopy AS
SELECT * FROM Customers;
```
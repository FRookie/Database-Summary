# 树状结构查询（Oracle）


```java

SELECT DISTINCT * 
FROM (
        SELECT CONNECT_BY_ISLEAF Leaf, MfgNum, ItemNum  
        FROM (SELECT MfgNum,ItemNum 
                FROM TempBOM 
                WHERE SUBSTR(RUNSERIAL, 1, 2) <> 'ML' 
                AND (EXPIRYDATE IS NULL)) 
        START WITH MfgNum = 'M470-053-01-EUA-5'  
        CONNECT BY PRIOR ITEMNUM = MfgNum
        ) b 
WHERE b.Leaf <= 10 
ORDER BY b.Leaf
;
```

> start with 子句：遍历起始条件，有个小技巧，如果要查父结点，这里可以用子结点的列，反之亦然。

> connect by 子句：连接条件。关键词prior，prior跟父节点列parentid放在一起，就是往父结点方向遍历；prior跟子结点列subid放在一起，则往叶子结点方向遍历，parentid、subid两列谁放在“=”前都无所谓，关键是prior跟谁在一起。

> order by 子句：排序，不用多说。

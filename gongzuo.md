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

        { CONNECT BY [ NOCYCLE ] condition [AND condition]... [ START WITH condition ]
        | START WITH condition CONNECT BY [ NOCYCLE ] condition [AND condition]...}

> 解释：

>> start with: 指定起始节点的条件

>> connect by: 指定父子行的条件关系

>> prior: 查询父行的限定符，格式: prior column1 = column2 or column1 = prior column2 and ... ，

>> nocycle: 若数据表中存在循环行，那么不添加此关键字会报错，添加关键字后，便不会报错，但循环的两行只会显示其中的第一条

>> 循环行: 该行只有一个子行，而且子行又是该行的祖先行

>> connect_by_iscycle: 前置条件:在使用了nocycle之后才能使用此关键字，用于表示是否是循环行，0表示否，1 表示是

>> connect_by_isleaf: 是否是叶子节点，0表示否，1 表示是

>> level: level伪列,表示层级，值越小层级越高，level=1为层级最高节点

> start with 子句：遍历起始条件，有个小技巧，如果要查父结点，这里可以用子结点的列，反之亦然。

> connect by 子句：连接条件。关键词prior，prior跟父节点列parentid放在一起，就是往父结点方向遍历；prior跟子结点列subid放在一起，则往叶子结点方向遍历，parentid、subid两列谁放在“=”前都无所谓，关键是prior跟谁在一起。

> order by 子句：排序，不用多说。

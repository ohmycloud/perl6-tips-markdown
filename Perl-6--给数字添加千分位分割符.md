## 使用链式函数调用
---

考虑最简单的一种情况, 不带小数点的数字:

``` perl6
"1234567890".comb.reverse.rotor(3,:partial).map(*.join('')).join(',').comb.reverse.join('')  
# 1,234,567,890
```



使用 `\\` 转义空白, 使代码对齐:

``` perl6
"1234567890".comb\
            .reverse\
            .rotor(3,:partial)\
            .map(*.join(‘’))\
            .join(‘,’)\
            .comb\
            .reverse\
            .join(‘’)\
            .say;

```





## 使用正则表达式
---

comming soon!



## 使用 Grammar
---

comming soon!

## Whatever 是什么?


> Placeholder for unspecified value/parameter - 未指定的值/参数的占位符。


`*` 字面量在 「term」 位置上创建 「Whatever」 对象。
`*` 的大部分魔法来自于 「Whatever 柯里化」. 当 `*` 作为 item 与很多操作符组合使用时, 编译器会把表达式转换为 「WhateverCode」 类型的闭包.

```perl6
my $c = * + 2;          # same as   -> $x { $x + 2 };
say $c(4);              # 6
```

如果一个表达式中有 N 个 `*`, 则会产生一个含有 `N` 个参数的闭包:

```perl6
my $c = * + *;          # same as   -> $x, $y { $x + $y }
```

在复杂的表达式中使用 `*` 也会产生闭包:

```perl6
my $c = 4 * * + 5;      # same as   -> $x { 4 * $x + 5 }
```

在 `*` 号身上调用方法也会产生闭包:

```perl6
<a b c>.map: *.uc;      # same as <a b c>.map: -> $char { $char.uc }
```

前面提到, 不是所有的操作符和语法都会把 `*` 柯里化为 「WhateverCode」。
下面这几种情况, `*` 仍旧是 「Whatever 对象」。

```
例外               Example    What it does

逗号               1,*,2      用一个 * 元素生成一个 Parcel
范围操作符         1..*       Range.new(:from(1), :to(*));
序列操作符         1 ... *    无限列表
智能匹配           1 ~~ *     返回 True
赋值               $x = *     把 * 赋值给 $x
绑定               $x := *    把 * 绑定给 $x
列表重复           1 xx *     生成无限列表
```

范围操作符被特殊处理. 它们不使用 Whatever-Stars 柯里化, 但是它们使用 「WhateverCode」 进行柯里化.

```perl6
say (1..*).WHAT;        # Range
say (1..*-1).WHAT;      # WhateverCode
```

上面的 `*-1` 是作为参数传递了。

下面这些也能使用:

```perl6
.say for 1..*;          # infinite loop
my @a = 1..4;
say @a[0..*];           # 1 2 3 4
say @a[0..*-2];         # 1 2 3
```

因为 Whatever-currying 是纯粹的语法编译器转换, 这不会在运行时把存储的 Whatever-stars 柯里化为 WhateverCodes.

```perl6
my $x = *;
$x + 2;                 # not a closure, dies because
                        # it can't coerce $x to Numeric
```

存储 Whatever-stars 的使用案例像上面提到的那样, 但要把柯里化异常的情况也包含进去. 例如, 如果你想要一个默认的无限序列:

```perl6
my $max    = potential-upper-limit() // *;
my $series = known-lower-limit() ... $max;
```

一个存储后的 `*` 会在智能匹配的特殊情况下生成 WhateverCode. 注意, 正被柯里化的并非真正储存的 `*`, 而是在 LHS 上的 `*`。

```perl6
my $constraint = find-constraint() // *;
my $maybe-always-matcher = * ~~ $constraint;
```

如果这个假定的 find-constraint 没有找到约束, 则 maybe-always-matcher 会对任何东西都返回 True.

```perl6
$maybe-always-matcher(555);      # True
$maybe-always-matcher(Any);      # True
​```


## Whatever Star

当作为一个「项」使用时， 我们把 `*` 叫做 "Whatever"。当不是实际值时，它用作占位符。例如, `1, 2, 3 ... *`，意思是没有终结点的自然数序列。

## Whatever 闭包

Whatever 最强大的用处是 「Whatever」 闭包。

对于 Whatever 没有特殊意义的普通操作符：把 Whatever 当作参数传递时就创建了一个闭包！ 所以，举个例子：

```perl6
* + 1 # 等价于 -> $a { $a + 1 }
* + * # 等价于 -> $a, $b { $a + $b }
```

一个星号占一个坑儿。

```perl6
@list.grep(* > 10)                  # 返回 @list 数组中所有大于 10 的元素
@list.grep( -> $ele { $ele > 10 } ) # 同上, 使用显式的闭包
@list.grep: -> $ele { $ele > 10 }   # 同上, 使用冒号调用方式
@list.grep: * > 10                  # 同上
@list.grep: { $_ > 10 }             # 同上


@list.map(* + *)                    # 返回 @list 数组中每两个元素的和
@list.map( -> $a, $b { $a+$b } )    # 同上, 使用显式的闭包
```

如果给 `@a[ ]` 的方括号里面传递一个闭包， 它会把 `@a` 数组的元素个数作为参数传递并计算！

数组的最后一个元素

```perl6
my @a =  1,22,33,11;
say @a[*-1];
say @a[->$a {$a-1}]; # $a  即为数组@a 的元素个数
```

数组的倒数第二个元素

```perl6
say @a[*-2];
say @a[->$a {$a-2}];
```

所以 `@a[*/2]` 是 `@a` 数组的中间元素, `@a[1..*-2]`  是 `@a` 中不包含首尾元素的其它元素。
`1, 1, * + * ... *`  是一个无限列表, `* + *` 是后来值的生成规则， 最后一个 `*` 表示没有终结测试。

把闭包存储在标量中

```perl6
my $a = -> $b { $b + 1 }
$a(3) # 4

my @add_by_one = @list.map($a); # 对 @list 中的每个元素加 1
```

Perl 6 的列表求值是惰性的,只要你不要求最后一个元素, 无限列表是没问题的。使用绑定 `(:=)` 操作符把列表赋值给变量：

```perl6    
my @fib := 1, 1, * + * ... *
```

如果我稍后要 `@fib[40]` 的值, 会生成足够多的元素以获取数组的第 41 个元素,那些生成的元素会被记忆。尽管未来, 如果列表未绑定给变量, 之前的值会被忘记, 大部分 Perl 6 列表函数能作用并生成惰性列表。

`@a.map` 和 `@a.grep` 都生成「惰性列表」， 即使 `@a` 不是惰性的。
`@fib.grep(* %% 2)` 是一个偶数惰性列表，例如 `@fib Z @a` 生成一个惰性列表： `@fib[0], @a[0], @fib[1], @a[1] ...`。
给 for 循环传递一个无限列表是没问题的， 它会循环直到停止。

但是要注意不能使用嵌套的闭包:

```perl6
my @a = 1 .. 10;
my @b = @a.map: { * ** 2 }
===SORRY!=== Error while compiling:
Malformed double closure; WhateverCode is already a closure without curlies, so either remove the curlies or use valid parameter syntax instead of * at line 2 ------> <BOL><HERE><EOL>
```

注意上面的错误信息, 说的已经很明显了, WhateverCode 已经是一个不带花括号的闭包了, 所以要么移除花括号, 要么使用合法的参数语法代替 `*` 号, 提示信息足够清楚了。所以, 按照提示:

方法一：使用 `$_` 代替 `*` 号

```
> my @b = @a.map: { $_ ** 2 }
[1 4 9 16 25 36 49 64 81 100]
```

方法二：

```
> my @b = @a.map:  * ** 2
[1 4 9 16 25 36 49 64 81 100]
```

方法三, 显式的使用 closure：

```
> my @b = @a.map: -> $item { $item ** 2 }
[1 4 9 16 25 36 49 64 81 100]
```

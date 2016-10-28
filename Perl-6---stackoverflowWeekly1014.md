## Perl 6 List Concatenation without Slip
[Perl 6 List Concatenation without Slip](http://stackoverflow.com/questions/34567902/perl-6-list-concatenation-without-slip)
在 Perl 5 中 `,` 操作符用来连接列表; 然而在 Perl 6 中需要使用 `|` 操作符, 即 slip 操作符。

```perl6
my @a = <a b c>;
my @b = <d e f>;
my @ab = |@a, |@b;
```

有比这短点的写法吗？

- 答案

你可以使用 「flat」sub:

```perl6
my @a  = <a b c>;
my @b  = <d e f>;
my @ab = flat @a, @b;
say @ab.perl; #> ["a", "b", "c", "d", "e", "f"]
my @abf = (@a, @b).flat;
say @abf.perl; #> ["a", "b", "c", "d", "e", "f"]
```

> 注意, flat 可以移除多层嵌套, 如果该层中的值不是标量的话。

```perl6
# 使用无符号变量
my \list = (1,2,(3,4,(5,6)));
say flat list; # (1 2 3 4 5 6)
```

但是使用 `|()` 只能脱去一层：

```perl6
my \list = (1,2,|( (3,4,(5,6)) ));
say list;  # (1 2 3 4 (5 6))
```


## 在 Perl 6 中有多少种方法写 Fibonacci 序列?

[How many ways are there to describe the Fibonacci sequence in Perl 6?](http://stackoverflow.com/questions/3980842/how-many-ways-are-there-to-describe-the-fibonacci-sequence-in-perl-6)


```perl6
my @fibs = (0, 1, -> $a, $b { $a + $b } ... *);
my @fibs = (0, 1, { $^a + $^b } ... *);  
my @fibs = (0, 1, *+* ... *);
```


```perl6
# 迭代
sub fib (Int $n --> Int) {
    $n > 1 or return $n;
    my ($prev, $this) = 0, 1;
    ($prev, $this) = $this, $this + $prev for 1 ..^ $n;
    return $this;
}
```


```perl6
# 递归
use experimental :cached;

 # 定义一个原型, 接收 Int 类型的参数并返回 Int 类型的结果
proto fib (Int $n --> Int) is cached {*}
multi fib (0)  { 0 }
multi fib (1)  { 1 }
multi fib ($n) { fib($n - 1) + fib($n - 2) }
```

```perl6
# 解析
sub fib (Int $n --> Int) {
    constant φ1 = 1 / constant φ = (1 + sqrt 5)/2;
    constant invsqrt5 = 1 / sqrt 5;
 
    floor invsqrt5 * (φ**$n + φ1**$n);
}
```

## Perl 6 的性能现在怎么样了?

[What performance increases can we expect as the Perl 6 implementations mature?](http://stackoverflow.com/questions/3135673/what-performance-increases-can-we-expect-as-the-perl-6-implementations-mature)

5 年之前是非常慢的, 现在已经没有那么慢了：

```perl6
$ perl6 -e 'say [+] 1..10**1000; say now - INIT now'
5000000000000000000000000000000000000000000000 ...
0.007447
```

```perl6
$ perl6 -e 'say [+] (1..100000).list; say now - INIT now'
5000050000
0.13052975
```

## 像 Int(Cool) 这样的类型强制有什么用？
[What is the point of coercions like Int(Cool)?](http://stackoverflow.com/questions/34874779/what-is-the-point-of-coercions-like-intcool)

类型强制能让你在子例程中拥有一个特殊的类型, 但是接收更广的输入。当该子例程被调用时, 参数会被自动转换为更窄的类型。

```perl6
sub double(Int(Cool) $x) {
    2 * $x
}

say double '21';    # 42
say double Any;     # Type check failed in binding $x; expected 'Cool' but got 'Any'
```

这里, `Int` 是参数将被强制的目标类型， 而 `Cool` 类型是子例程的输入参数的类型。

但是这个子例程有什么用呢？ `$x` 不就是 `Int` 类型吗？ 为什么你还约束调用者来为参数实现 `Cool`？

这个子例程所做的就是接收一个 Cool 的子类型的值, 然后尝试把这个值转换为 Int。那个时候它就是一个 Int 类型的值了, 不管它之前是什么类型的值。

所以：

```perl6
sub double ( Int(Cool) $n ) { $n * 2 }
```

真的可以被看作（我认为这是它在 Rakudo 中的实现方式）

```perl6
# Int 是 Cool 的子类型否则它就是 Any 或 Mu 类型
proto sub double ( Cool $n ) {*} # 定义一个原型

# 这儿有你写的内在部分
multi sub double (  Int $n ) { $n * 2 }

# 这儿是编译器为你写的部分
multi sub double ( Cool $n ) {
    # 调用其它 multi, 因为它现在是 Int 类型
    samewith Int($n);
}
```

所以它接受任何 Int, Str, Rat, FatRat, Num, Array, Hash, 等类型. 并在调用  &infix:<*> （带有 $n 和 2）之前尝试把它转换为 Int 类型。

```perl6
say double '  5  '; # 10
say double 2.5;     # 4
say double [0,0,0]; # 6
say double { a => 0, b => 0 }; # 4
```

你可以把它限制为 Cool 而非 Any 类型, 因为所有的 Cool 值本来要为 Int 提供一个强制。

```perl6
( :( Int(Any) $ ) can be shortened to just :( Int() $ ) )
```

你这样做的原因可能是因为你需要在子例程中让它是 Int 类型因为你正调用其它含有不同类型的代码来做不同的事情：

```perl6
sub example ( Int(Cool) $n ) returns Int {
    other-multi( $n ) * $n;
}

multi sub other-multi ( Int $ ) { 10 }
multi sub other-multi ( Any $ ) {  1 }

say example 5;   # 50
say example 4.5; # 40
```

在这种特殊情况下, 你可以把它写为下面的其中之一。

```perl6
sub example ( Cool $n ) returns Int {
    other-multi( Int($n) ) * Int($n);
}

sub example ( Cool $n ) returns Int {
    my $temp = Int($n);
    other-multi( $temp ) * $temp;
}

sub example ( Cool $n is copy ) returns Int {
    $n = Int($n);
    other-multi( $n ) * $n;
}
```

它们中没有一种比使用签名来为你强制类型更清晰。
通常对于这样简单的功能你可以使用如下几种方法中的任意一种, 它会很好地满足你想要的。

```perl6
my &double = * * 2; # WhateverCode
my &double = * × 2; # 同上

my &double = { $_ * 2 };       # bare block
my &double = { $^n * 2 };      # block with positional placeholder
my &double = -> $n { $n * 2 }; # pointy block

my &double = sub ( $n ) { $n * 2 } # anon sub
my &double = anon sub double ( $n ) { $n * 2 } # anon sub with name

my &double = &infix:<*>.assuming(*,2); # 柯里化
my &double = &infix:<*>.assuming(2);

sub double ( $n ) { $n * 2 } # same as :( Any $n )
```

我不太清楚 proto 是干什么用的。
答：当你调用一个 `multi sub` 的时候, 实际上你调用的是一个「原型 sub」(proto sub), 然后这个原型 sub 再调用合适的 multi 候选者。

































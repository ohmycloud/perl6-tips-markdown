## 控制流 gather/take

`gather` 是一个语句或者能返回一序列值的 block 前缀。该值来自于 gather 块中的动态作用域的 take 调用。

```perl6
my $a = gather {
    take 1;
    take 5;
    take 42;
}
say join ', ', @$a;          # 1, 5, 42
```

`gather/take` 能根据上下文按需(惰性地)生成值。如果你想强制惰性求值则使用 `lazy` 子例程或方法。绑定到一个标量或无符号的容器上也会强制惰性求值。
举个例子:

```perl6
my @vals = lazy gather {
    take 1;
    say "Produced a value";
    take 2;
}
say @vals[0];
say 'between consumption of two values';
say @vals[1];

# OUTPUT:
# 1
# between consumption of two values
# Produced a value
# 2
```

`gather/take` 是动态作用域的, 所以你可以从 gather 内部所调用的 subs 或方法中调用 take:

```perl6
sub weird(@elems, :$direction = 'forward') {
    my %direction = (
        forward  => sub { take $_ for @elems },
        backward => sub { take $_ for @elems.reverse },
        random   => sub { take $_ for @elems.pick(*) },
    );
    return gather %direction{$direction}();
}

say weird(<a b c>, :direction<backward> );          # (c b a)
```

如果值需要在调用者这边可变, 那么使用 [take-rw](https://docs.perl6.org/type/Mu#routine_take-rw)

## take-rw

```perl6
sub take-rw(\item)
```

将给定的项返回给闭合的 gather 块, 而不用引入一个新的容器。

```perl6
my @a = 1...3;
sub f(@list){ gather for @list { take-rw $_ } };
for f(@a) { $_++ };
say @a;
# OUTPUT«[2 3 4]»
```

## Perl 6 中的 Block

```perl6
class Block is Code { }
```

[Block](https://docs.perl6.org/type/Block) 是用于小规模代码复用的代码对象。 `Block` 由花括号括起来的一组语句创建。

如果没有显式的签名或占位符参数，那么 `Block` 会把 `$_` 作为位置参数：

```perl6
my $block = { uc $_; };
say $block.WHAT;            # (Block)
say $block('hello');        # HELLO
```

`Block` 和 `->` 或 `<->` 之间还可以拥有签名:

```perl6
my $add = -> $a, $b { $a + $b };
say $add(38, 4);            # 42
```

如果用 `<->` 引入签名, 那么参数默认标记为 `rw`:

```perl6
my $swap = <-> $a, $b { ($a, $b) = ($b, $a) };
my ($a, $b) = (2, 4);
$swap($a, $b);
say $a;                     # 4
```

类型不是 `Routine`（它是 `Block` 的子类）的 Blocks 对于 [return](https://docs.perl6.org/syntax/return)是透明的。

```perl6
sub f() {
    say <a b c>.map: { return 42 };
                   #   ^^^^^^   exits &f, not just the block
}
```

隐式地, 最后一条语句是 Block 的返回值:

```perl6
say {1}(); # 1
```

sink 上下文中的裸块会被自动执行:

```perl6
say 1;
{
    say 2;                  # executed directly, not a Block object
}
say 3;
```

## &?BLOCK	

Which block am I in? - 即我所在的块。有一个例子：

```perl6
for '.' {
    .Str.say when !.IO.d;
    .IO.dir()>>.&?BLOCK when .IO.d # 我们来点递归!
}
```

以上代码中, 从开括号 `{` 之后到闭合括号之前的这个花括号是一个 `Block` 块, `&?BLOCK` 表示当前 Block 块,  `.IO.dir()>>.&?BLOCK` 的意思是对于每个遍历到的目录/文件都执行一次当前块(即花括号)中的代码, 这样就使用 Block 达到了递归! 好神奇。 

![简书支持 SVG 图片](https://docs.perl6.org/images/type-graph-Block.svg)

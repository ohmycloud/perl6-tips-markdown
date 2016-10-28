## (Str) 方法 subst

[subst](https://docs.perl6.org/routine/subst) 取的是单词 substitution(替换)的前5个字符, 意为替换之意。

```perl6
multi method subst(Str:D: $matcher, $replacement, *%opts)
```

返回被调用的那个字符串, 其中 `$matcher` 被 `$replacement` 给替换掉了(或者返回原来的字符串, 如果没有找到匹配的话)。

`subst` 有一个「就地」替换的句法变体, 它被拼写为 `s/matcher/replacement/`。

`$matcher` 可以是一个正则表达式, 或者一个字符串字面值。 `Cool` 类型的非字符串 matcher 会被强转为字符串以用于字面上的匹配。

```perl6
my $some-string = "Some foo";
my $another-string = $some-string.subst(/foo/, "string"); # gives 'Some string'
$some-string.=subst(/foo/, "string"); # in-place substitution. $some-string is now 'Some string'
```

`replacement` 可以是一个闭包:

```perl6
my $i = 41;
my $str = "The answer is secret.";
my $real-answer = $str.subst(/secret/, {++$i}); # The answer to everything
```

下面是 subst 其它用法的例子:

```perl6
my $str = "Hey foo foo foo";
$str.subst(/foo/, "bar", :g);         # 全局替换 - 返回 Hey bar bar bar

$str.subst(/foo/, "no subst", :x(0)); # 有针对性的替换。要替换的次数. 返回未修改过的字符串.
$str.subst(/foo/, "bar", :x(1));      # 只替换第一次出现。

$str.subst(/foo/, "bar", :nth(3));    # 单独替换第 n 个匹配. 替换第三个 foo. 返回 Hey foo foo bar
```

第三个参数传递给散列, 例如 `:g` 被吞噬参数 `*%opts` 接收, 意为 `g => True`。
其中的 `:nth` 副词拥有可读的长得像英语那样的变体:

```perl6
say 'ooooo'.subst: 'o', 'x', :1st; # xoooo
say 'ooooo'.subst: 'o', 'x', :2nd; # oxooo
say 'ooooo'.subst: 'o', 'x', :3rd; # ooxoo
say 'ooooo'.subst: 'o', 'x', :4th; # oooxo
```

subst 还支持下面的副词:

|short | long | meaning |
|:----:|:----:|:--------|
|:g    |:global	|尽可能多次地替换|
|:nth(Int&#124;Callable)| |只替换第n次匹配; 别名: :st, :nd, :rd, 和 :th|
|:ss |:samespace	 |preserves whitespace on substitution|
|:ii |:samecase	|替换时保留大小写|
|:mm |:samemark	|保留字符标记(e.g. 'ü' replaced with 'o' 会导致 'ö')|
|:x(Int&#124;Callable)| |substitute exactly $x matches|

> 注意, 只有在 `s///` 形式中 `:ii` 暗指 `:i` 而 `:ss` 暗指 `:s`. 在方法的形式中 `:s` 和 `:i` 修饰符必须添加到正则表达式中, 而非添加到 subst 方法调用中。

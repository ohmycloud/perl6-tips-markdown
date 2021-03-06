[ reddit 上使用 Perl 6 处理日期](https://www.reddit.com/r/dailyprogrammer/comments/3wshp7/20151214_challenge_245_easy_date_dilemma/)

### 描述
---

下面的日期, 有些使用了  M D Y 格式, 有些使用了 Y M D 格式, 还使用了任意分隔符! 请把这些散乱的文本解析成合适的 ISO 8601  (YYYY-MM-DD) 格式化日期。

假设只有以 4  个数字开头的日期使用 Y M D 格式, 其它的使用 M D Y 格式。

### 输入样本
---

```
2/13/15
1-31-10
5 10 2015
2012 3 17
2001-01-01
2008/01/07
```

### 输出样本
---

```
2015-02-13
2010-01-31
2015-05-10
2012-03-17
2001-01-01
2008-01-07
```

### 扩展挑战 [中级]
---

使用 2014-12-24 作为相对日期的基准。

当添加 days(天数) 时, 要考虑到每月会有不同的天数, 忽略闰年。

当添加月和年时, 使用整个 units, 以至于:

one month before october 10 is september 10

one year after 2001-04-02 is 2002-04-02

one month after january 30 is february 28 (not march 1)

### Sally's inputs:
---

```
tomorrow
2010-dec-7
OCT 23
1 week ago
next Monday
last sunDAY
1 year ago
1 month ago
last week
LAST MONTH
10 October 2010
an year ago
2 years from tomoRRow
1 month from 2016-01-31
4 DAYS FROM today
9 weeks from yesterday
```

### Sally's expected outputs:
---

```
2014-12-25
2010-12-01
2014-10-23
2014-12-17
2014-12-29
2014-12-21
2013-12-24
2014-11-24
2014-12-15
2014-11-24
2010-10-10
2013-12-24
2016-12-25
2016-02-28
2014-12-28
2015-02-25
```

smls 大神给出了完整的 grammar：

``` perl6
my $today = Date.new(2014, 12, 24);

grammar MessyDate {
    rule TOP {
        |    <date>                 { make $<date>.made } # 跟在 regex 后面的花括号是闭包
        | :i <duration> ago         { make $today.earlier: |$<duration>.made     }
        | :i <duration> from <date> { make $<date>.made.later: |$<duration>.made }
    }

    rule date {
        | [ || <month> (<sep>?) <day>   [$0 <year>]?
            || <day>   (<sep>?) <month> [$0 <year>]?
            || <year>  (<sep>?) <month>  $0 <day>    ]
          { make Date.new: $<year>.made//$today.year, |$<month day>».made }

        | :i today          { make $today     }
        | :i yesterday      { make $today - 1 }
        | :i tomorrow       { make $today + 1 }
        | :i last <weekday> { make $today - ($today.day-of-week - $<weekday>.made) % 7 || 7 }
        | :i next <weekday> { make $today + ($<weekday>.made - $today.day-of-week) % 7 || 7 }
        | :i last <unit>    { make $today.earlier: |($<unit>.made => 1) }
        | :i next <unit>    { make $today.later:   |($<unit>.made => 1) }
    }

    rule duration {
        <count> <unit> { make $<unit>.made => $<count>.made }
    }

    token year {
        | <number(4)>        { make +$<number>       } # <number(4)> 是扩展的 <...> 语法, 实际是方法调用
        | <number(2, 0..49)> { make 2000 + $<number> }
        | <number(2, 50..*)> { make 1900 + $<number> }
    }

    token month {
        | <number(1..2, 1..12)> { make +$<number> }
        | :i Jan[uary]?   { make  1 } # [...] 是非捕获分组
        | :i Feb[ruary]?  { make  2 }
        | :i Mar[ch]?     { make  3 }
        | :i Apr[il]?     { make  4 }
        | :i May          { make  5 }
        | :i Jun[e]?      { make  6 }
        | :i Jul[y]?      { make  7 }
        | :i Aug[ust]?    { make  8 }
        | :i Sep[tember]? { make  9 }
        | :i Oct[ober]?   { make 10 }
        | :i Nov[ember]?  { make 11 }
        | :i Dec[ember]?  { make 12 }
    }

    token day { <number(1..2, 1..31)> { make +$<number> } }

    token weekday {
        | :i Mon[day]?    { make 1 }
        | :i Tue[sday]?   { make 2 }
        | :i Wed[nesday]? { make 3 }
        | :i Thu[rsday]?  { make 4 }
        | :i Fri[day]?    { make 5 }
        | :i Sat[urday]?  { make 6 }
        | :i Sun[day]?    { make 7 }
    }

    token sep   { <[-/.\h]> } # <[...]> 是 Perl 6 中的字符类
    token count { (<[0..9]>+) { make +$0 }  |  an? { make 1 } }
    token unit  { :i (day|week|month|year) s? { make $0.lc } }

    multi token number ($digits)        {  <[0..9]> ** {$digits} }
    multi token number ($digits, $test) { (<[0..9]> ** {$digits}) <?{ +$0 ~~ $test }> }
}

for lines() {
    say MessyDate.parse($_).made // "failed to parse '$_'";
}
```

在 grammar 中, 有两个 regex 的变体, `rule` 和 `token`。rule 默认不会回溯.  rule 与 token 的一个重要区别是, `rule` 这样的正则采取了 `:sigspace` 修饰符。 `rule` 实际上是

```
    regex :ratchet :sigspace { ... }
```
的简写.  ratchet 这个单词的意思是: (防倒转的)棘齿, 意思它是不能回溯的!  而 `:sigspace` 表明正则中的空白是有意义的, 而 `token` 实际上是
```
    regex :ratchet { ... }
```
的简写。 所以在 token 中, 若不是显式的写上 `\s`、`\h`、`\n` 等空白符号, 其它情况下就好像空白隐身了一样, 虽然你写了, 但是编译器却视而不见。


`//` 在左侧匹配失败时会在右侧提供一个默认值。

`<number(4)>`  和 `<number(2, 0..49)>` 中使用了扩展了的 `<...>` 元语法。 标识符(例如左面的 number)后面的第一个字符决定了闭合尖括号之前剩余文本的处理。它的底层语义是函数或方法调用, 所以, 如果标识符后面的第一个字符是左圆括号, 那么它要么是方法调用, 要么是函数调用:

`<number(4)>` 等价于 `<number=&number(4)>`

`<number(2, 0..49)>` 等价于 `<number=&number(2, 0..49)>`

``` perl
multi token number ($digits)        {  <[0..9]> ** {$digits} }
multi token number ($digits, $test) { (<[0..9]> ** {$digits}) <?{ +$0 ~~ $test }> }
```
在扩展的 `<...>` 语法中, 一个前置的 `?{` 或 `!{` 标示着代码断言:
```perl
(<[0..9]> ** {$digits}) <?{ +$0 ~~ $test }>
```
等价于：
```perl
(<[0..9]> ** {$digits}) { +$0 ~~ $test or fail }  # + 强制后面的$0为数值上下文, 以匹配 $test 中的数字
```
上面的两句代码, 具名 `regex`, `token`, 或 `rule` 是一个子例程, 所以可以传递 参数给具名 token。

这从标准输入里读取散乱的日期并把对应的 ISO 日期写到标准输出。

它能解析任务描述中的所有日期（包含扩展）, 还有 - 然而, 在它们中我得到 4 个人不同结果。请弄清它们是否是错误的, 并且为什么是错的:

---

2010-dec-7 --> 我得到  2010-12-07 而不是 2010-12-01

last week --> 我得到 2014-12-17 而不是 2014-12-15

1 month from 2016-01-31 --> 我得到  2016-02-29 而不是 2016-02-28

9 weeks from yesterday --> 我得到  2015-02-24 而不是 2015-02-25



有人在评论中问他 `make/made` 是类中的方法吗？

是的, 它们是 Match 类的方法。

### Match objects(注意 object 是复数)
---

每个 regex match(并且通过扩展, 每个 grammar token match)的结果被表示为一个 Match 对象。

通过这个对象你能访问各种信息片段:

- 匹配到的字符串

- 关于输入字符串匹配的开始和结束位置

- 每个位置捕获和具名捕获的sub-matches

- 与这个匹配有关的 AST 片段, 如果有的话

  ### AST 片段

Calling make inside a token/rule, sets the "AST fragment" that will be associated with the current match. Then later, you can get at that associated data by calling .made on the resulting Match object.

在 `token/rule` 里面调用 `make`, 设置将会与当前匹配关联的 "AST  片段"。然后, 你可以通过在当前结果 Match 对象身上调用 `.made` 方法来获取那个关联数据。

This is really just a free-form slot that allows you to store anything you want with the Match object and retrieve it later, though of course it is meant for building an AST like I do here.

这正是自由形式的插槽, 允许你使用 Match 对象存储任何你想要的东西并在以后检索它, 尽管显而易见这意味着像我那样创建一个 AST。

### 在 grammar 中创建 "AST"
---

Each token/rule in my grammar uses .made to retrieve the pieces of data that its sub-rule matches have made, combines them into a larger piece of data, and make's it for its own parent rule to retrieve. And so on.

在我的 grammar 中每个 `token/rule` 使用 `.made` 来取得它的 sub-rule 匹配构建的数据片段, 把它们组合成一个更大的数据片段, 这是为了让它的 parent rule 能检索。等等。

我在每个 token/rule 里面使用这些语法简写来引用 sub-matches 的 Match 对象:

- $0 引用 sub-match（由一个 () 捕获组引起） 的第一个位置处的 Match 对象。refers to the Match object of the first positional sub-match (caused by a ( ) capture group).
- $<date> 引用一个名字为 "date" 的具名 sub-match 的 Match 对象() refers to the Match object of the named sub-match "date" (通过 `<date>` 递归引用 名为 date 的 token 引起).

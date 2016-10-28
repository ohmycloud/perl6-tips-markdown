### [智能匹配](https://desgin.perl6.org/S03.html#Smart_matching)

这儿有一个标准 Perl 6（即在你的编译单元开始的时候所生效的 Perl 方言） 的智能匹配表格。智能匹配通常作用在当前「主题」(topic)上, 即作用在 `$_` 变量上. 在下面的表格中, `$_` 代表 *~~* 操作符的左侧, 或者代表 *given* 的参数, 或者代表其它主题化的参数。 **X** 代表 ~~ 操作符右侧要与之(`$_`)相匹配的模式, 或者在 *when* 后面的模式。(并且, 实际上, ~~ 操作符充当着一个小型的主题(topicalizer); 即, 为了右侧的计算, 它把 `$_` 绑定到左侧的值上。 使用底层的 .ACCEPTS 形式来避免这种主题化.)

第一节包含了特殊的(privileged)语法; 如果匹配能通过那些条目之一完成, 那它就会那样做。 这些特别的语法是通过它们的外形(form)而非它们的类型(type)进行分派的。 否则就使用表格中的剩余部分,并且匹配会根据正常的方法分派规则进行分派。 优化器(optimizer)被允许假定在编译时之后没有定义额外的匹配操作符, 所以, 如果在编译时模式类型就是显而易见的话, 那么跳转表(jump table)就可以被优化。 然而, 只要 *~~* 操作符是 Perl 中少有的几个不使用多重分派的操作符之一, 那么这部分表格的语法仍然有些特殊。 相反, 基于类型的智能匹配被直截了当地分派给了属于 **X** 模式对象的底层方法.

换句话说, 智能匹配首先根据模式(pattern)的外形/形式(form)或类型(下面的X)进行分派(dispatch), 然后那个模式自身决定是否关注和怎样关注主题(`$_`)的类型。 所以, 下表中的第二列实际上是初始(primary)列。 第一列中的 Any 条目标示了模式要么不关心主题的类型, 要么挑选那个条目作为默认项, 因为上面列出的类型越具体，它越不匹配。


```perl6
$_        X         所隐含的匹配类型           Match if (given $_)
======    =====     =====================   ===================
Any       True      ~~ True                 (parsewarn on literal token)
Any       False     ~~ False match          (parsewarn on literal token)
Any       Match     ~~ Successful match     (parsewarn on literal token)
Any       Nil       ~~ Benign failure       (parsewarn on literal token)
Any       Failure   Failure type check      (okay, 与类型相匹配)
Any       *         block 签名匹配            block 成功绑定到 |$_

Any       Callable:($)  item sub truth          X($_)
```

[S03-smartmatch/any-callable.t lines 5–14](https://github.com/perl6/roast/blob/master/S03-smartmatch/any-callable.t#L5-L14)  

```perl6
sub is_even($x) { $x % 2 == 0 }
sub is_odd ($x) { $x % 2 == 1 }

# 这里 Any 代表了数字 4, X 代表了调用子例程所返回的布尔值
say 'scalar sub truth (unary)'                      if 4 ~~  &is_even;
say 'scalar sub truth (unary, negated smart-match)' if 4 !~~ &is_odd;
```

```perl6
Any       Callable:()   simple closure truth    X() (ignoring $_)
```

[S03-smartmatch/any-callable.t lines 15–24](https://github.com/perl6/roast/blob/master/S03-smartmatch/any-callable.t#L15-L24) 

```
Any       Bool      simple truth            X (treats Bool value as success/failure)
```

[S03-smartmatch/any-bool.t lines 5–23](https://github.com/perl6/roast/blob/master/S03-smartmatch/any-bool.t#L5-L23)  

```perl6
Positional  List      lists are comparable    $_ »~~« X (but dwims ** wildcards!)
Any         Match     match success           X (treats Match value as success)
Any         Nil       benign failure          X (treats Nil value as failure)
Any         Failure   malign failure          X (passes Failure object through)
Any         Numeric   数值相等                 +$_ == X
Any         Stringy   字符串相等               ~$_ eq X


Associative Pair      test hash mapping       $_{X.key} ~~ X.value
Any         Pair      测试对象属性              ?."{X.key}" === ?X.value (例如. 文件测试)
```

```perl6
sub is-true() { True };
sub is-false() { False };
say '~~ non-syntactic True' if  0  ~~ is-true();   
say '~~ non-syntactic True' if 'a' ~~ is-true();     
```

```perl6
'a' ~~ .so; # True
0 ~~ .so;   # False
0 ~~ .not;  # True
```

```perl6
# 列表可比较
my @a = "iPad", "iTouch", "iWatch";
say so @a ~~ ("iPad", "iTouch", "iWatch"); # True, 必须是位置和元素都相同才相等
say so @a ~~ ("iPad", "iWatch", "iTouch"); # False


# 判断数值相等
my  $a = 3;
say so $a ~~ 30; # False
say so $a ~~ 3;  # True

# 判断字符串相等
# my $lan = "Perl 6";
say so $lan ~~ "Perl";   # False
say so $lan ~~ "Perl 6"; # True
```

[S03-smartmatch/any-pair.t lines 5–35](https://github.com/perl6/roast/blob/master/S03-smartmatch/any-pair.t#L5-L35)  


```perl6
Set         Set       identical sets          $_ === X
Any         Setty     force set comparison    $_.Set === X.Set

Bag         Bag       identical bags          $_ === X
Any         Baggy     force bag comparison    $_.Bag === X.Bag

Mix         Mix       identical bags          $_ === X
Any         Mixy      force mix comparison    $_.Mix === X.Mix

Associative Array     keys/list are comparable +X == +$_ and $_{X.all}:exists
Callable    Positional list vs predicate      so $_(X)
Any         Positional lists are comparable   $_[] «===» X[]

Hash        Hash      hash mapping equivalent $_ eqv X
Associative Hash      force hash comparison   $_.Hash eqv X
Callable    Hash      hash vs predicate       so $_(X)
Positional  Hash      attempted any/all       FAIL, point user to [].any and [].all for LHS
Pair        Hash      hash does mapping       X{.key} ~~ .value
Any         Hash      hash contains object    X{$_}:exists

Str         Regex     string pattern match    .match(X)
Associative Regex     attempted reverse dwim  FAIL, point user to any/all vs keys/values/pairs
Positional  Regex     attempted any/all/cat   FAIL, point user to any/all/cat/join for LHS
Any         Regex     pattern match           .match(X)

Range       Range     subset range            !$_ or .bounds.all ~~ X (mod ^'s)
```

[S03-smartmatch/range-range.t lines 5–29](https://github.com/perl6/roast/blob/master/S03-smartmatch/range-range.t#L5-L29)

```perl6
Any         Range     in real range           X.min <= $_ <= X.max (mod ^'s)
```

[S03-smartmatch/disorganized.t lines 30–145](https://github.com/perl6/roast/blob/master/S03-smartmatch/disorganized.t#L30-L145)

```perl6
Any         Range     in stringy range        X.min le $_ le X.max (mod ^'s)
Any         Range     in generic range        [!after] X.min,$_,X.max (etc.)

Any         Type      type membership         $_.does(X)

```


[S03-smartmatch/any-type.t lines 5–40](https://github.com/perl6/roast/blob/master/S03-smartmatch/any-type.t#L5-L40)  

```perl6
Signature Signature sig compatibility       $_ is a subset of X      ???
Callable  Signature sig compatibility       $_.sig is a subset of X  ???
Capture   Signature parameters bindable     $_ could bind to X (doesn't!)
Any       Signature parameters bindable     |$_ could bind to X (doesn't!)

Signature Capture   parameters bindable     X could bind to $_

Any       Any       scalars are identical   $_ === X
```

[S03-smartmatch/any-any.t lines 5–29](https://github.com/perl6/roast/blob/master/S03-smartmatch/any-any.t#L5-L29)  


如果没有其它模式要求 X, 最后那个 rule 才会被应用.

所有的智能匹配类型都是itemized; ~~ 和 given/when 都为它们的参数提供了 item 上下文, 并自动线程化(autothread)任何 junctive 匹配, 以使最终对 .ACCEPTS 的分派看不到任何「复数」（plural）. 所以,上面的 $_ 和 X 都是被当作标量的潜在容器对象。 (不过，你可以显式地使 ~~ 亢奋。 在这种情况下, 所有的智能匹配都是使用 .ACCEPTS 的基于类型的分派来完成的, 不是前面表格中的基于形式的分派)

基于类型的底层方法分派的真正形式是 :


```perl6
X.ACCEPTS($_)
```

作为单个分派调用, 这仅仅只关注最初的 X 的类型。 ACCEPT 方法接口是通过 Pattern role 来定义的。 任何组成 Pattern role 的类都可以选择提供单个 ACCEPTS 方法来处理一切, 这对应于上面那些左侧只有一个 Any 条目的模式类型。或者在类中,类也能选择提供多个 ACCEPTS multi-methods, 然后这些会在类中根据 `$_` 的类型进行重新分派.

智能匹配表格主要用于反应编译时识别的形式和类型. 为了避免条目的激增, 表格假设下面的类型会有相似的表现:



```perl6
实际类型                     Use entries for
===========                 ===============
Iterator Seq                Array
SetHash BagHash MixHash     Hash
named values created with
  Class, Enum, or Role,
  or generic type binding   Type
Char Cat                    Str
Int UInt etc.               Num
Byte                        Str or Int
Buf                         Str or Array of Int
```

(注意, 然而, 这些映射可以显式地通过定义合适的 ACCEPTS 方法进行重写。 如果在编译时, 重定义出现的比智能匹配的分析早, 那么优化器也可以可访问到信息)。

如果并且只有在它跟 Unicode 的关系被清楚地声明或类型化时, 一个包含任何ASCII 范围之外的字节或整数的 Buf 类型才可能被静静地提升为 Str 类型用于模式匹配。 这种类型信息可能来自输入文件句柄, 或者 Buf role 可能是一个允许你使用各种已知的编码来实例化 buffers 的参数类型。 在没有这种类型信息的情况下, 你仍然可以跟 buffer 进行模式匹配, 但是任何把 buffer 当作不是整数序列的尝试都是错误的, 这通常会发生警告.

与 Grammar 相匹配会把 grammar 当作类型名, 而不是一个 grammar。 你需要使用 .parse 和 .parsefile 方法来调用一个 grammar.

与 Signature 相匹配不会真的绑定任何变量, 而只是测试那个签名是否能绑定。 要真的绑定到一个签名上, 使用模式 `*` 把绑定代理到 when 语句块里。 在 when 里面与 `*` 相匹配很特殊。它从随后的 block 是否与主题变量相绑定中接收真假, 所以你可以做有序的签名匹配:


```perl6
given $capture {
    when * -> Int $a, Str $b { ... }
    when * -> Str $a, Int $b { ... }
    when * -> $a, $b         { ... }
    when *                   { ... }
}
```


当多重分派无序的语义不足以定义代码的”主从秩序”("pecking order")时, 这会很有用。 注意, 你要么绑定给一个裸的 block 要么绑定给一个箭头 block(pointy  block)。 绑定给裸的 block 很方便的把主题放在 `$_` 中,所以上面最后那种形式等价于一个 default.(占位符参数也能用于裸 block 形式, 尽管它们的类型不能被那样指定.)

没有为 Any 模式定义的模式匹配, 所以,如果你想要一个右侧是 Any 的反转的智能匹配测试, 那么你总是通过显式地调用使用了 `$_` 作为模式的底层 ACCEPTS 方法来获取它. 例如


```perl6
$_       X    想要的匹配类型           右侧所使用的东西
======   ===  ====================   ========================
Callable Any  item sub truth         .ACCEPTS(X) or .(X)
Range    Any  in range               .ACCEPTS(X)
Type     Any  type membership        .ACCEPTS(X) or .does(X)
Regex    Any  pattern match          .ACCEPTS(X)
etc.
```


同样的技巧会允许你倾向于给混合对象以默认匹配规则, 只要你在 `$_` 身上以点方法开头:

```perl6
given $somethingordered {
    when .values.'[<=]'     { say "increasing" }
    when .values.'[>=]'     { say "decreasing" }
}
```

必要时, 你可以定义一个宏来得到”反转的 when”:


```perl6
my macro statement_control:<ACCEPTS> () { "when .ACCEPTS: " }
given $pattern {
    ACCEPTS $a      { ... }
    ACCEPTS $b      { ... }
    ACCEPTS $c      { ... }
}
```

各种提议但弃用了的智能匹配行为可以很容易地(并且我们希望更易读)被模仿成下面这样:


```perl6
$_      X      Type of Match Wanted   What to use on the right
======  ===    ====================   ========================
Array   Num    array element truth    .[X]
Array   Num    array contains number  *,X,*
Array   Str    array contains string  *,X,*
Array   Seq    array begins with seq  X,*
Array   Seq    array contains seq     *,X,*
Array   Seq    array ends with seq    *,X
Hash    Str    hash element truth     .{X}
Hash    Str    hash key existence     .{X}:exists
Hash    Num    hash element truth     .{X}
Hash    Num    hash key existence     .{X}:exists
Buf     Int    buffer contains int    .match(X)
Str     Char   string contains char   .match(X)
Str     Str    string contains string .match(X)
Array   Scalar array contains item    .any === X
Str     Array  array contains string  X.any
Num     Array  array contains number  X.any
Scalar  Array  array contains object  X.any
Hash    Array  hash slice exists      .{X.all}:exists .{X.any}:exists
Set     Set    subset relation        .{X.all}:exists
Set     Hash   subset relation        .{X.all}:exists
Any     Set    subset relation        .Set.{X.all}:exists
Any     Hash   subset relation        .Set.{X.all}:exists
Any     Set    superset relation      X.{.all}:exists
Any     Hash   superset relation      X.{.all}:exists
Any     Set    sets intersect         .{X.any}:exists
Set     Array  subset relation        X,*          # (conjectured)
Array   Regex  match array as string  .Cat.match(X)  cat(@$_).match(X)
```

(注意, .cat 方法和 Cat 类型强转都接收单个对象, 不像 cat 函数, 它作为一个列表操作符, 接收一个句法的列表(或 multilist ) 并展开它. 然而,所有的这些都返回一个 Cat 对象.)

布尔表达式能返回布尔值, 例如比较操作符或一元 ? 操作符.它们可能显式或隐式地引用 `$_`. 如果它们一点也没有引用 `$_`, 那也是 okay 的 — 那种情况下你就使用 switch 结构作为比一串 elsifs 可读性更好的备选分支. 然而, 注意,那意味着你不能这样写:


```perl6
given $boolean {
    when True  {...}
    when False {...}
}
```

因为它总是会选择 True 这种情况. 相反, 使用某些在内部使用的像条件上下文的东西:


```perl6
given $boolean {
    when .Bool == 1 {...}
    when .Bool == 0 {...}
}
```

最好，就使用 if 语句。在任何情况下, 如果你想使用 ~~ 或 when 进行智能匹配, 它会依照句法识别 True 或 False, 并提醒你它不会按你期望的那样做. 编译器也允许在它能检测的范围之内提醒任何其它不测试 $_ 的布尔结构,.

同样地, 任何接收一个 Matcher的函数(诸如 grep) 不会接收一个 Bool 类型的参数, 因为那总是标示着编程错误.(可以使用 * 来匹配任何东西, 如果那就是你想要的. 或者使用一个返回常量布尔值的闭包)

也要注意 regex 匹配不返回 Bool, 而是返回一个能用作布尔值的 Match 对象(或 Nil). 如果想要, 使用显式的 ? 或 so 来强制转为布尔值. Match 对象代表一个成功的匹配, 并被智能匹配当作与 True 的值相同, 类似地, Nil 代表着失败, 并且不能直接用于智能匹配的右侧. 测试 definedness 来代替或使用 * === Nil.

> (这一段可能有误)操作符的主要用途是在布尔上下文中返回一个 boolean-ish 的值。然而，对于某些诸如正则表达式的操作数，在 item 或列表上下文中操作符的使用把上下文转换为那个操作数，以至于，例如。，正则表达式可以返回一个所匹配的子字符串的列表，就像 Perl 5中一样。这可以通过返回一个能在列表上下文中返回一个列表或者在布尔上下文中返回一个布尔值的对象来完成。正则表达式匹配 Match 对象的情况是一种捕获(Capture), 它拥有这些能力。

带有诸如 :g那样的修饰符的正则匹配想返回多个匹配, 使用 List来做 so.就像任何列表一样, 如果有一个或多项, 值就是被计算为真. 如果没有匹配, 返回空列表, 它在布尔上下文中计算为 false.

为了智能匹配, 所有的 Set, Bag, 和 Mix 值和对应的散列类型, SetHash, BagHash, 和 MixHash 是等价的. 即, Hash 容器中键代表唯一对象, 键值代表那些唯一键的重复次数.(当然, Set 能有 0 或 1次重复, 因为保证唯一性). 所以,所有这些 Mixy 类型只比较键, 不比较键值. 使用 eqv 来测试键和键值的相等性.


尽管为了执行智能匹配需要一个检测范围边界的实现, 对于任何按顺序的类型，例如数字，字符串，或版本号，两个 Range 对象智能匹配的结果实际上没有按照边界定义,而是作为在通过设定的间隔所包围的两个值的集合之间的子集关系。如果并且只有所有能被左侧的范围所匹配的潜在元素也能被右侧的范围所匹配结果才被定义为真。因此一个空范围被"过度指定"到什么长度是无关紧要的。如果左侧的范围为空，那么它总是匹配，因为不存在伪造它的值。如果右侧的范围为空，那么只有左侧的范围为空时它才能匹配。

Cat 类型允许你拥有一个无限可扩展的字符串。你可以通过把数组或迭代器「喂」(feeding)给一个 Cat 来匹配数组或迭代器, 它本质上是某种形式的迭代器上的一个 Str 接口。然后一个 Regex 可以与之相匹配就像它是一个普通的字符串一样。正则引擎可以询问那个字符串是否拥有更多字符，并且如果可能的话字符串会从它底层的迭代器中扩展自己。（注意这样的字符串拥有不确定数量的字符， 所以如果你在模式中使用了 `.*`， 或者你询问那个字符串里面有多少个字符，或者甚至你打印整个字符串，它可能感到有必要吞噬完剩余的字符串，这可能也可能不够敏捷。)


「cat」 操作符接收一个(潜在惰性的)列表并返回一个 「Cat」 对象。在字符串上下文中, 这惰性地把列表中的每一个元素强转为字符串, 表现为不确定长度的字符串。 你可以像这样搜索一个 gather:

```perl6
my $lazystr := cat gather for @foo { take .bar }
$lazystr ~~ /pattern/;
```

Cat 接口允许 regex 使用 `<,>` 断言来匹配元素的边界, 并且那个匹配所返回的 StrPos 对象可以被分解为在列表元素中的元素索引和位置。 如果底层的数据结构是一个可变数组, 那么对数组所做的更改(例如通过 shift 或 pop)由 Cat 追踪, 以使元素编号仍然正确。 字符串, 数组, 列表, 序列, 捕获还有树节点都可以通过正则表达式或者或多或少的通过可交换的签名来进行模式匹配。

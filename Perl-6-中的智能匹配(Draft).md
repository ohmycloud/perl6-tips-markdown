### 智能匹配
---



智能匹配通常作用在当前”主题”(topic)上, 即作用在 `$_` 变量上. 在下面的表格中, `$_` 代表 `~~` 操作符的左侧, 或者作为 `given` 的参数,  或者作为其它主题化的参数.  `X` 代表 `~~` 操作符右侧要匹配的模式, 或者在 `when` 后面的模式.(实际上, `~~` 操作符充当着一个小型的主题; 即, 为了右侧的计算, 它把 `$_` 绑定到左侧的值上. 使用底层的 `.ACCEPTS` 形式来避免这种主题化.)



第一节包含了含有享有特权的语法; 如果匹配能通过那些条目之一完成, 那它就会那样做. 这些特别的语法是通过它们的形式而非它们的类型进行分派的. 否则就使用表格中的剩余部分,并且匹配会根据普通的方法分派规则进行分派.  允许优化器(optimizer)假定编译时之后没有定义额外的匹配操作符, 所以, 如果在编译时模式类型就显而易见的, 那么跳转表(jump table)可以被优化. 然而, 这部分表格的语法仍然有点特权的, 跟 `~~` 操作符一样, 是 Perl 中少有的几个不使用多重分派的操作符之一. 相反, 基于类型的智能匹配会被单个地分派给属于 `X` 模式对象的底层方法.



换句话说, 智能匹配首先根据模式(pattern)的`形式`或`类型`(下面的`X`)进行`分派`(dispatch), 然后那个模式自身决定`是否`和`怎样`注意主题(`$_`)的类型. 所以, 下面表格中的第二列就是初始列. 第一列中的 `Any` 条目标示了模式要么不关心主题的类型,  要么把那个条目作为默认值, 因为上面列出的更具体的类型不匹配.



``` perl
    $_          X         Type of Match Implied   Match if (given $_)
    ======      =====     =====================   ===================
    Any         True      ~~ True                 (parsewarn on literal token)
    Any         False     ~~ False match          (parsewarn on literal token)
    Any         Match     ~~ Successful match     (parsewarn on literal token)
    Any         Nil       ~~ Benign failure       (parsewarn on literal token)
    Any         Failure   Failure type check      (okay, matches against type)
    Any         *         block signature match   block successfully binds to |$_
    Any         Callable:($)  item sub truth      X($_)
                                                  S03-smartmatch/any-callable.t lines 5–14
    Any         Callable:()   simple closure truth    X() (ignoring $_)
                                                  S03-smartmatch/any-callable.t lines 15–24

    Any         Bool      simple truth            X (treats Bool value as success/failure)
                                                  S03-smartmatch/any-bool.t lines 5–23

    Any         List      list truth              X (treats list size as success/failure)
    Any         Match     match success           X (treats Match value as success)
    Any         Nil       benign failure          X (treats Nil value as failure)
    Any         Failure   malign failure          X (passes Failure object through)
    Any         Numeric   numeric equality        +$_ == X
    Any         Stringy   string equality         ~$_ eq X
    Any         Whatever  always matches          True
    Associative Pair      test hash mapping       $_{X.key} ~~ X.value
    Any         Pair      test object attribute   ?."{X.key}" === ?X.value (e.g. filetests)
                                                  S03-smartmatch/any-pair.t lines 5–35

    Set         Set       identical sets          $_ === X
    Any         Setty     force set comparison    $_.Set === X.Set
    Bag         Bag       identical bags          $_ === X
    Any         Baggy     force bag comparison    $_.Bag === X.Bag
    Mix         Mix       identical bags          $_ === X
    Any         Mixy      force mix comparison    $_.Mix === X.Mix
    Positional  Array     arrays are comparable   $_ «===» X (dwims * wildcards!)
                                                  S03-smartmatch/array-array.t lines 5–70

    Associative Array     keys/list are comparable +X == +$_ and $_{X.all}:exists
    Callable    Positional list vs predicate      so $_(X)
    Any         Positional lists are comparable   $_[] «===» X[]
    Hash        Hash      hash mapping equivalent $_ eqv X
    Associative Hash      force hash comparison   $_.Hash eqv X
    Callable    Hash      hash vs predicate       so $_(X)
    Positional  Hash      attempted any/all    FAIL, point user to [].any and [].all for LHS
    Pair        Hash      hash does mapping       X{.key} ~~ .value
    Any         Hash      hash contains object    X{$_}:exists
    Str         Regex     string pattern match    .match(X)
    Associative Regex     attempted reverse dwim  FAIL, point user to any/all vs keys/values/pairs
    Positional  Regex     attempted any/all/cat   FAIL, point user to any/all/cat/join for LHS
    Any         Regex     pattern match           .match(X)
    Range       Range     subset range            !$_ or .bounds.all ~~ X (mod ^'s)
                                                  S03-smartmatch/range-range.t lines 5–29

    Any         Range     in real range           X.min <= $_ <= X.max (mod ^'s)
                                                  S03-smartmatch/disorganized.t lines 30–145

    Any         Range     in stringy range        X.min le $_ le X.max (mod ^'s)
    Any         Range     in generic range        [!after] X.min,$_,X.max (etc.)
    Any         Type      type membership         $_.does(X)
                                                  S03-smartmatch/any-type.t lines 5–40

    Signature   Signature sig compatibility       $_ is a subset of X      ???
    Callable    Signature sig compatibility       $_.sig is a subset of X  ???
    Capture     Signature parameters bindable     $_ could bind to X (doesn't!)
    Any         Signature parameters bindable     |$_ could bind to X (doesn't!)
    Signature   Capture   parameters bindable     X could bind to $_
    Any         Any       scalars are identical   $_ === X
                                                   S03-smartmatch/any-any.t lines 5–29
```





如果没有其它模式要求 X, 最后那个 rule 才会被应用.

所有的智能匹配类型都是`itemized`; `~~` 和 `given/when` 都为它们的参数提供了 item (标量) 上下文, 并自动展开任何 junctive 匹配, 以使最终给 `.ACCEPTS` 的分派看不到任何复数. 所以,上面的 `$_` 和 `X` 是能被看作标量的潜在容器对象. (你可以显式地使 `~~` 亢奋, 在这种情况下, 所有的智能匹配是使用 `.ACCEPTS` 的基于类型的分派来完成的, 不是前面表格中的基于形式的分派)



基于类型的底层方法分派真正形式是 :

``` perl
X.ACCEPTS($_)
```

作为单个分派调用, 这仅仅只关注初始化的 X 的类型. `ACCEPT` 方法接口是通过 `Pattern` role 来定义的. 任何组成 `Pattern` role 的类都可以选择提供单个 `ACCEPTS` 方法来处理一切, 这对用于那些上面说的 Any 中只有一个条目模式类型. 

或者在类中,类也能选择提供多个 `ACCEPTS`  multi-methods, 这些然后会在类中根据 `$_` 的类型进行重新分派.



智能匹配表格主要用于反应编译时能识别的形式和类型. 为了避免条目的扩展, 表格假设下面的类型会表现的类似:



| 实际类型                           | 把条目用作               |
| :----------------------------- | :------------------ |
| Iterator Array                 | List                |
| 使用 Class,Enum,Role,或一般类型绑定的具名值 |                     |
| Cat                            | Str                 |
| Int UInt等                      | Num                 |
| BUf                            | Str or Array of Int |

(注意, 然而, 这些映射可以通过显式的 ACCEPTS 方法的定义进行重写. 如果在编译时, 重定义比智能匹配的分析早, 那么信息对优化器也是可访问的).



 一个在 ASCII 范围之外, 包含任何字节或整数的 `Buf` 类型可能会静静地提升为 `Str` 类型用于匹配, 只有在它跟 Unicode 的关系被清楚地声明或类型化时. 这种类型信息可能来自输入文件句柄, 或者 `Buf` role 可能是一个参数类型, 它允许你使用各种编码来实例化 buffers. 在没有这样的类型信息的情况下, 你仍然可以跟 buffer 进行模式匹配, 但是任何把 buffer 当作不是整数序列的尝试都是错误的, 这通常会发生警告.

匹配 Grammar 会把 grammar 当作类型名, 而不是一个 grammar. 你需要使用 `.parse` 和 `.parsefile` 方法来调用一个 grammar.



匹配 `Signature` 不会真的绑定任何变量, 而是测试签名是否能绑定. 要真的绑定到一个签名上, 使用模式  `*` 来代理绑定到 `when`语句块上代替. 匹配 `*` 在 when 里面很特殊 .  随后的 block 是否绑定了主题变量, 所以你可以做有序的签名匹配:



``` perl
given $capture {
        when * -> Int $a, Str $b { ... }
        when * -> Str $a, Int $b { ... }
        when * -> $a, $b         { ... }
        when *                   { ... }
    }
```

 当多重分派无序的语义不足以定义代码的”主从秩序”时, 这会很有用. 注意, 你可以绑定给一个裸的 block 或 一个箭头 block. 绑定给裸的 block很方便的把主题放在 `$_ 中,所以上面最后那个形式等价于一个 default.(占位符参数也能用于裸 block 形式, 尽管它们的类型不能被那样指定.)



没有给 `Any` 模式定义模式匹配,  所以,如果你想要一个反转的智能匹配测试, 右边是一个 `Any`,这时你总是通过显式的调用使用了 `$_` 作为模式的底层 `ACCEPTS` 方法来获取它. 例如:



``` perl
    $_       X    Type of Match Wanted   What to use on the right
    ======  ===  ====================   ========================
    Callable Any  item sub truth         .ACCEPTS(X) or .(X)
    Range    Any  in range               .ACCEPTS(X)
    Type     Any  type membership        .ACCEPTS(X) or .does(X)
    Regex    Any  pattern match          .ACCEPTS(X)
    etc.
```

同样的技巧会允许你倾向于给混合对象以默认匹配规则, 只要你在 `$_` 身上以点方法开头:



``` perl
given $somethingordered {
        when .values.'[<=]'     { say "increasing" }
        when .values.'[>=]'     { say "decreasing" }
    }
```

必要时, 你可以定义一个宏来得到”反转的 when”:



``` perl
my macro statement_control:<ACCEPTS> () { "when .ACCEPTS: " }
    given $pattern {
        ACCEPTS $a      { ... }
        ACCEPTS $b      { ... }
        ACCEPTS $c      { ... }
    }
```



各种建议但不推荐使用的智能匹配行为 可以很容易地(并且我们希望更易读)被模仿成下面这样:



``` perl
    $_      X      Type of Match Wanted   What to use on the right
    ======  ===    ====================   ========================
    Array   Num    array element truth    .[X]
    Array   Num    array contains number  *,X,*
    Array   Str    array contains string  *,X,*
    Array   Parcel array begins /w Parcel X,*
    Array   Parcel array contains Parcel  *,X,*
    Array   Parcel array ends with Parcel *,X
    Hash    Str    hash element truth     .{X}
    Hash    Str    hash key existence     .{X}:exists
    Hash    Num    hash element truth     .{X}
    Hash    Num    hash key existence     .{X}:exists
    Buf     Int    buffer contains int    .match(X)
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

(注意, `.cat` 方法和 `Cat` 类型都强制必须接收单个对象, 不像 `cat` 函数, cat 函数作为一个列表操作符, 接收一个句法的列表(或multilist ) 并展开它. 然而,所有的这些都返回 `Cat` 对象.)



布尔表达式能返回布尔值, 例如比较操作符或一元 `?` 操作符.它们可能显式或隐式地引用 `$_`. 如果它们一点也没有引用 `$_`, 那也是 okay 的 — 那种情况下你就使用 switch 结构作为比一串  elsifs 可读性更好的备选分支. 然而, 注意,那意味着你不能这样写:

``` perl
 given $boolean {
        when True  {...}
        when False {...}
    }
```

因为它总是会选择 `True` 这种情况. 相反, 使用某些像条件上下文的东西在内部使用更好:

``` perl
 given $boolean {
        when .Bool == 1 {...}
        when .Bool == 0 {...}
    }
```

就像使用 `if` 语句一样. 在任何情况下, 如果你想使用 `~~` 或 `when` 进行智能匹配, 它会依照语法识别 True 或 False, 并提醒你那不会按你期望的那样做. 编译器也允许提醒任何其它不测试 `$_` 的布尔结构, 在他能检测的范围之内.



同样地, 任何接收一个 Matcher的函数(诸如 grep) 不会接收一个 Bool 类型的参数, 因为那总是标示着编程错误.(可以使用 `*` 来匹配任何东西, 如果那就是你想要的. 或者使用一个闭包来返回一个常量布尔值)



也要注意 regex 匹配不返回 Bool, 而是返回一个能用作布尔值的 Match 对象(或 Nil). 如果想要, 使用显式的 `?` 或 `so` 来强制转为 布尔值. Match 对象代表一个成功的匹配, 并被智能匹配当作与 True 的值相同, 类似地, Nil 代表着失败, 并且不能直接用于智能匹配的右侧. 测试 definedness 来代替或使用 `* === Nil`.



带有诸如 `:g`那样的修饰符的正则匹配想返回多个匹配, 使用 `List`来做 `so`.就像任何列表一样, 如果有一个或多项, 值就是被计算为真. 如果没有匹配, 返回空列表, 它在布尔上下文中计算为 false.



为了智能匹配, 所有的 `Set`, `Bag`, 和`Mix` 值和对应的散了类型 `SetHash`, `BagHash`, 和`MixHash` 是等价的.即, Hash 容器中键代表唯一对象, 键值代表那些唯一键的重复次数.(当然, Set 能有 0 或 1次重复, 因为保证唯一性). 所以,所有这些 Mixy 类型只比较键, 不比较键值. 使用 `eqv` 代替, 来测试键和键值的相等性. 

Despite the need for an implementation to examine the bounds of a range in order to perform smartmatching, the result of smartmatching two Range objects is not actually defined in terms of bounds, but rather as a subset relationship between two (potentially infinite) sets of values encompassed by the intervals involved, for any orderable type such as real numbers, strings, or versions. The result is defined as true if and only if all potential elements that would be matched by the left range are also matched by the right range. Hence it does not matter to what extent the bounds of a empty range are "overspecified". If the left range is empty, it always matches, because there exists no value to falsify it. If the right range is empty, it can match only if the left range is also empty.尽管为了执行智能匹配, 需要一个实现来检查范围的边界,  两个 Range 对象的智能匹配结果实际上没有按照边界定义,而是作为子集关系

The Cat type allows you to have an infinitely extensible string. You can match an array or iterator by feeding it to a Cat, which is essentially a Str interface over an iterator of some sort. Then a Regex can be used against it as if it were an ordinary string. The Regex engine can ask the string if it has more characters, and the string will extend itself if possible from its underlying iterator. (Note that such strings have an indefinite number of characters, so if you use .* in your pattern, or if you ask the string how many characters it has in it, or if you even print the whole string, it may feel compelled to slurp in the rest of the string, which may or may not be expeditious.)

 cat 操作符接收一个(潜在惰性的)列表并返回一个 Cat 对象.在字符串上下文中, 这惰性地强制列表中的每一个元素为字符串, 字符串的长度不明. 你可以像这样搜索一个 gather:

``` 
 my $lazystr := cat gather for @foo { take .bar }
 $lazystr ~~ /pattern/;
```

Cat 接口允许 regex 使用 `<,>` 断言来匹配元素的边界, Match 对象提供了一种方式, 能获取元素在列表中的索引和位置. 如果底层的数据结构是一个可变数组, 数组的改变由 Cat 追踪, 以至于元素数字仍然正确. 字符串, 数组,列表,序列,捕获还有树节点都可以是正则匹配的模式, 或由签名匹配饿模式





























[S03/Smart matching/any-callable.t lines 5–14](https://github.com/perl6/roast/blob/master/S03-smartmatch/any-callable.t#L5-L14)

``` per
use v6;
use Test;
plan 4;

#L<S03/"Smart matching"/Any Callable:($) item sub truth>
{
    sub is_even($x) { $x % 2 == 0 }
    sub is_odd ($x) { $x % 2 == 1 }
    ok 4 ~~ &is_even,    'scalar sub truth (unary)';
    ok 4 !~~ &is_odd,    'scalar sub truth (unary, negated smart-match)';
    ok !(3 ~~ &is_even), 'scalar sub truth (unary)';
    ok !(3 !~~ &is_odd), 'scalar sub truth (unary, negated smart-match)';
}
```

[S03/Smart matching/any-callable.t lines 15–24](https://github.com/perl6/roast/blob/master/S03-smartmatch/any-callable.t#L15-L24)

``` perl
use v6;
use Test;
plan 2;

#L<S03/"Smart matching"/Any Callable:() simple closure truth>
{
    sub uhuh { 1 }
    sub nuhuh { Mu }

    ok((Mu ~~ &uhuh), "scalar sub truth");
    ok(!(Mu ~~ &nuhuh), "negated scalar sub false");
}

```

[S03/Smart matching/any-bool.t lines 5–23](https://github.com/perl6/roast/blob/master/S03-smartmatch/any-bool.t#L5-L23)

``` perl
use v6;
use Test;
plan 8;

#L<S03/Smart matching/Any Bool simple truth>

{
    sub is-true() { True };
    sub is-false() { False };
    ok   0  ~~ is-true(),      '~~ non-syntactic True';
    ok  'a' ~~ is-true(),      '~~ non-syntactic True';
    nok  0  ~~ is-false(),     '~~ non-syntactic True';
    nok 'a' ~~ is-false(),     '~~ non-syntactic True';
}

{
    nok  0   ~~ .so,           'boolean truth';
    ok   'a' ~~ .so,           'boolean truth';
    ok   0   ~~ .not,          'boolean truth';
    nok  'a' ~~ .not,          'boolean truth';
}

```

[S03/Smart matching/any-pair.t lines 5–35](https://github.com/perl6/roast/blob/master/S03-smartmatch/any-pair.t#L5-L35)

``` perl
use v6;
use Test;
plan 8;

#L<S03/Smart matching/Any Pair test object attribute>
{
    # ?."{X.key}" === ?X.value
    # 意思是:
    # 在对象上调用名为`X.key` 的方法, 用`?`强制为布尔值
    # 并检查是否和 `X.value`的布尔值相同

    class SmartmatchTest::AttrPair {
        has $.a = 4;
        has $.b = 'foo';
        has $.c = Mu;
    }
    my $o = SmartmatchTest::AttrPair.new();
    ok  ($o ~~ :a(4)),      '$obj ~~ Pair (Int, +)';
    ok  ($o ~~ :a(2)),      '$obj ~~ Pair (Int, +)';
    ok !($o ~~ :b(0)),      '$obj ~~ Pair (different types)';
    ok  ($o ~~ :b<foo>),    '$obj ~~ Pair (Str, +)';
    ok  ($o ~~ :b<ugh>),    '$obj ~~ Pair (Str, -)';
    ok  ($o ~~ :c(Mu)),     '$obj ~~ Pair (Mu, +)';
    ok  ($o ~~ :c(0)),      '$obj ~~ Pair (0, +)';
    ok !($o ~~ :b(Mu)),     '$obj ~~ Pair (Mu, -)';
    # not explicitly specced, but implied by the spec and decreed 
    # by TimToady: non-existing method or attribute dies:
    # http://irclog.perlgeek.de/perl6/2009-07-06#i_1293199

}
```

[S03/Smart matching/array-array.t lines 5–70](https://github.com/perl6/roast/blob/master/S03-smartmatch/array-array.t#L5-L70)

``` perl
use v6;
use Test;
plan 36;

#L<S03/Smart matching/arrays are comparable>
{
    ok((("blah", "blah") ~~ ("blah", "blah")), "qw/blah blah/ .eq");
    ok(!(      (1, 2) ~~ (1, 1)    ),    "1 2 !~~ 1 1");
    ok(!(   (1, 2, 3) ~~ (1, 2)    ),    "1 2 3 !~~ 1 2");
    ok(!(      (1, 2) ~~ (1, 2, 3) ),    "1 2 !~~ 1 2 3");
    ok(!(          [] ~~ [1]), "array smartmatch boundary conditions");
    ok(!(         [1] ~~ []),  "array smartmatch boundary conditions");
    ok((           [] ~~ []),  "array smartmatch boundary conditions");
    ok((          [1] ~~ [1]), "array smartmatch boundary conditions");

    ok((1,2,3,4) ~~ (1,*),   'array smartmatch dwims * at end');
    ok((1,2,3,4) ~~ (1,*,*), 'array smartmatch dwims * at end (many *s)');
    ok((1,2,3,4) ~~ (*,4), 'array smartmatch dwims * at start');
    ok((1,2,3,4) ~~ (*,*,4), 'array smartmatch dwims * at start (many *s)');
    ok((1,2,3,4) ~~ (1,*,3,4), 'array smartmatch dwims * 1 elem');
    ok((1,2,3,4) ~~ (1,*,*,3,4), 'array smartmatch dwims * 1 elem (many *s)');
    ok((1,2,3,4) ~~ (1,*,4), 'array smartmatch dwims * many elems');
    ok((1,2,3,4) ~~ (1,*,*,4), 'array smartmatch dwims * many elems (many *s)');
    ok((1,2,3,4) ~~ (*,3,*), 'array smartmatch dwims * at start and end');
    ok((1,2,3,4) ~~ (*,*,3,*,*), 'array smartmatch dwims * at start and end (many *s)');
    ok((1,2,3,4) ~~ (*,1,2,3,4), 'array smartmatch dwims * can match nothing at start');
    ok((1,2,3,4) ~~ (*,*,1,2,3,4), 'array smartmatch dwims * can match nothing at start (many *s)');
    ok((1,2,3,4) ~~ (1,2,*,3,4), 'array smartmatch dwims * can match nothing in middle');
    ok((1,2,3,4) ~~ (1,2,*,*,3,4), 'array smartmatch dwims * can match nothing in middle (many *s)');
    ok((1,2,3,4) ~~ (1,2,3,4,*), 'array smartmatch dwims * can match nothing at end');
    ok((1,2,3,4) ~~ (1,2,3,4,*,*), 'array smartmatch dwims * can match nothing at end (many *s)');
    ok(!((1,2,3,4) ~~ (1,*,3)), '* dwimming does not cause craziness');
    ok(!((1,2,3,4) ~~ (*,5)), '* dwimming does not cause craziness');
    ok(!((1,2,3,4) ~~ (1,3,*)), '* dwimming does not cause craziness');

    # now try it with arrays as well
    my @a = 1, 2, 3;
    my @b = 1, 2, 4;
    my @m = (*, 2, *); # m as "magic" ;-)

    ok (@a ~~  @a), 'Basic smartmatching on arrays (positive)';
    ok (@a !~~ @b), 'Basic smartmatching on arrays (negative)';
    ok (@b !~~ @a), 'Basic smartmatching on arrays (negative)';

    ok (@a ~~  @m), 'Whatever dwimminess in arrays';
    ok (@a ~~ (1, 2, 3)), 'smartmatch Array ~~ List';
    ok ((1, 2, 3) ~~ @a), 'smartmatch List ~~ Array';

    ok ((1, 2, 3) ~~ @m), 'smartmatch List ~~ Array with dwim';

    ok (1 ~~ *,1,*),     'smartmatch with Array RHS co-erces LHS to list';
    ok (1..10 ~~ *,5,*), 'smartmatch with Array RHS co-erces LHS to list';
}
```

[S03/Smart matching/range-range.t lines 5–29](https://github.com/perl6/roast/blob/master/S03-smartmatch/range-range.t#L5-L29)

``` perl6
> (2..3).bounds
2 3
> (2..3).bounds.all
all(2, 3)
> my @a = (1,3,5,8);
1 3 5 8
> @a.bounds;
Method 'bounds' not found for invocant of class 'Array'
> (2..3).bounds.all ~~ 1..4
all(True, True)
```



``` perl
use v6;
use Test;
plan 14;

#L<S03/Smart matching/Range Range subset range>
{
    # .bounds.all ~~ X (mod ^'s)
    # 意思是:
    # 检查 .min 和 .max 是否都在 Range X 里面.
    # (though this is only true to a first approximation, as
    # those .min and .max values might be excluded)

    ok  (2..3 ~~ 1..4),     'proper inclusion +';
    ok !(1..4 ~~ 2..3),     'proper inclusion -';
    ok  (2..4 ~~ 1..4),     'inclusive vs inclusive right end';
    ok  (2..^4 ~~ 1..4),    'exclusive vs inclusive right end';
    ok !(2..4 ~~ 1..^4),    'inclusive vs exclusive right end';
    ok  (2..^4 ~~ 1..^4),   'exclusive vs exclusive right end';
    ok  (2..3 ~~ 2..4),     'inclusive vs inclusive left end';
    ok  (2^..3 ~~ 2..4),    'exclusive vs inclusive left end';
    ok !(2..3 ~~ 2^..4),    'inclusive vs exclusive left end';
    ok  (2^..3 ~~ 2^..4),   'exclusive vs exclusive left end';
    ok  (2..3 ~~ 2..3),     'inclusive vs inclusive both ends';
    ok  (2^..^3 ~~ 2..3),   'exclusive vs inclusive both ends';
    ok !(2..3 ~~ 2^..^3),   'inclusive vs exclusive both ends';
    ok  (2^..^3 ~~ 2^..^3), 'exclusive vs exclusive both ends';
}
```

[S03-smartmatch/disorganized.t lines 30–145](https://github.com/perl6/roast/blob/master/S03-smartmatch/disorganized.t#L30-L145)

``` perl
use v6;
use Test;
plan 42;

=begin pod
This tests the smartmatch operator, defined in L<S03/"Smart matching">
=end pod

sub eval_elsewhere($code){ EVAL($code) }

#L<S03/Smart matching/Any undef undefined not .defined>
{ 
    ok("foo" ~~ .defined, "foo is ~~ .defined");
    nok "foo" !~~ .defined,   'not foo !~~ .defined';
    nok((Mu ~~ .defined), "Mu is not .defined");
}

# TODO: 
# Set   Set
# Hash  Set
# Any   Set
# Set   Array
# Set   Hash
# Any   Hash

# Regex tests are in spec/S05-*

#L<S03/"Smart matching"/in range>
{ 
    # more range tests in t/spec/S03-operators/range.t
    ok((5 ~~ 1 .. 10), "5 is in 1 .. 10");
    ok(!(10 ~~ 1 .. 5), "10 is not in 1 .. 5");
    ok(!(1 ~~ 5 .. 10), "1 is not i n 5 .. 10");
    ok(!(5 ~~ 5 ^..^ 10), "5 is not in 5 .. 10, exclusive");
};

# TODO:
# Signature Signature
# Callable  Signature
# Capture   Signature
# Any       Signature

# Signature Capture  

# reviewed by moritz on 2009-07-07 up to here.

=begin Explanation
You may be wondering what the heck is with all these try blocks.
Prior to r12503, this test caused a horrible death of Pugs which
magically went away when used inside an EVAL.  So the try blocks
caught that case.
=end Explanation

{
    my $result = 0;
    my $parsed = 0;
    my @x = 1..20;
    try {
        $result = all(@x) ~~ { $_ < 21 };
        $parsed = 1;
    };
    ok $parsed, 'C<all(@x) ~~ { ... }> parses';
    ok ?$result, 'C<all(@x) ~~ { ... } when true for all';

    $result = 0;
    try {
        $result = !(all(@x) ~~ { $_ < 20 });
    };
    ok $result,
        'C<all(@x) ~~ {...} when true for one';

    $result = 0;
    try {
        $result = !(all(@x) ~~ { $_ < 12 });
    };
    ok $result, 'C<all(@x) ~~ {...} when true for most';

    $result = 0;
    try {
        $result = !(all(@x) ~~ { $_ < 1  });
    };
    ok $result, 'C<all(@x) ~~ {...} when true for one';
};

# need to test in EVAL() since class definitions happen at compile time,
# ie before the plan is set up.
eval-lives-ok 'class A { method foo { return "" ~~ * } }; A.new.foo',
              'smartmatch in a class lives (RT 62196)';

# RT #69762
{
    ok sub {} ~~ Callable, '~~ Callable (true)';
    nok 68762 ~~ Callable, '~~ Callable (false)';
    ok 69762 !~~ Callable, '!~~ Callable (true)';
    nok sub {} !~~ Callable, '!~~ Callable (false)';

    ok sub {} ~~ Routine, '~~ Routine (true)';
    nok 68762 ~~ Routine, '~~ Routine (false)';
    ok 69762 !~~ Routine, '!~~ Routine (true)';
    nok sub {} !~~ Routine, '!~~ Routine (false)';

    ok sub {} ~~ Sub, '~~ Sub (true)';
    nok 68762 ~~ Sub, '~~ Sub (false)';
    ok 69762 !~~ Sub, '!~~ Sub (true)';
    nok sub {} !~~ Sub, '!~~ Sub (false)';

    ok sub {} ~~ Block, '~~ Block (true)';
    nok 68762 ~~ Block, '~~ Block (false)';
    ok 69762 !~~ Block, '!~~ Block (true)';
    nok sub {} !~~ Block, '!~~ Block (false)';

    ok sub {} ~~ Code, '~~ Code (true)';
    nok 68762 ~~ Code, '~~ Code (false)';
    ok 69762 !~~ Code, '!~~ Code (true)';
    nok sub {} !~~ Code, '!~~ Code (false)';
}
{

    class RT68762 { our method rt68762 {} };

    ok &RT68762::rt68762 ~~ Method, '~~ Method (true)';
    nok 68762            ~~ Method, '~~ Method (false)';
    ok 69762              !~~ Method, '!~~ Method (true)';
    nok &RT68762::rt68762 !~~ Method, '!~~ Method (false)';

}

# RT 72048
{
    role RT72048_role {}
    class RT72048_class does RT72048_role {}

    ok RT72048_class.new ~~ RT72048_role, 'class instance matches role';
    nok RT72048_class.new !~~ RT72048_role, 'class instance !!matches role';
}

ok ("foo" ~~ *) ~~ WhateverCode, 'thing ~~ * autoprimes';
ok ("foo" ~~ *.chars == 3) ~~ Bool, 'thing ~~ WhateverCode is a boolean';
ok ?(* ~~ "foo")('foo'), '* ~~ "foo" is WhateverCode';
```

[S03-smartmatch/any-type.t lines 5–40](https://github.com/perl6/roast/blob/master/S03-smartmatch/any-type.t#L5-L40)

``` perl
#L<S03/Smart matching/type membership>
{ 
    class Dog {}
    class Cat {}
    class Chihuahua is Dog {} # i'm afraid class Pugs will get in the way ;-)
    role SomeRole { };
    class Something does SomeRole { };

    ok (Chihuahua ~~ Dog),      "chihuahua isa dog";
    ok (Something ~~ SomeRole), 'something does dog';
    ok !(Chihuahua ~~ Cat),     "chihuahua is not a cat";
}

# RT #71462
{
    is 'RT71462' ~~ Str,      True,  '~~ Str returns a Bool (1)';
    is 5         ~~ Str,      False, '~~ Str returns a Bool (2)';
    is 'RT71462' ~~ Int,      False, '~~ Int returns a Bool (1)';
    is 5         ~~ Int,      True,  '~~ Int returns a Bool (2)';
    is 'RT71462' ~~ Set,      False, '~~ Set returns a Bool (1)';
    is set(1, 3) ~~ Set,      True,  '~~ Set returns a Bool (2)';
    is 'RT71462' ~~ Numeric,  False, '~~ Numeric returns a Bool (1)';
    is 5         ~~ Numeric,  True,  '~~ Numeric returns a Bool (2)';
    is &say      ~~ Callable, True,  '~~ Callable returns a Bool (1)';
    is 5         ~~ Callable, False, '~~ Callable returns a Bool (2)';
}

# RT 76610
{
    module M { };

    lives-ok { 42 ~~ M }, '~~ module lives';
    ok not $/, '42 is not a module';
}
```

[S03-smartmatch/any-any.t#L5-L29](https://github.com/perl6/roast/blob/master/S03-smartmatch/any-any.t#L5-L29)

``` perl
use v6;
use Test;
plan 8;

#L<S03/Smart matching/Any Any scalars are identical>
{
    class Smartmatch::ObjTest {}
    my $a = Smartmatch::ObjTest.new;
    my $b = Smartmatch::ObjTest.new;
    ok  ($a ~~  $a),    'Any ~~  Any (+)';
    ok !($a !~~ $a),    'Any !~~ Any (-)';
    ok !($a ~~  $b),    'Any ~~  Any (-)';
    ok  ($a !~~ $b),    'Any !~~ Any (+)';
}


{
    $_ = 42;
    my $x;
    'abc' ~~ ($x = $_);
    is $x, 'abc', '~~ sets $_ to the LHS';
    is $_, 42, 'original $_ restored';
    'defg' !~~ ($x = $_);
    is $x, 'defg', '!~~ sets $_ to the LHS';
    is $_, 42, 'original $_ restored';
    'defg' !~~ ($x = $_);
}
```

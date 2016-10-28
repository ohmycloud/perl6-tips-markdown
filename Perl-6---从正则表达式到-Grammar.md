「[原文链接](http://theperlfisher.blogspot.fr/2016/02/from-regular-expressions-to-grammars-pt_28.html)」

如果你是正则表达式新人(至少当它们用于 Perl 6 中时), 那我建议你从这个系列的[第一部分](http://theperlfisher.blogspot.ro/2016/02/from-regular-expressions-to-grammars-pt.html)开始。那些掌握了一定正则表达式的人可以跳过[上周](http://theperlfisher.blogspot.ro/2016/02/from-regular-expressions-to-grammars-pt_20.html)的文章。现在, 继续演示!

## 上周轶事

我们开始开发一个接收诸如

```javascript
var a = 3; console.log("Hey, did you konw a = " + a + "?");
```
 Javascript 表达式的 Perl 6 编译器, 并把这段代码转换为 [Rakudo Perl](http://perl6.org/) 那样的编译器能运行的 Perl 6 代码。在我们开始之前, 想想转换后的 Perl 6 代码看起来是什么样的可能会是个好主意。如果你已经知道了 Perl 5, 那么你应该熟悉这样的代码。

```perl
 my $a = 3;
 say "Hey, did you konw a = " ~ $a ~ "?";
```

 我们将需要确保我们的正则表达式捕获到了 Javascript 的要素。如果你还记得上一次, 我们使用这样一组正则表达式来捕获我们的文本:

```perl6
 my rule Number                { \d+                                                          };
 my rule Variable              { \w+                                                          };
 my rule String                { '"' <-[" ]>+ '"'                                             };
 my rule Assignment-Expression { var <Variable> '=' <Number>                                  };
 my rule Function-Call         { console '.' log '(' <String> '+' <Variable> '+' <String> ')' };

 say 'var a = 3; console.log("Hey, did you konw a = " + a + "?");' ~~ rule { <Assignment-Expression> ';'  <Function-Call> ';' };
 ```

 如果你把这段代码放到一个 Perl 6 源文件中并运行它, 那么它的输出第一次看起来可能会有点奇怪:

```
｢var a = 3; console.log( "Hey, did you know a = " + a + "?" );｣
 Assignment-Expression => ｢var a = 3｣
    Variable => ｢a ｣
    Number => ｢3｣
 Function-Call => ｢console.log( "Hey, did you know a = " + a + "?" )｣
    String => ｢"Hey, did you know a = " ｣
    Variable => ｢a ｣
    String => ｢"?" ｣
```
如果你愿意暂时忽略  「」  标记, 你会看到匹配被缩进了, 几乎像资源管理器窗口一样, '<Assignment-Expression>' 作为目录, 'Variable' 和 'Number' 作为目录里面的文件。实际上, 那离真相不远了。 当我看到这种结构时, 我发现使用一点添加的语法能帮助我们像这样来观察它:

```
$/ => ｢var a = 3; console.log( "Hey, did you know a = " + a + "?" );｣
 <Assignment-Expression> => ｢var a = 3｣
    <Variable> => ｢a ｣
    <Number> => ｢3｣
 <Function-Call> => ｢console.log( "Hey, did you know a = " + a + "?" )｣
    <String> => ｢"Hey, did you know a = " ｣
    <Variable> => ｢a ｣
    <String> => ｢"?" ｣
```

这几乎让怎么打印出文本变得更容易, 并在我们的正则表达式中指出了一个小问题。我们来打印给变量 *a* 所赋的数字, 从这儿开始。第一行告诉我们目录的根, 或者匹配树是 `$/`。 如果你在测试文件的末尾添加上 `say $/;` 并返回它, 那么你会看到整个表达式被打印出了 2 次。 那一定意味着 `$/` 就是整个匹配。

每向下推进一层就是把 `=>` 箭头的左侧的东西添加到 `$/` 的右边。把之前的 `say` 语句修改为 `say  $/<Assignment-Expression>;`, 并看看输出发生了什么改变。它现在看起来应该像这样:

```
｢var a = 3｣
  Variable => ｢a ｣
  Number => ｢3｣
```

让我们把把标记(不可见)添加进来, 所以我们能知道到了哪里...

```
$/<Assignment-Expression> => ｢var a = 3｣
  <Variable> => ｢a ｣
  <Number> => ｢3｣
```


我们现在能看到我们的目标, 数字 3, 仅仅实在更下面的一层。和上次一样, 我们能够添加表达式左侧的东西, 所以我们就动手吧。

```perl6
say $/<Assignment-Expression><Number>;
  ｢3｣
```

我们几乎得到我们想要的了。 「」  挡道, 所以我们在这儿把值转换回数字。我把转换(cast)用引号扩起来, 因为它不是C/C++ 程序员那样认为的"casting"。我们想做的大约等价于 `sscanf(str,"%d",&num)`, 但是在 Perl 6 中, 这个操作符更加简单:

```perl6
say +$/<Assignment-Expression><Number>;
  3
```
如果不深入更多细节, 那么 `$/` 是一个里面藏着隐式数字、字符串和布尔值的对象。前面添加的 `+` 把隐藏在 `$/` 对象中的数字显示出来了。



## 从 Javascript 到 Perl

我们离从 Javascript 生成 Perl 6 代码不远了。让我们使用上面所学的开始我们的第一个语句, 赋值语句。

```perl6
say 'my $' ~ $/<Assignment-Expression><Variable> ~ ' = ' ~
      $/<Assignment-Expression><Number> ~ ';';

my $a = 3;

```

我们仅仅使用了 7 行 Perl 6 就把代码从一种语言转换为另外一种语言。并且大部分的 Perl 6 代码都是可重用的, 因为字符串, 数字, 和 Javascript/C/Java 风格的变量名在大部分语言之间是通用的。

上次, 我们学习了怎么使用正则表达式来创建匹配。这次我们学会了怎么使用我们说匹配到的东西, 还有怎么在 *say* 语句中找出我们想要的东西。 不可见的匹配标记相当有用, 我可能会写一个模块来把它们放回到匹配表达式中, 那应该不难。

那个方案有一个问题, 如果我们看一下 `<Function-Call>` 匹配, 会很容易发现那个问题。

```perl6
$/<Function-Call> => ｢console.log( "Hey, did you know a = " + a + "?" )｣
  <String> => ｢"Hey, did you know a = " ｣
  <Variable> => ｢a ｣
  <String> => ｢"?" ｣
```

当我们写了 `say $/<Function-Call><String>;` 时, 我们会获取哪个 `<String>`? 在你运行这段代码之前, 先猜测一下。会是第一个吗, 因为一旦匹配对象被创建,  Perl 6 就不会把它替换掉? 会是最后一个吗, 因为最后一个"覆盖"了第一个? 编译器会仅仅"感到困惑"然后什么也不打印吗? 运行一下看看!

它实际上以一个列表的形式把两个匹配都返回了, 所以你可以引用任何一个。 我们的不可见标记现在看起来长这样:

```
$/<Function-Call> => ｢console.log( "Hey, did you know a = " + a + "?" )｣
  <String>[0] => ｢"Hey, did you know a = " ｣
  <Variable> => ｢a ｣
  <String>[1] => ｢"?" ｣
```

所以, 如果我们想打印第一个字符串, 我们可以写上 `say $/<Function-Call><String>[0];` 并得到含有时髦的日语标记的   「"Hey, did you know a = " 」。幸运的是有一种便捷方式来避免那些日语标记, 就像数字 3 中的那样:

```perl6
say ~$/<Function-Call><String>[0];
 "Hey, did you know a = "
```

**~** 操作符使匹配字符串化, 就像 `+` 让返回的匹配数字化一样。所以你可能自己把最后一行写作:

```perl6
say 'say ' ~ $/<Function-Call><String>[0] ~ ' ~ '
  ' $' ~ $/<Function-Call><Variable> ~ ' ~ '
  $<Function-Call><String>[1] ~ ';';

say "Hey, did you know a = " ~ $a ~ "?";
```

我们已经把我们的两行 Javascript 代码编译成 Perl 6 代码了。

## 重构

现在已经能工作了, 但是有很多重复。目前我们得到是:

```perl6
my rule Variable               { \w+                                                          };
my rule String                 { '"' <-[ " ]>+ '"'                                            };
my rule Assignment-Expression  { var <Variable> '=' <Number>                                  };
my rule Function-Call          { console '.' log '(' <String> '+' <Variable> '+' <String> ')' };

'var a = 3; console.log( "Hey, did you know a = " + a + "?" );' ~~ rule { <Assignment-Expression> ';' <Function-Call> ';' }

say 'my $' ~ $/<Assignment-Expression><Variable> ~ ' = ' ~ $/<Assignment-Expression><Number> ~ ';';
say 'say ' ~ $/<Function-Call><String>[0] ~ ' ~ $' ~ $/<Function-Call><Variable> ~  ' ~ ' ~ $/<Function-Call><String>[1] ~  ';';
```

那些 rules 看起来相当好,  `<String>` 和 `<Variable>` 的重复也是不可避免的。 但是看看 `say` 语句, 你会看到 `<Assignment-Expression>` 和 `<Function-Call>` 重复了自身好几次。避免这种重复的一种方法是创建一个临时变量, 但是那可能会变得丑陋。

```perl6
my $assignment-expression = $/<Assignment-Expression>;
say 'my $' ~ $assignment-expression<Variable> ~ ' = ' ~  $assignment-expression<Number> ~ ';'
```

相反, 我们利用 Perl 6 的子例程签名, 并且重用 `$/` 变量名以使我们能重用上面所写的代码, 然后拿掉 <Assignment-Expression> 部分。 我会把子例程的名字命名为 rule 的名字, 只是为了直接了当。(你会在之后看到为什么这样做。)

```perl6
sub  assignment-expression($/) {
    'my $' ~ $/<Variable> ~ ' = ' ~ $/<Number> ~ ';'
}

say assignment-expression( $/<Assignment-Expression> );
```

让我们对 <Function-Call> 也做同样的事情, 创建一个含有 `$/` 子例程签名的同名函数。 它现在写在一行里面就很整洁了, 并且只重复 *<String>* 部分, 因为它不得不重复。

```perl6
sub function-call( $/ ) {
     'say ' ~ $/<String>[0] ~ ' ~ ' ~ $/<Variable> ~ ' ~ ' ~ $/<String>[1] ~ ';'
}

say function-call( $/<Function-Call> );
```

## 对象化

一路上我做了相当多的选择,让我们到达这里。这就是我们上次重构的地方:

```perl6
my rule Number                { \d+                                                          };
my rule Variable              { \w+                                                          };
my rule String                { '"' <-[ " ]>+ '"'                                            };
my rule Assignment-Expression { var <Variable> '=' <Number>                                  };
my rule Function-Call         { console '.' log '(' <String> '+' <Variable> '+' <String> ')' };

'var a = 3; console.log( "Hey, did you know a = " + a + "?" );' ~~ rule { <Assignment-Expression> ';' <Function-Call> ';' }

sub assignment-expression( $/ ) {
    'my $' ~ $/<Variable> ~ ' = ' ~ $/<Number> ~ ';'
}

sub function-call( $/ ) {
    'say ' ~ $/<String>[0] ~ ' ~ $' ~ $/<Variable> ~ ' ~ ' ~ $/<String>[1] ~ ';';
}
say assignment-expression( $/<Assignment-Expression> );
say function-call( $/<Function-Call> );
```

这就是我们的回报。我们先捡起最后那两个 `say` 语句。 我们还没有给顶层 rule 一个名字, 所以我们就叫它... 好吧, 现在还是叫 'top' 吧。

```perl6
sub top( $/ ) { assignment-expression( $/ ) ~ function-call( $/ ) }
```

## 收回你的吐槽

我们暂时还没有对处于文件顶层的 rules 做太多处理, 所以让我们开始工作吧。 在 Perl 6 中, 就一般编程而言, 把你的代码打包复用是不错的注意。而 Perl 6 让我们使用 `class` 关键字将我们的程序打包, 我们拥有的那些 rules 从任何意义上来说实际上不是 "代码"。而它们能够用于代码中, 并且我们确实使用了它们, 它们自身实际上并没有做出任何决定。

所以我们不应该使用 `class` 关键字来把它们打包到一块。相反, 有另外一种便捷的类型用于把一堆正则表达式和 rules 打包到一块儿, 它叫做 `grammar`。
它的语法就像声明一个 「rule」 那样。

```perl6
grammar Javascript {
    rule Number                { \d+                                                          };
    rule Variable              { \w+                                                          };
    rule String                { '"' <-[ " ]>+ '"'                                            };
    rule Assignment-Expression { var <Variable> '=' <Number>                                  };
    rule Function-Call         { console '.' log '(' <String> '+' <Variable> '+' <String> ')' };      

    rule TOP                   { <Assignment-Expression> ';' <Function-Call> ';'              };
}
```

你会注意到, 我们给我们的顶层 rule 也起了个名字, 并且暂时把它叫做 「TOP」 吧。 如果你正在家独自玩耍, 你可能已经做出更改并想知道 「'var a = 3;...' ~~ rule { ... }」 是怎么起作用的, 因为键入诸如 「'var a = 3;...' ~~ JavaScript;」这样的东西可能不会那么有作用。

Grammars 就像类一样, 在里面它们实际上是一块可能的代码。 它们本身不会工作, 它们必须从潜在的转换为动态的代码。我们可以像你在类中做的那样:

```perl6
my $JavaScript = JavaScript.new;
```

现在我们拥有了一个可以工作的变量。 现在, 让我们来使用它。所有的 grammar 类都有一个内置的 「parse()」 方法, 以使我们能得到 grammar 中的正则表达式。 我们来修改我们的匹配语句以利用 parse() 方法:

```perl6
$JavaScript.parse('var a = 3; console.log( "Hey, did you know a = " + a + "?" );');
```

我们的代码应该又能工作了。

## 接收动作

现在我们已经把我们所有的匹配的东西打包到一个小型的类里面了, 如果我们能对那些子例程做同样的处理将会很棒。 我们在这儿试试, 把我们的子例程放到它们自己的命名空间中, 就像我们对 rule 做的那样。 我们必须从 「sub」 修改为 「method」, 而我们的 「top」 方法将会使用 「self.」 去调用其它方法。

```perl6
class Actions {
    method assignment-expression( $/ ) {
        'my $' ~ $/<Variable> ~ ' = ' ~ $/<Number> ~ ';'
    }

    method function-call( $/ ) {
        'say ' ~ $/<String>[0] ~ ' ~ $' ~ $/<Variable> ~ ' ~ ' ~ $/<String>[1] ~ ';';
    }

    method top( $/ ) {
        self.assignment-expression( $/<Assignment-Expression> ) ~
        self.function-call( $/<Function-Call> )
    }
}
```

就像之前那样, 我们可以在一行里面创建 Actions 对象:

```perl6
my $actions = Actions.new;
```

并且调用 top 几乎像我们之前做的那样:

```perl6
say $actions.top( $/ );
```

我们已经修改了很多东西了, 所以我们来看看到哪了。

```perl6
grammar JavaScript {
  rule Number                { \d+                                                          };
  rule Variable              { \w+                                                          };
  rule String                { '"' <-[ " ]>+ '"'                                            };
  rule Assignment-Expression { var <Variable> '=' <Number>                                  };
  rule Function-Call         { console '.' log '(' <String> '+' <Variable> '+' <String> ')' };
  rule TOP                   { <Assignment-Expression> ';' <Function-Call> ';'              }
}
my $j = JavaScript.new;

$j.parse('var a = 3; console.log( "Hey, did you know a = " + a + "?" );');

class Actions {
    method assignment-expression( $/ ) {
      'my $' ~ $/<Variable> ~ ' = ' ~ $/<Number> ~ ';'
    }

    method function-call( $/ ) {
      'say ' ~ $/<String>[0] ~ ' ~ $' ~ $/<Variable> ~ ' ~ ' ~ $/<String>[1] ~ ';';
    }

    method top( $/ ) {
      self.assignment-expression( $/<Assignment-Expression> ) ~
      self.function-call( $/<Function-Call> )
    }
}

my $actions = Actions.new;
say $actions.top($/);
```

不用担心, 我们快要到了。既然我们有了一个单独的类来处理 Actions, 我们把方法重命名为 grammar 中所匹配的 rule 的名字, 以使我们不会忘记它们是什么。

```perl6
class Actions {
    method Assignment-Expression( $/ ) {
        'my $' ~ $/<Variable> ~ ' = ' ~ $/<Number> ~ ';'
    }

    method Function-Call( $/ ) {
        'say ' ~ $/<String>[0] ~ ' ~$' ~ $/<Variable> ~ ' ~ ' ~ $/<String>[1] ~ ';';
    }

    method TOP( $/ ) {
        self.Assignment-Expression( $/<Assignment-Expression> ) ~
        self.Function-Call( $/<Function-Call> )

    }
}
```

更进一步, 我们还有最后一点魔法能够利用。 我们将把 $javascript 和 $actions 对象像这样组合在一块。

```perl6
say $javascript.parse('...', :actions($actions) );
```

「:actions(...)」 给 「parse()」方法声明的可选参数。我们正告诉正则表达式引擎, 任何时候, 像 <Function-Call> 或 <TOP> 这样的 rule 匹配时, 我们会在我们的类中让它调用对应的同名方法。

这几乎是按原样工作的, 但是如果你运行修改后的代码,  你会发现解析返回了原来的匹配对象, 带着日语引用标记。所以看起来好像我们又回到了原地。不完全是。


继续, 我们在其中之一的方法中添加一个临时的 "say 'Hello';" 语句, 仅仅是为了确认它们正被调用。这是正则引擎正在工作并且可能正解析它所going over 的一个重要证据。 你甚至可以使用某些我们上面已经学到的技巧然后写上 「say $/<Variable>;」 来查看匹配是否正像你想的那样运行。继续运行并玩玩, 做完的时候再回到这儿。

## 混合信号(Mixed Signals)

正发生的是方法正被调用, 但是它们的输出被丢弃。我们来捕获输出然后使用 grammar 的最后一个特性, 抽象语法树。现在,  这可能会勾起坐在教室里看黑板上画出的盒子和线段的场景, 但是也没有那么糟糕了。我们已经看到了一个, 实际上 **say()** 的输出就是一个 AST。

我们来看下其它语法树, 我们在后台创建的那个。在 "$javascript.parse(...)" 调用的末尾添加上 「.ast」, 这会给我们展示我们自己创建的语法树。

如果你这样做了, 你会看到它打印了(Any), 这通常等价于「匹配失败」, 单是我们从之前的测试中知道匹配没有失败。所以这儿发生了什么? 当我们的方法运行的时候, 它们返回输出, 但是 Perl  6 不知道怎么处理这些输出, 或者说它不知道把输出安装到它所创建的 AST中的哪个位置。

关键是一个叫做 「make」的小东西。在方法的开头, 把这个添加到过去我们放置 「say」的地方。

```perl6
class Actions {
    method Assignment-Expression( $/ ) {
      make 'my $' ~ $/<Variable> ~ ' = ' ~ $/<Number> ~ ';'
    }

    method Function-Call( $/ ) {
      make 'say ' ~ $/<String>[0] ~ ' ~ $' ~ $/<Variable> ~ ' ~ ' ~ $/<String>[1] ~ ';'
    }

    method TOP( $/ ) {
      make $/<Assignment-Expression>.ast ~ $/<Function-Call>.ast
    }
}
```

还有, 因为 Perl 6 为我们调用方法, 我们不需要自己来调用 self.Function-Call(...), 我们需要做的全部工作就是查看 Function-Call(...)返回给我们的语法树。 最终我们做到了。一个完整, 虽然微小的编译器。为了防止你在编辑时迷失, 这儿有一个最终的结果。

```perl6
grammar JavaScript {
  rule Number                { \d+                                                          };
  rule Variable              { \w+                                                          };
  rule String                { '"' <-[ " ]>+ '"'                                            };
  rule Assignment-Expression { var <Variable> '=' <Number>                                  };
  rule Function-Call         { console '.' log '(' <String> '+' <Variable> '+' <String> ')' };
  rule TOP                   { <Assignment-Expression> ';' <Function-Call> ';'              }
}

class Actions {
  method Assignment-Expression( $/ ) {
    make 'my $' ~ $/<Variable> ~ ' = ' ~ $/<Number> ~ ';'
  }

  method Function-Call( $/ ) {
    make 'say ' ~ $/<String>[0] ~ ' ~ $' ~ $/<Variable> ~ ' ~ ' ~ $/<String>[1] ~ ';';
   }

  method TOP( $/ ) {
    make $/<Assignment-Expression>.ast ~ $/<Function-Call>.ast }
  }

my $j = JavaScript.new;
my $a = Actions.new;
say $j.parse(
   'var a = 3; console.log( "Hey, did you know a = " + a + "?" );',
   :actions($a)).ast;
```

## 到哪里去

一个简单但整洁的更改是你可以扩展 Assignment-Expression 来既接收数字又接收字符串。上次我们谈论了 rules 中的轮试,所以这个提示应该足够让你开始了:

```perl6
rule Assignment-Expression { var <Variable> '=' (<Number> | <String>) }
```

你必须修改下 Assignment-Expression 方法以使它起作用。或者你可以狡猾一点然后发现( <Number> | <String> ) 可以转换为它自己的小的普通的 "Term" rule, "rule Term { <Number> | <String> }", 然后添加一个 action "method Term($/) { make $/<Number> or $/<String>}" 而只在 Assignment-Expression 中修改一个东西。

当我尝试把选项聚合到一个模块中时我无意中发现了一个常见问题的简洁解决办法。我想拥有一个子例程作为吞噬具名参数、做一些检查、并且要么返回 True 以满足那个 where 从句, 要么死掉并抛出一个合适的错误信息的 where 从句。 那很简单但是不彻底。那个 where 从句没有提供它所检查的参数的名字, 这需要拥有一个合适的错误信息。我们来简化下代码。

```perl6
sub checker(*%colon-keys){ sub ($value-from-where-clause) { True } }
sub f( *%h where checker(:a, :b, :c) ) {}
```

子例程 checker 在 where 从句真正检测任何东西之前被调用。在 Perl 6 中 where 从句是一种句法。如果提供一个表达式，它会调用该表达式，并保持它的返回值然后针对 %h 的值做实际检查。这并不在编译时发生，且返回值不进行缓存。在我们的例子中返回一个接受一个参数(where需要的)的匿名子例程，并且必须返回 True 以让 where 从句的检查通过。

如你所见 checker 接收一组冒号对(colon-pairs)。剩下的问题我们能提供给我们可能要输出的异常，特别的附加信息。如果我们希望该参数是可选的，避免奇怪的语法。我们可以有一个可选的位置，但我们就不能在checker中混合位置和具名参数。存储该值将是微不足道，因为返回的匿名子例程可以具有闭包变量。我们只需要填充它。幸运的是，子返回作为引用返回，然后由那个 where 从句调用。我们可以偷偷加进另一个呼叫，只要我们大家别忘了把代码引用返回给匿名子例程(sub)。

返回引用给同一个对象是通过链式方法调用完成的。通常情况下，当事情在 Perl 6 的领地上变得复杂时，我们就混入一个 role 好了。然后，我们有了一个可以返回自身的方法。

```perl6
use v6;

sub g($i){
    my $closure-variable;
    sub ($value-from-where-clause) {
        say [$closure-variable, $value-from-where-clause];
        $value-from-where-clause == $i
            or die "bad value $value-from-where-clause for $closure-variable"
    } but role :: {
        method side-channel($second-value){
            $closure-variable = $second-value;
            self
    }
  }
}

sub f($a where g(42).side-channel($a.VAR.name) ){ 'OK' }

say f(42);
# OUTPUT
# [$a 42]
# OK
```

还有，我们有了它 - 一个到 where 从句调用的副通道, 或者任何我们可以用高阶函数的其它点。现在我可以去为选项组提供MTA错误消息了。

[i-left-my-keys-in-a-side-channel](https://gfldex.wordpress.com/2016/09/26/i-left-my-keys-in-a-side-channel/)

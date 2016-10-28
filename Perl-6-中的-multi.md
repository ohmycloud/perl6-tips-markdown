[Perl 6 中的 multi](https://docs.perl6.org/syntax/multi)

## (函数) multi 声明符

Perl 6 允许你使用同一个名字但是不同签名写出几个子例程。当子例程按名字被调用时, 运行时环境决定哪一个子例程是最佳匹配, 然后调用那个候选者。你使用 `multi` 声明符来声明每个候选者。

```perl6
multi congratulate($name) {
    say "祝你生日快乐, $name";
}

multi congratulate($name, $age) {
    say "祝 $age 岁生日快乐, $name";
}

congratulate 'Camelia'; # 祝你生日快乐, Camelia
congratulate 'Rakudo', 15; # 祝你 15 岁生日快乐, Rakudo 
```

分发/分派(dispatch) 可以发生在参数的数量(元数)上, 但是也能发生在类型上:

```perl6
multi as-json(Bool $d) { $d ?? 'true' !! 'false' }
multi as-json(Real $d) { ~$d }
multi as-json(@d)      { sprintf '[%s]', @d.map(&as-json).join(', ') }

say as-json([True, 42]); # [true, 42]
```

不带任何指定例程类型的 `multi` 总是默认为 `sub`, 但是你也可以把 `multi` 用在方法(methods)上。那些候选者全都是对象的 `multi` 方法:

```perl6
class Congrats {
    multi method congratulate($reason, $name) {
        say "Hooray for your $reason, $name";
    }
}

role BirthdayCongrats {
    multi method congratulate('birthday', $name) {
        say "Happy birthday, $name";
    }
    multi method congratulate('birthday', $name, $age) {
        say "Happy {$age}th birthday, $name";
    }
}

my $congrats = Congrats.new does BirthdayCongrats;

$congrats.congratulate('升职', 'Cindy');   #-> 恭喜你升职,Cindy
$congrats.congratulate('birthday', 'Bob'); #-> Happy birthday, Bob 
```

## proto


[proto](https://docs.perl6.org/syntax/proto) 从形式上声明了 `multi` 候选者之间的`共性`。 proto 充当作能检查但不会修改参数的包装器。看看这个基本的例子:



```perl6
proto congratulate(Str $reason, Str $name, |) {*}
multi congratulate($reason, $name) {
   say "Hooray for your $reason, $name";
}
multi congratulate($reason, $name, Int $rank) {
   say "Hooray for your $reason, $name -- you got rank $rank!";
}

congratulate('being a cool number', 'Fred');     # OK
congratulate('being a cool number', 'Fred', 42); # OK
congratulate('being a cool number', 42);         # Proto match error
```

所有的 `multi congratulate` 都会遵守基本的签名, 这个签名中有两个字符串参数, 后面跟着可选的更多的参数。 `|` 是一个未命名的 `Capture` 形参, 它允许 `multi` 接收额外的参数。第三个 congratulate 调用在编译时失败, 因为第一行的 proto 的签名变成了所有三个 multi congratulate 的共同签名, 而 42 不匹配 `Str`。

```perl6
say &congratulate.signature #-> (Str $reason, Str $name, | is raw)
```

你可以给 `proto` 一个函数体, 并且在你想执行 dispatch 的地方放上一个 `{*}`。

```perl6
# attempts to notify someone -- returns False if unsuccessful
proto notify(Str $user,Str $msg) {
   my \hour = DateTime.now.hour;
   if hour > 8 or hour < 22 {
      return {*};
   } else {
      # we can't notify someone when they might be sleeping
      return False;
   }
}
```

`{*}` 总是分派给带有参数的候选者。默认参数和类型强制转换会起作用单不会传递。

```perl6
proto mistake-proto(Str() $str, Int $number = 42) {*}
multi mistake-proto($str,$number) { say $str.WHAT }
mistake-proto(7,42);   #-> (Int) -- coercions not passed on
mistake-proto('test'); #!> fails -- defaults not passed on
```












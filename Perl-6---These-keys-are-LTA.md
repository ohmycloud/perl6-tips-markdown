
在折腾枚举作为子例程布尔选项的时候, 我发现默认的错误信息不够酷。

- `Constraint type check failed for parameter '@options'`

让错误信息变得更具体有点困难。我们来创建几个 exceptions 来告诉我们当东西出错时究竟发生了什么。

```perl6
class X::Paramenter::Exclusive is Exception {
has $.type;
    method message {
        "Parameters of {$.type.perl} are mutual exclusive"
    }
}
```

现在我们能检查 *Find::Type* 的选项是否是独占的从而抛出异常。

```perl6
&& ( exclusive-argument(@options, Find::Type) or fail X::Paramenter::Exclusive.new(type => Find::Type) )

class X::Parameter::UnrecognisedOption is Exception {
    has $.type;
    has $.unrecognised;
    method message {
        "Option { $.unrecognised } not any of { $.type.map({ (.^name ~ '::') xx * Z~ .enums.keys.flat }).flat.join(', ') }"
    }
}
```

因为枚举对类型而言是容器, 我们能使用集合操作符来检查并挑选出不匹配的选项(基本上, +options 吞噬的任何东西我们都不知道)。

```perl6
or fail X::Parameter::UnrecognisedOption.new(type => (Find::Type, Find::Options),
    unrecognised => .item ∖ (|Find::Type::.values, |Find::Options::.values) )
```

把错误信息拼接在一块儿有点复杂因为我们可以在一个给定的枚举中得到所有枚举键的一个列表, 除了那些不知道它们的限定名的。 我们不得不在枚举名的前面手动前置一个 `::`。

```perl6
class X::Parameter::UnrecognisedOption is Exception {
    has $.type;
    has $.unrecognised;
    method message {
        "Option { $.unrecognised } not any of { $.type.map: { (.^name ~ '::') xx * Z~ .enums.keys.flat } }"
    }
}
```

这导致了一个更令人惊叹的错误信息:

```
Option 42 not any of Type::File, Type::Dir, Type::Symlink, Options::Recursive, Options::Keep-going
```

这看起来都很普通。我们拥有一个参数化的吞噬参数, 它带有一个或多个枚举而那些枚举可能拥有一个 flag 告诉它们是否用作单选按钮。听起来这个 idiom 会很适合这个模块。

> 翻译的不好, 最好看原文。我也不知道  LTA 是什么意思

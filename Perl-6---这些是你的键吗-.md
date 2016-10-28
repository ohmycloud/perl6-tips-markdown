
## Are these your keys?

[are these your keys](https://gfldex.wordpress.com/2016/09/21/are-these-your-keys/)

在键是常量的散列中我经常打错键的名字。通过混合一个重载方法 `AT-KEY` 的 role 让键限制在一个给定的字符串列表中并不困难。但是那会是一个运行时错误并且用一个运行时错误代替另一个运行时错误。

枚举确实拥有一组常量值的键并提供散列能使用的同一性。Perl 6 的确允许枚举作为键的约束并且如果我们查询了一个没有定义为枚举的键的话会在编译时抛出异常。 然而, 如果我们把散列限制为一组给定的键, 我们可能想要输出所有可能的键, 而不仅仅是有关联值的那些键。我们可以通过混进一个 role 来解决它。

```perl6
enum Noms(<Greenstuff Walkingstuff Syntetics>);
(my %eaten{Noms} is default(0)) does role :: {
    method keys { Noms::.values }
    method kv { 
        gather for self.keys -> \k { 
            take k, self.{k}
        }
    }
};

%eaten{Greenstuff}++;
dd %eaten;
# Hash[Any,Noms]+{<anon|75781152>} %eaten = (my Any %{Noms} = Noms::Greenstuff => 1)
dd %eaten.keyof;
# Noms
dd %eaten.keys;
# (Noms::Walkingstuff, Noms::Greenstuff, Noms::Syntetics).Seq
dd %eaten.kv;
# ((Noms::Walkingstuff, 0), (Noms::Greenstuff, 1), (Noms::Syntetics, 0)).Seq
```

默认值 0 对于散列计数是有意义的。对于未定义值来说一个 Failure 可能更合适, 假如我们确定了键的话。

当然我们可以把那些方法插入到一个合适的具名 role 中, 并且当我们在它上面时, 也能留意到数组。枚举的默认值类型是从 0 开始的 Int 型, 这符合数组的胃口。

```perl6
role Enumkeys[::T] {
method keys { T::.values }
  multi method kv(Positional:D)  { gather for self.keys -> \k { take k, self.[k] } }
  multi method kv(Associative:D) { gather for self.keys -> \k { take k, self.{k} } }
}
```

如果我们确实使用了一组有限制的键, 那我们应该把数组的尺寸限制为枚举的键的数量。

```perl6
multi sub prefix:<+>(Any:U \E where .HOW ~~ Metamodel::EnumHOW){ E.enums.elems }
```

因为数组不是真正拥有键值, Perl 6 不会有助于使用如果我们使用不带枚举键的数组的话, 但是简单的键入错误会在编译时被捕获。还有, 我们需要做一个运行时混入(mixin), 那是 my 语句周围的圆括号的所来自的地方。

```perl6
(my @eaten[+Noms]) does Enumkeys[Noms];
@eaten[Syntetics]++;
dd @eaten.kv;
# (Noms::Walkingstuff, Any, Noms::Greenstuff, Any, Noms::Syntetics, 1).Seq
```

Perl 6 让使用者和正确的程序离的更近了, 一次一个键。

更新:  commit/fef3655 数组定型直接理解了枚举, 前缀 `prefix:<+>` 因此不再需要了。

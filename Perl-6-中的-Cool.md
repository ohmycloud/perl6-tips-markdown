[class Cool](https://docs.perl6.org/type/Cool) - 可以作为字符串或数字互换的值

```perl6
class Cool is Any { }
```

我们看下官方文档给出的英文解释:

> Value that can be treated as a string or number interchangeably

`Cool`, 也称为 **C**onvenient **OO** **L**oop，是字符串、数字和其他内置类的基类，大多数你可以互换使用。

`Cool` 中的方法强迫调用者到一个更具体的类型，然后调用该类型的相同的方法。 例如，[Int](https://docs.perl6.org/type/Int) 和 [Str](https://docs.perl6.org/type/Str) 都继承自 `Cool`，并且在 `Int` 上调用 `substr` 方法会首先将整数转换为 `Str`。

```perl6
123.substr(1, 1);   # '2', same as 123.Str.substr(1, 1)
```

下面的内置类型继承自 `Cool`:  [Array](https://docs.perl6.org/type/Array)、 [Bool](https://docs.perl6.org/type/Bool)、 [Complex](https://docs.perl6.org/type/Complex)、 [Cool](https://docs.perl6.org/type/Cool)、 [Duration](https://docs.perl6.org/type/Duration)、 [Map](https://docs.perl6.org/type/Map)、 [FatRat](https://docs.perl6.org/type/FatRat)、 [Hash](https://docs.perl6.org/type/Hash)、 [Instant](https://docs.perl6.org/type/Instant)、 [Int](https://docs.perl6.org/type/Int)、 [List](https://docs.perl6.org/type/List)、 [Match](https://docs.perl6.org/type/Match)、 [Nil](https://docs.perl6.org/type/Nil)、 [Num](https://docs.perl6.org/type/Num)、 [Range](https://docs.perl6.org/type/Range)、 [Seq](https://docs.perl6.org/type/Seq)、 [Stash](https://docs.perl6.org/type/Stash)、 [Str](https://docs.perl6.org/type/Str)。

下表总结了 `Cool` 所提供的方法，以及它们所被强制到的类型：

|method	        |coercion type|
|:-------------:|:-----------:|
|abs	        |Numeric|
|conj	        |Numeric|
|sqrt	        |Numeric|
|sign	        |Real   |
|rand	        |Numeric|
|sin	        |Numeric|
|asin	        |Numeric|
|cos	        |Numeric|
|acos	        |Numeric|
|tan	        |Numeric|
|tanh	        |Numeric|
|atan	        |Numeric|
|atan2	        |Numeric|
|atanh	        |Numeric|
|sec	        |Numeric|
|asec	        |Numeric|
|cosec	        |Numeric|
|acosec	        |Numeric|
|cotan	        |Numeric|
|cotanh	        |Numeric|
|acotan	        |Numeric|
|sinh	        |Numeric|
|asinh	        |Numeric|
|cosh	        |Numeric|
|acosh	        |Numeric|
|sech	        |Numeric|
|asech	        |Numeric|
|cosech	        |Numeric|
|acosech	    |Numeric|
|acotanh	    |Numeric|
|cis	        |Numeric|
|log	        |Numeric|
|exp	        |Numeric|
|roots	        |Numeric|
|log10	        |Numeric|
|unpolar	    |Numeric|
|round	        |Numeric|
|floor	        |Numeric|
|ceiling	    |Numeric|
|truncate	    |Numeric|
|chr	        |Int    |
|ord	        |Str    |
|chars	        |Str    |
|fmt	        |Str    |
|uniname	    |Str    |
|uninames	    |Seq    |
|unival	        |Str    |
|univals	    |Str    |
|uniprop	    |Str    |
|uniprop-int	|Str    |
|uniprop-str	|Str    |
|uniprop-bool	|Str    |
|unimatch       |Str    |
|uc	            |Str    |
|lc	            |Str    |
|fc	            |Str    |
|tc	            |Str    |
|tclc	        |Str    |
|flip	        |Str    |
|trans	        |Str    |
|index	        |Str    |
|rindex	        |Str    |
|ords	        |Str    |
|split	        |Str    |
|match	        |Str    |
|comb	        |Str    |
|subst	        |Str    |
|sprintf	    |Str    |
|printf	        |Str    |
|samecase	    |Str    |
|trim	        |Str    |
|trim-leading	|Str    |
|trim-trailing	|Str    |
|EVAL	        |Str    |
|chomp	        |Str    |
|chop	        |Str    |
|codes	        |Str    |

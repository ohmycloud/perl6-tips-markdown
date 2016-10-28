## [with orwith without](https://docs.perl6.org/syntax/with%20orwith%20without)

`with` 语句就像 `if` 但是是为了测试是否定义而非真假。此外, 它主题化了条件, 这很像 `given`:

```perl6
with "abc".index("a") { .say } # print 0
```

代替 `elsif`, `orwith` 用于把是否定义的测试链接起来:

```perl6
# The below code says "Found a at 0"
my $s = "abc";
with   $s.index("a") { say "Found a at $_" }
orwith $s.index("b") { say "Found b at $_" }
orwith $s.index("c") { say "Found c at $_" }
else                 { say "Didn't find a, b or c" }
```

你可以混合基于 `if` 的从句和基于 `with` 的从句：

```perl6
# This says "Yes"
if 0 { say "No" } orwith Nil { say "No" } orwith 0 { say "Yes" };
```

至于 `unless`, 你可以使用 `without` 来检测未定义(undefinedness), 但是你不可以添加 `else` 从句:

```perl6
my $answer = Any;
without $answer { warn "Got: $_" }
```

我们还有 `with` 和 `without` 语句修饰符：

```perl6
my $answer = (Any, True).roll;
say 42 with $answer;
warn "undefined answer" without $answer;
```

测试下：

```
> my $answer = (Any, True).roll;
True
> say 42 with $answer;
42
> my $answer = (Any, True).roll;
(Any)
> say 42 with $answer;
()
```



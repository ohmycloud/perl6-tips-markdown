### 找出两个文件中共有的行, 顺序无关紧要
---

在 Perl 5 里, 你可以这样:

``` perl
#!/usr/bin/env perl
use 5.010; use warnings; use strict;

my %filea = map { $_ => 1 } do { open my $fa, '<', 'filea' or die $!; <$fa> };
my %fileb = map { $_ => 1 } do { open my $fb, '<', 'fileb' or die $!; <$fb> };
for( keys %filea ){
    print if $fileb{$_};
}
```

在 Perl 6 中就长这样:

``` perl6
#!/usr/bin/env perl6
use v6;

my @a := "filea".IO.lines;
my @b := "fileb".IO.lines;
.say for keys( @a ∩ @b );
```

因为  Perl 6 中的"惰性列表”, 底层实现能把工作分割成不同的任务并行执行, 然后在需要结果的时候返回给它们. 所以, 这种情况下, `@a` 和  `@b` 的填充可以同时运行.但是要点是, 如果你有耗费时间,不彼此依赖的操作, 或者函数A要传递一个 item列表给函数B, 这些操作可能并行执行, 提高速度, 你不需要做任何线程相关的东西. 非常赞!

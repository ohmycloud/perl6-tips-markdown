> This is Rakudo version 2016.07.1 built on MoarVM version 2016.07
implementing Perl 6.c.

现在的 Perl 6 还是比较慢的。现在我们打印从 1 开始计数的第 10000 个素数:

## 并发版本

```perl6
sub find-prime($count) {
    my $channel = Channel.new;
    
    my $promise = start {
        for ^$count {
            $channel.send($_) if .is-prime;
        }
        
        LEAVE $channel.close unless $channel.closed;
    }
    
    return $channel.list but role :: { method channel { $channel } };;
}

my @primes = find-prime(110000);

for @primes {
    @primes.channel.close if $++ > 5000; # hard-close the channel after 5000 found files
    .say if $++ %% 100 # print every 100th file
}

say @primes[10000];
say now - INIT now;
```

上面的并发查找素数, 用时 **35** 秒多! [but](https://raw.githubusercontent.com/perl6/specs/master/S14-roles-and-parametric-types.pod) 的作用类似于 `does`。

## gather/take 版

```perl6
my @vals = lazy gather {
    for ^Inf {
        take $_ if .is-prime;
    }
}

say @vals[10000];
say now - INIT now;
```

`gather/take` 类似于 Python 中的 `yield` , 用时 25 秒多。

## 普通版本

如果不使用 `is-prime` 函数, 查找第 10000 个素数就比较慢: 

```perl6
# 下面这几种写法, 性能很差

 lazy gather {
     CANDIDATE: for  2 .. 110000 -> $candidate {
    	 for  2 .. sqrt $candidate  -> $divisor {
    		next CANDIDATE if $candidate % $divisor == 0;
    	 }
    	take $candidate;
     }
 } ==> my @vals;

say @vals[10000];
say now - INIT now;
```

```perl6
lazy gather {
    for ^Inf {
      take $_ if .is-prime;
    }
} ==> my @vals;

say @vals[10000];
say now - INIT now; 
```

```perl6
 my @vals = lazy gather {
    for 1..110000 {
      take $_ if (2 ** ($_ - 1) % $_ == 1 || $_ == 2);
     }
 }

 say @vals[10000];
 say now - INIT now; 
```


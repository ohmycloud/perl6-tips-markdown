# PERL 6: 编程未来长什么样?

作者: [Zoffix Znet](http://tpm2016-2.zoffix.com/#/) 

# HASH SOME PASSWORDS

```perl6
use Crypt::Bcrypt;
my @passes = rand.Str xx 32;

my @hashes;
for @passes {
    @hashes.push: bcrypt-hash $^pass, :15rounds;
}
say @hashes;
```

> 运行花费了 63 秒

# 要使我们的程序多线程化需要多少行额外的代码?

地球上还没有任何语言能比 Perl 6 做得更好!

我们不需要额外的代码行, 只要两个单词就够了!

# 多线程

```perl6
use Crypt::Bcrypt;
my @passes = rand.Str xx 32;

my @hashes;
for @passes {
    @hashes.push: start bcrypt-hash $^pass, :15rounds;
}
say await @hashes;
```

在代码中添加两个额外的单词就比原来快了 2,000%。 

但是等等, 我还有话说!

## 单线程

```perl6
use Crypt::Bcrypt;
my @passes = rand.Str xx 32;

my @hashes = @passes.map: {bcrypt-hash $^pass, :15rounds};
say @hashes;
```

## 多线程

```perl6
use Crypt::Bcrypt;
my @passes = rand.Str xx 32;

my @hashes = @passes.race.map: {bcrypt-hash $^pass, :15rounds};
say @hashes;
```

# 但是再等等, 我还要再唠两句!

这是 HYPER 运算符: `»`

```perl6
@bunch-of-things».call-this-method-on-every-item;
```

## 短点!

```perl6
use Crypt::Bcrypt;
my @passes = rand.Str xx 32;
say @passes».&bcrypt-hash: :15rounds;
```

## 再短点!

```perl6
use Crypt::Bcrypt;
say (rand.Str xx 32)».&bcrypt-hash: :15rounds;
```

# 但是我们所有的核心怎样了呢?

- 2016: 单线程化
- 2018: 自动线程化

让编译器来解决这个东西。

有很多很酷的东西关于：
并发/PARALLELIZM/异步/PROMISES/SUPPLIES/CHANNELS/JUNCTIONS/FEEDS
它门都是语言自身的一部分(而非模块)！

编程未来是什么样的呢? 更多核心。更多的 [Perl 6](http://perl6.org).

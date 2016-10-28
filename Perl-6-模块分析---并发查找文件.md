第一次分析 Perl 6 中的模块, 学到很多东西。[perl6-concurrent-file-find](https://github.com/gfldex/perl6-concurrent-file-find) 是使用 Perl 6 的并发特性进行文件的快速查找, 里面使用了 `Block`、`Junction`、`»` 运算符、`递归`、`异常处理`、`智能匹配` 等特性。由于知识、智力有限, 可能会有错误和纰漏, 先写篇笔记, 日后再来修缮。


```perl6
class X::IO::NotADirectory does X::IO is export {
    has $.path;
    method message {
        "«$.path» is not a directory"
    }
}

class X::IO::CanNotAccess does X::IO is export {
    has $.path;
    method message {
        "Cannot access «$.path»: permission denied"
    }
}

class X::IO::StaleSymlink does X::IO is export {
    has $.path;
    method message {
        "Stale symlink «$.path»"
    }
}
```

上面的代码自定义了 3 个与 `IO` 错误相关的类并导出, 每个类中有一个 `$.path` 属性和 `message` 方法。如果在查找文件/目录时出现异常则会打印出相关的信息, 每个信息中会包含该文件/目录的绝对路径。


![X::IO](http://upload-images.jianshu.io/upload_images/326727-e9fb734d50339858.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


自定义的类都遵守了 [X::IO](https://docs.perl6.org/type/X$COLON$COLONIO) 这个角色(Role), 角色 `X::IO` 是一个用于 IO 错误相关的通用角色, `X::IO` 没有提供任何额外的方法, 它遵守 [X::OS](https://docs.perl6.org/type/X$COLON$COLONOS) 角色,

```perl6
role X::IO does X::OS {}
```

而 X::OS 角色的定义为:

```perl6
role X::OS { has $.os-error }
```

它是由操作系统报告的一些错误触发的所有异常的通用角色（失败的 IO，系统调用，fork，内存分配）。


定义一个参数互斥类并导出。如果传递给 `find` 方法的参数之间有相互排斥, 则抛出这个异常。

```perl6
class X::Paramenter::Exclusive is Exception is export {
    has $.type;
    method message {
        "Parameters {$.type} are mutual exclusive"
    }
}
```

下面来看 `find` 方法的签名。第一个参数 `$dir` 是需要用户指定的文件查找的**起始路径**, `$dir` 是一个 **Str** 类型的参数, 在 `find` 方法中被强制为 IO 类型。`$dir` 前面的 `IO(Str)`, 表示它既可以是一个字符串, 也可以是一个 `IO` 路径, 如果传入的是字符串, 那么在该 find 方法中就会被强制为 `IO` 类型。`$dir` 后面的 `where` 从句(从句的内容是一个 Block, 直接执行)是对该参数的约束：该路径必须真实存在(否则抛出异常)、必须是一个目录(否则抛出异常)、路径必须可访问(否则抛出异常)。

where 从句的 `Block` 中使用了`点语法`, 即调用了 IO 方法, 调用者是 `$_`, 这里就是 `$dir` 了。下面还会见到这种 `Block` 和点语法的用法。

```perl6
sub find ( 
    IO(Str) $dir where { 
           ( .IO.e || fail X::IO::DoesNotExist.new(path => .Str ) ) 
        && ( .IO.d || fail X::IO::NotADirectory.new(path => .Str) )
        && ( .IO.r || fail X::IO::CanNotAccess.new(path => .Str ) )
    },
    :$name, :$exclude, :$exclude-dir, :$include, :$include-dir, :$extension,
    :&return-type = { .IO.Str },
    :$no-thread = False,
    :$file = True, :$directory, :$symlink,
    :$max-depth where { $^a ~~ Int || $^a ~~ ∞ && $^a > 0 } = ∞,
    :$recursive = True, :$follow-symlink = False,
    :$keep-going = True, :$quiet = False
) is export { ... }
```

`find` 方法的其余参数都是一个一个的 `Pair` 了, 它们就像袜子一样, 都是一对儿一对儿的:

|副词                      |等价于                         |
|-------------------------:|:-----------------------------:|
|:$name                    |name           => $name        |
|:$exclude                 |exclude        => $exclude     |
|:$exclude-dir             |exclude-dir    => $exclude-dir |
|:$include                 |include        => $include     |
|:$include-dir             |include-dir    => $include-dir |
|:$extension               |extension      => $extension   |
|:&return-type             |return-type    => &return-type |
|:$no-thread = False       |no-thread      => False        |
|:$file = True             |file           => True         |
|:$directory               |directory      => $directory   |
|:$symlink                 |symlink        => $symlink     |
|:$max-depth               |max-depth      => $max-depth   |
|:$recursive = True        |recursive      => True         |
|:$follow-symlink = False  |follow-symlink => False        |
|:$keep-going = True       |keep-going     => True         |
|:$quiet = Flase           |quiet          => False        |

`:$no-thread = False` 表示默认开启多线程, `:$file = True` 表示默认查找文件, `:$recursive = True` 表示默认开启递归查找, `:$keep-going = True` 表示遇到异常也会继续查找而不退出, `:$quiet = Flase` 表示默认静默查找。

有一个参数很奇怪, 它后面还有一个 where 从句, 用以约束最大深度, `$max-depth` 必须为整型或者它的值无穷大并且为正值

```perl6
:$max-depth where { $^a ~~ Int || $^a ~~ ∞ && $^a > 0 } = ∞
```

下面是 `find` 方法的函数主体: 

`$*SPEC.dir-sep` 为路径分割符, Windows 下为 ＼, Linux 下为 `/`。使用 `constant` 声明了一个常量来保存路径分割符。
`&max-depth` 是一个可调用的 `Block`：

```perl6
constant dir-sep = $*SPEC.dir-sep; 
my &max-depth = $max-depth < ∞ ?? { .IO.path.split(dir-sep).elems <= $max-depth } !! { True }; 
```

```perl6
my @tests; # 保存文件查找所用的条件测试
my @types; # 
```
    
下面的代码再次用到了 `Block` 和主题化变量 `$_`。

```perl6    
@types.append({.f}) if $file; 
@types.append({.d}) if $directory;
@types.append({.l}) if $symlink;

@tests.append(@types.any); 
@tests.append({.basename.Str ~~ $name}) with $name; 
```

我们往 `@types` 数组里面追加了一个 `Block`, 这个数组中的元素不是普通值, 它是一个带有占位符参数(`$_`)的花括号块儿。 `{.f}` 等价于 `{$_.f}`。那么这里的 `$_` 代表着什么呢? 从这句代码所在的上下文可以知道, find 方法的函数主体中并没有显式的声明 `$_` 主题变量, 也没有 `for`、 `given`、`where` 等能主题化的关键字, 所以, `{.f}`、`{.d}`、`{.l}` 中的 `$_` 来自别的地方。怎么知道它来自哪个地方呢？ 我们知道只有当有文件、有目录、有 sysmlink  的时候才把相关的 Block 追加到 `@types` 数组中。既然数组中每个元素都是 `Block`, 那么对每个 Block 可以进行调用。我们现在先知道这些。

`@types.any` 是一个 any `Junction`, 意思为查找时遇到文件、目录或 symlink 中的任意一种类型都可以保留下来。
`with` 修饰符的意思是如果定义了具体的文件名, 则在 `@tests` 数组中追加一个 `Block`, 这个 `Block` 检查找到的文件名和要查找的文件名是否匹配, 如果匹配的话这个 Block 的值就为 True, 否则为 Flase。


下面的代码是查找文件时用到的一些测试, `exclude-tests` 是要排除在外的文件名：

```perl6    
my @exclude-tests;
for $exclude.list -> $exclude {
    @exclude-tests.push({ .Str ~~ $exclude })        if $exclude ~~ Regex;
    @exclude-tests.push({ $exclude.(.IO) })          if $exclude ~~ Callable ^ Regex;
    @exclude-tests.push({ .Str.contains($exclude) }) if $exclude ~~ Str;
}
@tests.append(@exclude-tests.none);
```

`$exclude` 来自于传递进来的 Pair 参数, 它可以包含单个值, 也可以包含多个值, 所以使用 `.list` 用来遍历所有的可能。`$exclude` 可以是正则表达式、`Callable` 和普通字符串。


与上面的类似, 下面是要包含在内的文件名。 `@include-tests` 数组中保存的是要包含在查询结果中的文件, 数组中每一个元素都是一个 `Block`
每个 `Block` 接收一个参数作为 `$_` 的值, 每个 `.` 号前面是有一个参数的, 这个会在调用 `Block` 时传入：

```perl6
my @include-tests;
for $include.list -> $include {
    
    @include-tests.push({ .Str ~~ $include })        if $include ~~ Regex;
    @include-tests.push({ $include.(.IO) })          if $include ~~ Callable ^ Regex;
    @include-tests.push({ .Str.contains($include) }) if $include ~~ Str;
}
@tests.append(@include-tests.any) if @include-tests;
```

文件名后缀与之类似, 可以是正则表达式、`Callable ^ Regex` 和普通字符串：

```perl6
my @extension-tests;
for $extension.list -> $test {
    @extension-tests.push({ .extension ~~ $test if .extension }) if $test ~~ Regex;
    @extension-tests.push({ $test.(.extension) })  if $test ~~ Callable ^ Regex;
    @extension-tests.push({ $test eq .extension }) if $test ~~ Str;
}
@tests.append(@extension-tests.any) if @extension-tests;
```

目录测试, 如果 `$follow-symlink` 为真则在该路径是目录 && sysmlink && （路径不存在就直接抛出 fail 异常）, 然后返回这个目录。

```perl6
my @dir-tests = $follow-symlink
    ?? { .d && .l && ( !.e && fail X::IO::StaleSymlink.new(:path(.Str)) ); .d }
    !! { .d && ! .l };
```

排除目录的测试:

```perl6
my @exclude-dir-tests;
for $exclude-dir.list -> $exclude {
    @exclude-dir-tests.push({ .Str ~~ $exclude })        if $exclude ~~ Regex;
    @exclude-dir-tests.push({ $exclude.(.IO) })          if $exclude ~~ Callable & !Regex;
    @exclude-dir-tests.push({ .Str.contains($exclude) }) if $exclude ~~ Str;
}
@dir-tests.append(@exclude-dir-tests.none);
```

包含目录的测试:

```perl6
my @include-dir-tests;
for $include-dir.list -> $include {
    @include-dir-tests.push({ .Str ~~ $include })        if $include ~~ Regex;
    @include-dir-tests.push({ $include.(.IO) })          if $include ~~ Callable & !Regex;
    @include-dir-tests.push({ .Str.contains($include) }) if $include ~~ Str;
}
@dir-tests.append(@include-dir-tests.any) if @include-dir-tests;
```
    
下面是并发查找部分：
    
```perl6    
my $channel = Channel.new;
my &start = -> ( &c ) { c } if $no-thread;
my $promise = start { ... }
```

在 `for` 这个块中, `$_` 代表当前找到的文件/文件夹。如果有错误发生(例如禁止访问)并且非安静模式下会打印出有异常的文件/目录的路径并继续，否则重新抛出[rethrow](https://docs.perl6.org/routine/rethrow)这个异常：

```perl6
for dir($dir) {
    CATCH { default { if $keep-going { warn .Str unless $quiet } else { .rethrow } } }
    
    # 如果这个文件/目录是link的并且不存在,就抛出异常
    if .IO.l && !.IO.e {
        X::IO::StaleSymlink.new(path=>.Str).throw;
    }
...
```

调用一个关闭了的 channel 会导致 [X::Channel::SendOnClosed](https://docs.perl6.org/type/X$COLON$COLONChannel$COLON$COLONSendOnClosed).
`last` 不仅会退出当前块, 还会退出当前 for 循环。

```perl6
CATCH { when X::Channel::SendOnClosed { last } }
```

`@tests».(.IO)` 意为对 `@tests` 中的每一个 `Block` 元素都执行一次 `.(.IO)` 调用，参数为 `.IO`， 即 `$_.IO`, 语法为 `$someBlock.(paramter)`。
`all` 代表所有测试条件都通过, 即每一个 `Block` **调用**都返回真:

```perl6
$channel.send(.&return-type) if all @tests».(.IO);
```

`&return-type` 是一个 `Block`, 所以可以用点语法调用, 这个 `Block` 为 `{ .IO.Str }`, 所以调用后得到的是路径的字符串表示。
        
如果所有的目录测试条件通过并且最大深度为真并且递归查找开启, 则对该目录中的每一个元素（文件/目录）
按照存在和文件名排序后再对每一项执行一次当前块(`&?BLOCK`)

```perl6
.IO.dir().sort({.e && .f}).map(&?BLOCK) if $recursive && .&max-depth && all @dir-tests».(.IO)
```

离开的时候关闭 Channel

```perl6
LEAVE $channel.close;
```

```perl6
return $channel.list but role :: { method channel { $channel } };
```


该模块还提供了一个简单的查找方法:

```perl6
sub find-simple ( IO(Str) $dir,
    :$keep-going = True,
    :$no-thread = False
) is export {
    my $channel = Channel.new;

    my &start = -> ( &c ) { c } if $no-thread;

    my $promise = start { 
        for dir($dir) {
            CATCH { default { if $keep-going { note .Str } else { .rethrow } } }
            
            if .IO.l && !.IO.e {
                X::IO::StaleSymlink.new(path=>.Str).throw;
            }
            {
                CATCH { when X::Channel::SendOnClosed { last } }
                $channel.send(.IO) if .IO.f; # 这里的 $_ 代表当前查找到的文件
                $channel.send(.IO) if .IO.d; # 这里的 $_ 代表当前查找到的目录
            }
            # 如果目录存在, 默认对该目录执行递归查找
            .IO.dir()».&?BLOCK if .IO.e && .IO.d;
        }
        LEAVE $channel.close unless $channel.closed;
    }

    return $channel.list but role :: { method channel { $channel } };
}
```

`but` 就像 `does` 那样,  但是创建了对象的一份拷贝, 然后把角色混进那个对象中，保持原对象不变。

```perl6
but  role :: { method channel { $channel }  }
```

创建了一个匿名的角色(role),  把名为  `channel` 的方法混进拷贝后的 `$channel.list` 对象中。
---

### Readme 文件


# perl6-concurrent-file-find

concurrent File::Find for Perl 6

# 概要

```perl6
use v6;
use Concurrent::File::Find;

find(%*ENV<HOME>
    , :extension('txt', {.contains('~')}) # ends in .txt or ends in something that contains a ~
    , :exclude('covers') # exclude any path that contains covers, both for files and directories
    , :exclude-dir('.') # exclude any directory-path that contains a . 
    , :file # return file paths
    , :!directory # don't return directory paths
    , :symlink # return symlink paths
    , :max-depth(5) # but not deeper then 5 directories deep
    , :follow-symlink # follow symlinks (no loop detection yet)
    , :keep-going # on error (no access, stale symlink, etc.), keep going
    , :quiet # don't report errors on STDERR
).elems.say; # count how many files and symlinks we got

sleep 10;

# find-simple 函数返回的是一个 `List`, 这个列表混合了一个匿名角色,
# @l 数组因此拥有了一个名为 `channel` 的方法。
my @l := find-simple(%*ENV<HOME>, :keep-going, :!no-thread); # binding to avoid eagerness

for @l {
    # @l 的 channel 方法是一个 `$channel` 变量, 它可以被关闭。
    @l.channel.close if $++ > 5000; # hard-close the channel after 5000 found files
    .say if $++ %% 100 # print every 100th file
}
```

# 描述

## 例程

### sub find

将由后台线程获取到的文件、目录和符号链接作为 `Str` 的`列表`返回。 该列表得到了一个混合到唯一方 `channel` 中的角色，它能可用于关闭 `List` 后面的 channel，以中止任何仍在进行的获取。 这是有点靠不住，当底层的 `Promise`被 `DESTROY` 时可能会产生警告。 有如下所述的各种包括和排他的过滤器选项。 对于任何目录, 在返回的列表中文件被排在目录之前。只有在返回项目后，才可能递归到子目录中。


#### Matcher

一些参数采用匹配器或一组匹配器。 给定列表时使用的“Junction”类型取决于参数。 因为匹配器 `Str`，`Regex` 和 `Callable` 被接受。 除非另有说明，否则 `Str` 匹配文件名的某一部分并且区分大小写或匹配整个路径。 `Regexp` 智能匹配 `IO::Path.Str` 并且 `Callable` 用 `IO::Path` 调用。

#### 参数

`IO(Str) $dir` - 从哪个目录开始, 要么是 `IO::Path`, 要么是 `Str`.

`:$file = True` - 也返回文件

`:$directory` - 也返回目录

`:$symlink` - 也返回符号链接

`:&return-type = { .IO.Str }` - 默认把匹配到的项目转换为 `Str`。该 block 被馈以 `IO::Path` 对象.结果原样返回, 而不是由 `find` 本身使用，你可以在这里疯狂。

`:$name` - 返回匹配所提供的匹配器的任意目录的任意文件。使用 `Str` 作为匹配器需要确切的, 区分大小写的匹配。

`:$include` - 返回任何 `IO::Path.basename` 匹配所提供的匹配器的文件。使用 `Str` 作为匹配器需要部分匹配。 

`:$exclude` - 不返回任何匹配了所提供的匹配器的文件。使用 `Str` 作为匹配器需要部分匹配。

`:$include-dir` - 返回或下降到匹配了所提供的匹配器的目录。使用 `Str` 作为匹配器需要部分匹配。

`:$exclude-dir` - 不返回或下降到匹配了所提供的匹配器的目录。使用 `Str` 作为匹配器需要部分匹配。.

`:$extension` - 返回任何匹配 `IO::Path.extension` 的项目.使用 `Str` 作为匹配器需要确切的, 区分大小写的匹配。

`:$recursive = True` - 下降到子目录。

`Int :$max-depth = ∞` - 尽可能深地下降到子目录中。

`:$follow-symlink = False` - 跟随符号链接. 还没有循环检测.

`:$keep-going = True` - 发生错误时 (拒绝访问, 陈旧的符号链接, 等等.) 继续但是在标准错误 $*ERR 上输出警告。

`:$quiet = False` - 配合 $keep-going, 不输出警告.

`:$no-thread = False` - 禁止创建 `Promise`. 用于调试.

### sub find-simple

和 `find` 一样, 但是没有了过滤器选项, 并且永远是递归的, 跟随现有的符号链接(尚没有循环检测)也没有排序。速度快可能包含少量 bug。它可能抛出 `X::IO::StaleSymlink`.

#### 参数

`IO(Str) $dir` - `Path` as `IO::Path` or Str at where to start looking for files

`:$keep-going = True` - 出现错误不停止

`:$no-thread = False` - 不创建 `Promise`, 调试时有用

## 异常

### `X::IO::NotADirectory does X::IO`

尝试获取不是目录的路径的内容

### `X::IO::CanNotAccess does X::IO`

访问文件夹(目录)被操作系统拒绝。

### `X::IO::StaleSymlink does X::IO`

我们的意图是返回或跟随一个确实存在，但没有目标的符号链接，

### `X::Paramenter::Exclusive is Exception`

命名参数一起使用是互斥的。

# 警告

尚不支持循环检测。 只要有 readlink 和/或 stat 的便携版本，就会添加循环检测。 到那时避免 `:follow-symlink` 或使用 `:max-depth`。

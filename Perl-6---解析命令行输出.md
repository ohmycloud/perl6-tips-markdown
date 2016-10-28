### 哦, 列, 你在哪里?

当 [ramiroencinas](https://github.com/ramiroencinas) 给 Perl 6 的生态系统增加 [FileSystem::Capacity::VolumesInfo](https://github.com/ramiroencinas/perl6-FileSystem-Capacity)模块的时候, 我说他没有添加 macOS 支持。而当我尝试为这个模块贡献源代码时我才发现知道一丢丢 Perl 6 的特性就能节省很多时间。 `FileSystem::Capacity::VolumesInfo` 这个模块所做的就是解析 `df` 命令的输出, 它看起来长这样:

```
$ df -k -P
Filesystem                                  1024-blocks       Used Available Capacity  Mounted on
/dev/disk3                                   1219749248  341555644 877937604    29%    /
devfs                                               343        343         0   100%    /dev
/dev/disk1s4                                  133638140  101950628  31687512    77%    /Volumes/Untitled
map -hosts                                            0          0         0   100%    /net
map auto_home                                         0          0         0   100%    /home
map -fstab                                            0          0         0   100%    /Network/Servers
//Pawel%20Pabian@biala-skrzynka.local./Data  1951417392 1837064992 114352400    95%    /Volumes/Data
/dev/disk5s2                                 1951081480 1836761848 114319632    95%    /Volumes/Time Machine Backups
bbkr@localhost:/Users/bbkr/foo 123           1219749248  341555644 877937604    29%    /Volumes/osxfuse
```

(如果你看到了折行的或者截断的输出请在这儿核对一下原始数据)

虽然这对人类来说看起来不错, 但是对于解析器来说十一个棘手的任务。

- 列的宽度是动态的 - 所以每列的值不能使用带有硬编码位置的子字符串来提取。
- 列和列之间是通过空白分割的, 空白被填充到列与列之间, 并且它们的值也能包含空白 - 所以不能通过空白使用 split 函数来提取值。
- 文件系统的名字拥有不同的转义模式。
- 有些列是左对齐的, 有些列是右对齐的, 有一列是居中对齐的。

所以让我们来使用 Perl 6 中的特性来处理这混杂的东西。

### 捕获命令行输出

```perl6
my ($header, @volumes) = run('df', '-k', '-P', :out).out.lines;
```

方法 `run` 执行 shell 命令并返回 [Proc](https://docs.perl6.org/type/Proc)对象。方法 `out` 创建一个 [管道](https://docs.perl6.org/type/IO$COLON$COLONPipe)对象以接收 shell 命令的输出。方法 `lines` 把该输出按行分割, 第一行保存到 `$header` 变量中, 剩下的行保存到 `@volumes` 数组中。

### 解析 header

```perl6
my $parsed_header = $header ~~ /^
    ('Filesystem')
    \s+
    ('1024-blocks')
    \s+
    ('Used')
    \s+
    ('Available')
    \s+
    ('Capacity')
    \s+
    ('Mounted on')
$/;
```

我们这样做是因为匹配对象保存了每个捕获, 并且每个捕获都知道它所匹配的开始位置和结束位置, 举个例子:

```perl6
say $parsed_header[1].Str;
say $parsed_header[1].from;
say $parsed_header[1].to;
```

会返回:

```
1024-blocks
44
55
```

那会在动态列宽问题上帮助我们很多!


### 提取每行的值

首先我们必须要查看 FileSystem 和 1024-blocks 这两列之间的边界。因为 FileSystem 是左对齐的而 1024-blocks 是右对齐的, 所以两列中的数据都占据那些 headers 之间的空白, 举个例子:

```
Filesystem                      1024-blocks
/dev/someverybigdisk        111111111111111
me@host:/some directory 123    222222222222
         |                      |
         |<----- possible ----->|
         |<--- border space --->|
```

我们不能简单地按空白分割。但是我们知道 1024-blocks 这一列在哪里结束, 所以结束在和 1024-blocks 同一位置的那个数字就是我们的容量大小(volume size)。要提取它, 我们可以使用另外一个有用的 Perl 6 特性 - 正则表达式位置锚点([regexp position anchor](https://design.perl6.org/S05.html#Positional_matching%2C_fixed_width_types))。

```perl6
for @volumes -> $volume {
    $volume ~~ / (\d+) <.at($parsed_header[1].to)> /;
    say 'Volume size is ' ~ $/[0] ~ 'KB';
}
```

它查找对齐于 header 末端位置的数字序列。每个其它的列都能使用这个花招来提取, 如果我们知道那个数据准线的话。

```perl6
$volume ~~ /
    # first column is not used by module, skip it
    \s+

    # 1024-blocks, 右对齐
    (\d+) <.at($parsed_header[1].to)>

    \s+

    # Used, 右对齐
    (\d+) <.at($parsed_header[2].to)>

    \s+

    # Available, 右对齐
    (\d+) <.at($parsed_header[3].to)>

    \s+

    # Capacity, 居中对齐, 不会比 header 还长
    <.at($parsed_header[4].from)>
        \s* (\d+ '%') \s*
    <.at($parsed_header[4].to)> 

    \s+

    # Mounted on, 左对齐
    <.at($parsed_header[5].from)>(.*)
$/;
```

### 益处!

通过在正则表达式中使用 header 名字的位置和位置锚点我们在 macOS 上得到了防炸的 df 解析器, 它能工作在普通的磁盘, 随身存储器, NFS / AFS / FUSE 共享, 古怪的目录名和不同的转义模式中。

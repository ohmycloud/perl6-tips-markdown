## 散列也是容器

假设我们想计算某个东西的出现次数, 我们通常的做法是弄一个 "seen-hash" 散列。有时候我们有一组待查询的键, 其中有些键可能不在我们所扫描的数据中。那是一种特殊情况, 但是 Perl 6 能够完美地解决, 因为散列也是容器, 因此我们能够拥有默认值。

```perl6
my $words = <Hashes are containers too>.lc;
constant alphabet = 'a' .. 'z';

my %seen of Int is default(0);

%seen{$_}++ for $words.comb;
put "$_: %seen{$_}" for alphabet;
```

输出结果:
```
a: 3
b: 0
c: 1
d: 0
e: 3
f: 0
g: 0
h: 2
i: 1
j: 0
k: 0
l: 0
m: 0
n: 2
o: 3
p: 0
q: 0
r: 2
s: 3
t: 2
u: 0
v: 0
w: 0
x: 0
y: 0
z: 0
```
`$words` 中没有出现的特殊字符由 `is default(0)` 处理了。
默认值可以被精心设计。我们来弄一个在数值上下文中为默认值为 0 但是在字符串上下文中为默认值为 NULL 并且总是被定义的一个散列。

```perl6
my %seen of Int is default(0 but role :: { method Str() {'NULL'} });
say %seen<not-there>, %seen<not-there>.defined, Int.new(%seen<not-there>);

# OUTPUT
# «NULLTrue0␤»
```

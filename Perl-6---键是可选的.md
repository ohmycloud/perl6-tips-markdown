## Keys are optional

[Keys are optional](https://gfldex.wordpress.com/2016/09/22/keys-are-optional/)

在我探索一个并发的 File::Find 模块时我发现我需要一组排序用的参数(例如 Bool)和互相排斥的参数。枚举相当易于组合, 把好用的名字引入到作用域中(直接地或通过 export)并且应该让相互排斥变得容易。容易的那部分有点幼稚因为类型化的吞噬参数尚未支持(还没有)。如果没有容易的方式, 肯定会有一个可行的困难的方式。


首先, 我们定义两个枚举用作要查找的选项。

```perl6
package Find {
  enum Type (<File Dir Symlink>);
  enum Options (<Recursive Keep-going>);
}
```

现在我们可以让一个 where 从句首先检查吞噬数组的所有成员要么是 Find::Type 类型要么是 Find::Options 类型。然后我们可以检查 Find::Options 拥有多少个元素。因为如果有太多的话只有一个我们可以抱怨独占。

```perl6
+@options where {
  @options.all (elem) (Find::Type::.values (|) Find::Options::.values)
  && (+(@options.grep: * ~~ Find::Type) <= 1 or die "can only check for one type at a time")
}
```

在主体中我们可以使用 junctions 和智能匹配来检查选项是否出现。

```perl6
my Bool $recursive = any(@options) ~~ Find::Recursive;
my %tests = Find::File => {so .f}, Find::Dir => {so .d}, Find::Symlink => {so .l};
@tests.append(%tests{@options.grep: * ~~ Find::Type});
```

然后在它的参数列表的末尾那个子例程使用一组 flags 来调用。

```
find(%*ENV<HOME>, include => {.extension eq 'txt'}, exclude => ['cfg', /.xml $/] , Find::File, Find::Recursive, Find::Keep-going);
```

同样在具名参数上那也可以有同样的可能, 但是在一个 where 从句中我找不到方法来处理独占。我喜欢在签名中处理尽可能多的参数因为它让写文档变得更加容易了。使用换行符把所有的参数分开然后把类型约束和 where 从句转换成普通的英语。还有, 让枚举作为 flags 感觉很 Perl 6 而那正是这篇博客所谈论的。



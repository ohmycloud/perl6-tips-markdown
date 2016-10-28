[Sequencer Precedence](https://docs.perl6.org/language/operators#Sequencer_Precedence)

### infix ==>

这个流操作符(feed operator)从它的左侧接收结果并把结果作为最后一个参数传递给下一个(右侧的)例程(routine)。

这个操作符的优先级很松散所以你需要使用圆括号把结果赋值给其它变量, 或者你甚至可以使用另外一个流操作符! 在接收单个参数或第一个参数为 block 的程序/方法的例子中, 你必须经常使用圆括号来调用(尽管这对于最后一个例程/方法不是必须的)。

```perl6
# Traditional structure, read bottom-to-top
my @result =
    sort               # (4) Sort, result is <Earth People>
    grep { /<[PE]>/ }, # (3) Look for P or E
    map { .tc },       # (2) Capitalize the words
    <people of earth>; # (1) Start with the input

# Feed (left-to-right) with parentheses, read top-to-bottom
@result = (
    <people of earth>  # (1) Start with the input
    ==> map({ .tc })   # (2) Capitalize the words
    ==> grep /<[PE]>/  # (3) Look for P or E
    ==> sort           # (4) Sort, result is <Earth People>
);

# For illustration, method chaining equivalent, read top-to-bottom
@result =
    <people of earth>  # (1) Start with the input
    .map({ .tc })      # (2) Capitalize the words
    .grep(/<[PE]>/)    # (3) Look for P or E
    .sort;             # (4) Sort, result is <Earth People>

# To assign without the need of parentheses use another feed operator
<people of earth>
    ==> map({ .tc })
    ==> grep /<[PE]>/
    ==> sort()
    ==> @result;

# It can be useful to capture a partial result, however, unlike
# the leftward feed operator, it does require parentheses or a semicolon
<people of earth>
    ==> map({ .tc })
    ==> my @caps; @caps # also could wrap in parentheses instead
    ==> grep /<[PE]>/
    ==> sort()
    ==> @result;
```

这个流操作符能让你在例程之外构建方法链那样的模式并且方法的结果能在不相关的数据上调用。在方法链中, 你被限制于使用数据身上可用的方法或使用之前的方法调用的结果。使用流操作符, 那个限制没有了。写出来的代码比一系列用多个换行符打断的方法调用更加可读。

注: 在将来, 这个操作符会在它获得并行地运行列表操作的能力之后有所变化。它会强制左侧的操作数作为一个闭包(它能被克隆并运行在子线程中)变得可闭合。

## infix <==

这个向左的流操作符从右侧接收结果并把结果作为最后的一个参数传递给它前面的(左侧的)例程。这为一系列列表操作函数阐明了从右到左的数据流。


```perl6
# Traditional structure, read bottom-to-top
my @result =
    sort                   # (4) Sort, result is <Earth People>
    grep { /<[PE]>/ },     # (3) Look for P or E
    map { .tc },           # (2) Capitalize the words
    <people of earth>;     # (1) Start with the input

# Feed (right-to-left) with parentheses, read bottom-to-top
@result = (
    sort()                 # (4) Sort, result is <Earth People>
    <== grep({ /<[PE]>/ }) # (3) Look for P or E
    <== map({ .tc })       # (2) Capitalize the words
    <== <people of earth>  # (1) Start with the input
);

# To assign without parentheses, use another feed operator
@result
    <== sort()             # (4) Sort, result is <Earth People>
    <== grep({ /<[PE]>/ }) # (3) Look for P or E
    <== map({ .tc })       # (2) Capitalize the words
    <== <people of earth>; # (1) Start with the input

# It can be useful to capture a partial result
@result
    <== sort()
    <== grep({ /<[PE]>/ })
    <== my @caps # unlike ==>, there is no need for additional statement
    <== map({ .tc })
    <== <people of earth>;
```

和向右的流操作符不一样, 这个结果不能严格地映射为方法链。然而, 和上面传统的结构中每个参数使用一行分割相比,  feed 操作符写出的代码比逗号更具描述性。向左的流操作符也允许你打断语句并捕获一个可能对调试来说极其方便的中间结果或者接收那个结果并在最终结果身上创建另外一个变种。

注意: 在将来, 这个操作符会在它获得并行地运行列表操作的能力之后有所变化。它会强制右侧的操作数作为一个闭包变得可闭合(它能被克隆并运行在子线程中)

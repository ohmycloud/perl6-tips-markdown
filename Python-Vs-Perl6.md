## zip()

```
# OUTPUT
What is your name?  It is lancelot.
What is your quest?  It is the holy grail.
What is your favorite color?  It is blue.
```

**Python**

```python
questions = ['name', 'quest', 'favorite color']
answers = ['lancelot', 'the holy grail', 'blue']
for q, a in zip(questions, answers):
     print 'What is your {0}?  It is {1}.'.format(q, a)
```

**Perl6**

```perl6
my @questions = ('name', ' quest',  'favorite color');
my @answers = ("lancelot", "the holy grail", "blue");
for zip(@questions, @answers) -> ($question, $answer) {
    say "What is you $question?  It is $answer";
}
```

## 按照日期字符串排序

给定一个列表，按照日期字符串进行排序:

```
list = [ {'date': '2010-04-01','people': 1047, 'hits': 4522},  
         {'date': '2010-04-03', 'people': 617, 'hits': 2582},  
         {'date': '2010-04-02', 'people': 736, 'hits': 3277}
       ]
```
       
**Python**

```python 
import operator 
list = [ {'date': '2010-04-01','people': 1047, 'hits': 4522},  
         {'date': '2010-04-03', 'people': 617, 'hits': 2582},  
         {'date': '2010-04-02', 'people': 736, 'hits': 3277}
       ] 
sorted( list, key = operator.itemgetter('date') )
```

输出(默认是升序):

``` 
{'date': '2010-04-01', 'hits': 4522, 'people': 1047},\
{'date': '2010-04-02', 'hits': 3277, 'people': 736}, \
{'date': '2010-04-03', 'hits': 2582, 'people': 617}
```

**Perl6**

```perl6
my @days = Date.new('2005-02-28') .. Date.new('2016-09-22');
my @list = "/perl6/" <<~<< @days;
 
# 根据日期进行排序

# 降序
my @sorted_des = @list.sort: { $^b.split('/')[2] leg $^a.split('/')[2] }
say .key, "\n", .values for @sorted_des.classy(*.substr(7,7)).sort( { .key } )

# 如果对想对生成的散列再做一下加工（计算每个月有多少天）
# 就对生成的散列使用 map, 利用散列的键和值重新做一个映射
say .key, " ", ~.values for @sorted_des.classy(*.substr(7,7)).map( {   .key => .value.elems  } ).sort( { .key } )
say .key, " ", ~.values for @sorted_des.classy(*.substr(7,7)).map( { ; .key => .value.elems  } ).sort( { .key } )
say .key, " ", ~.values for @sorted_des.classy(*.substr(7,7)).map( {   .key => .value.elems; } ).sort( { .key } )


# 升序
# @list.sort: { .split('/')[2] };
# @list.sort: { $^a.split('/')[2] leg $^b.split('/')[2] }

```

## 统计字符串各字符出现的次数

abcdefghijklmnopqrstuvwxyz...abcdefghijklmnopqrstuvwxyz（1千万个a-z，不可直接a=1千万......）中每个字母的个数。要求除了更好的方式（如更加Pythonic的方式），还要计算越快越好，并打印出代码执行时间。

**Python**

```python
# -*- coding: utf-8 -*-
"""
下面这个例子就是使用 Counter 模块统计一段句子里面所有字符出现次数
"""
from collections import Counter

s = "sssdfdfwewqewfgfhghghgfhgfh10000000"
c = Counter(s)

# 获取出现频率最高的5个字符
print(c.most_common(5))

# Result:
[(' ', 54), ('e', 32), ('s', 25), ('a', 24), ('t', 24)]
```

**Perl6**

```perl6
my $a = ('a' .. 'z').roll(10000000);
say .key, ' => ', .value for $a.cache.classify(*.Str).map({.key => .value.elems}).sort({-.value});
say now - INIT now;
```

输出:（当然现在速度不能忍受 180s左右）

```
q => 385797
a => 385584
o => 385573
e => 385391
c => 385286
p => 385222
...
```

## 获取类的所有方法

**Python**

```python
for item in dir("hello good people"): print(item)
```

**Perl6**

```perl6
for "hello good people".^methods -> $method { say $method }
```

调用类中的每个方法:

```python
import copy

def all_the_methods(thing):
    for meth in dir(thing):
        if meth.startswith('_'):
            print("Skipping " + meth)
            continue

        try:
            print(meth, ' => ', getattr(copy.copy(thing), meth)())
        except Exception as e:
            print(Exception, ":", e)


all_the_methods("hello good people")
```

在 Python 中要将字符串当作**函数名** 使用时, 请使用 `getattr(调用者, 字符串方法)`。 具体请参考 [通过函数名的字符串来调用这个函数](https://taizilongxu.gitbooks.io/stackoverflow-about-python/content/59/README.html)。来看看这一段方法的输出:

```
Skipping __add__
...
capitalize  =>  Hello good people
islower  =>  True
...
lower  =>  hello good people
split  =>  ['hello', 'good', 'people']
<class 'Exception'> : translate() takes exactly one argument (0 given)
...
```

**Perl6**

```perl6
sub all-the-methods($thing) {
  for $thing.^methods -> $method {
    say "{$method.name} => {$thing.clone.$method.gist}";
    CATCH { default { say "{$method.name} => ERROR" } }
  }
}

all-the-methods <this is words>;
all-the-methods "hello little fishies!";
```

接收一系列方法, 遍历这些方法并一次性调用它们。在这里我们做了一次克隆(`.clone`), 以使如果它是可变函数的话我们将在它的一个副本上工作。注意, `.clone` **不是** 深度复制, 所以它虽然能在我这个简单的字符串例子中工作但是它可能在其它东西上没有作用。`.gist` 调用给了我们结果一个很好的可打印版本。

我们不用参数调用那些方法, 很多方法对不带参数不感冒。所以这儿我们提供了一个 CATCH 块以打印出通用信息 -- 它在一个循环里面所以在捕获一个错误之后它会跳转到下一个方法上。

这儿是输出:

```
# <this is words>
permutations => ((this is words) (this words is) (is this words) (is words this) (words this is) (words is this))
join => thisiswords
pick => is
roll => words
reverse => (words is this)
rotate => (is words this)
append => ERROR

# "hello little fishies!"
ords => (104 101 108 108 111 32 108 105 116 116 108 101 32 102 105 115 104 105 101 115 33)
wordcase => Hello Little Fishies!
uc => HELLO LITTLE FISHIES!
flip => !seihsif elttil olleh
chop => hello little fishies
contains => ERROR
```

很多函数的结果是 `ERROR`, 特别是那些讨厌的接收某种参数的方法。然而, 这些不带参数的很多方法能正常工作并返回某些值。原文[All_The_Methods](https://thelackthereof.org/TLT/2016.09.23/All_The_Methods)


## 时髦的过滤函数

**Python**

```python
for f in filter(lambda f: f(-1)>=f(1),
                [lambda x:x, lambda x:x**2, lambda x:x**3]):
    for x in range(-10, 11):
        print x, f(x)
```

**Perl6**

```perl6
for (* ** 1, * ** 2, * ** 3).grep({ $_.(-1) >= $_.(1) }) -> $f {
  for -10 .. 10 -> $x {
    say "$x: { $f.($x) }"
  }
}
```

Whatever-Star 等价于:

```
# Whatever-Star notation
* ** 3

# Pointy-block notation
-> $x { $x ** 3 }

# Subref notation
sub ($x) { $x ** 3 }
```

上面的例子打印:

```
-10: 100
-9: 81
-8: 64
-7: 49
-6: 36
-5: 25
-4: 16
-3: 9
-2: 4
-1: 1
0: 0
1: 1
2: 4
3: 9
4: 16
5: 25
6: 36
7: 49
8: 64
9: 81
10: 100
```

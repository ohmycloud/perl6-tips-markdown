## Perl 6 Colons

在 Perl 6 中到处都是冒号, 我搜集了你在 Perl 6 中使用[冒号](https://thelackthereof.org/Perl6_Colons)的所有方式:


### 命名空间

```perl6
# namespace
package A::B { ... }
class   A::B { ... }

# Namespace separator
my $x = A::B.new;

# Dynamic namespace
my $x = ::("A::B").new;

# Pseudopackage representing null namespace
say ::;

# Anonymous class
class :: is Int {...}

# Current class in a compile-tiem var
class Who { method myname { say ::?CLASS } }

# Call Int method of Int class
42.Int::Int; # => 42

# Treat package X as a hash
say X::.keys;
```

### 类型

```perl6
# T takes value's type
-> Numeric ::T \x { say T }(42); # => (Int)

# Type abverb (smiley)
Int:D;
```

### 绑定

```perl6
# Binding
my $y := $a;

# Container identify
$y =:= $a; # True

# Compile-time (read-only) binding
my $z ::= $y;
```

### Signature

```perl6
# signature literal
my $sig = :(Int $foo);

# signature on Callable &var
my &f:(Str) = -> Str {};
```

### Colon-Pair 语法

```perl6
# Colon-Pair syntax (often seen as adverbs)
my $x = 'cat';
:foo           # 'foo' => True
:foo(True)     # 'foo' => True
:foo(5)        # 'foo' => 5
:foo('dog')    # 'foo' => 'dog'
:foo($x)       # 'foo' => 'cat'

# Value quoting
:foo<bar>      # 'foo' => 'bar'
:foo<bar baz>  # 'foo' => ('bar', 'baz')
:foo<$x>       # 'foo' => '$x'

# value quoting with interpolation
:foo<<$x>>     # 'foo' => 'cat'
:foo<<$x dog>> # 'foo' => ('cat', 'dog')

# Numeric-prefix extraction
say (:73day) # day => 73

# False instead of true
say (:!foo) # foo => False

# Var name as key, content as value
my @foo=1,2;
:@foo # foo => [1,2]
:@foo # foo => [1,2]
:$x   # x   => 'cat'

# Base conversion via colon-pair
say :16("dead") # 57005

# binding pair in sig:
my $a;
sub b (:foo($a)! is rw) { $a = 42};
b(:foo($a));
say $a # 42
```

### Regex

```perl6
# Declare vars in regex scope
my regex foo { :my $var; }

# FUTURE -- http://design.perl6.org/S05.html#Backtracking_control (some of these implemented already, Larry says he plans to do more this year)
my regex bar { a: b:? c:! :: ::> }

# Unicode character classes
say so "a" ~~ /<[:Alpha]>/
```

### Operators as Methods

```perl6
my $a = 1;
$a.infix:["+"](3); # $a + 3
$a.infix:<+>(3);   # $a + 3, but shorter with quote brackets
$a.prefix:<++>;    # ++$a
$a.postfix:<++>;   # ++$a
$a.:<++>;          # Shorthand for prefix (no param)
$a.:<+>(3);        # Shorthand for infix (param provided)
```

### Misc

```perl6
# Precedence dropper
@stuff.map({ $_ + 1 }); # Explicit parameter
@stuff.map: { $_ + 1 }; # Parameters come after ':'

# Invocant marker method invocation
$x.say-hi(names => [<me you>]);  # Normal way
say-hi($x: names => [<me you>]); # Invocant marker way

# object hashes
my $x = :{ (now) => "when?" };

# longname
sub infix:<ya> { ... }

# Specific uses of longnames
use Foo:from<Perl5>;

# Loop labels
MYLABEL: for ^100 {
  for ^200 {
    last MYLABEL;
  }
}

# Twigil for formal named param for a block (like $^x)
say { $:add ?? $^a + $^b !! $^a - $^b }( 4, 5 ) :!add
```

### 评论

下面这两句是等价的。

```perl6
class TreeNode { has TreeNode @children; }
class TreeNode { has ::?CLASS @children; }
```

冒号真是令人惊叹! Awesome.

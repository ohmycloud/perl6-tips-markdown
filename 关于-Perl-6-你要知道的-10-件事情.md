
# Ten Things You Need To Know about Perl 6

```
Jeffrey Goff, Evozon Systems LLC
OSCON London 2016
https://github.com/drforr
http://theperlfisher.blogspot.ro
```


## 资源

* perl6.org (obviously)
* docs.perl6.org - Online documentation
* modules.perl6.org - CPAN Lite

* rakudo.org - Where to get the latest
* https://github.com/tadzik/rakudobrew - Perlbrew for Perl 6
* https://github.com/tadzik/panda/ - Module installer
* irc://irc.freenode.org/perl6 - #perl6 on IRC.freenode.org
* http://rosettacode.org/wiki/Category:Perl_6 - Sample source


## 要闻

* 干净, 一次性的可解析的文法
* 对用户友好的错误信息
* 友好的 Unicode
* 对初学者有帮助的符号
* Math that works
* 函数签名
* 用正则表达式引导
* OO with a type lattice built on a Metaprogramming model
* 自定义运算符
* 内置并发

## 一次性文法

```perl6
my @doctor = { :first('Christopher'), :last('Eccleston'), years => 1 },
             { :first('David'),       :last('Tennant'),   years => 4 },
             { :first('Matt'),        :last('Smith'),     years => 4 },
             { :first('Peter'),       :last('Capaldi'),   years => 3 };

say 'First New Who Doctor: ', @doctor[0]{'first'}, ' ', @doctor[0]<last>;

say "Average run: " ~ @doctor.elems R/ sum map { .<years> }, @doctor;
```

`R/` 运算符会对调两边的表达式再进行计算。

```perl6
my %hash := {  :name('Larry Wall'), :sex('male')  }; # {name => Larry Wall, sex => male}
say %hash{'name'}; #     Larry Wall
say %hash.<name>;  #     Larry Wall
```     

访问散列的键值的另外一种语法是 %hash`.<key>`, 尖括号自动为 `key` 添加引号。

## 错误信息

```perl6
sub mean( @a ) {
  my $sum = sum( @a )
  ( $sum / @a.elems )
}
say mean 1, 2, 4;
```

```
$ perl6 test.pl6

Two terms in a row across lines (missing semicolon or comma?)
at /home/jgoff/test.pl6:2
------>   my $sum = sum( @a )⏏<EOL>
    expecting any of:
        infix
        infix stopper
        postfix
        statement end
        statement modifier
        statement modifier loop
```

## Unicode Friendly

```perl6
my $α = 2 + ⅒  ; 

say $α ÷ 2;

# 1.05
```

## Or, *really* old-school

```perl6
use Slang::Roman;

say 0rↀ  + 0rDCLXVI;

# 1666

my $a = 0rIV / 0rM;

say $a;

# 0.004
```

 
This is really just a teaser to show off how powerful the internals really are.
The module itself is a bit too complex to show, but I like to think that the
fact that I can go in and muck about with the actual grammar, changing how
numbers are read and stored internally speaks volumes.
 
## 有意义的符号

```perl6
my @powers = 1, 2, 4, 8;

my %foo = :name('Jeff'), :division('Programming');

say @powers[ 1, 2 ];

say %foo<name>;

# (2 4)

# Jeff
```

## 浮点数

```
$ perl -E 'say 0.1 + 0.2 - 0.3'

+5.55111512312578e-17

+$ ruby -e 'print 0.1 + 0.2 - 0.3'

+5.55111512312578e-17

+$ python -c 'print 0.1 + 0.2 - 0.3'

+5.55111512312578e-17

+$ perl6 -e 'say 0.1 + 0.2 - 0.3'

+0

+$ perl6 -e'say (0.1).nude'

+(1 10)
```

## 函数签名

```perl6
sub plus( Int $a, Int $b ) {
	$a + $b
}

say plus 1, 2;

# 3

say plus 1, 2, 3;
```

```
===SORRY!=== Error while compiling -e
Calling a(Int, Int, Int) will never work with declared signature
(Int $a, Int $b)
at -e:1
------> ub foo( Int $a, Int $b ) { $a + $b }; say ⏏foo 1, 2, 3;
```

## 不那么正规的表达式

```perl6
my $id = '[ { "first": "Christopher", "last": "Eccleston", "id": 1 } ]';

my regex Str     { '"' (<-[ " ]>+) '"' };
my regex Integer {         \d+         };

my regex Value {                       <Integer> | <Str>               };

my regex Pair  {             <Str> ':' \s+ <Value>                     };
my regex Pairs {     '{' \s+       <Pair>+    %% (',' \s+) \s+ '}'     };
my regex List  { '[' \s+           <Pairs>+   %% (',' \s+)     \s+ ']' };

$id ~~ m{ <List> };

say $/;
```

## 面向对象

```perl6
enum Chirality <left right>;
enum Effort <minimum average maximum>;

class Hand {
	has Chirality $.chirality;
	has $.content; 
}

role Humanoid {
	has Hand $.left-hand  .= new( :chirality( left ) );
	has Hand $.right-hand .= new( :chirality( right ) );
}

class Character does Humanoid {
	has Str $.name is required;
	has Int @.Attr where 0 < * <= 18;

	method attack( Character $enemy, Effort $effort ) { ... } 
}

class Sword { has Int $.Dmg }

my $Wade = Character.new(:name('Wade'));
$Wade.left-hand.content = Sword.new( Dmg => 5 );
$Wade.right-hand.content = Sword.new( Dmg => 5 );

my $Francis = Character.new(:name('Francis'));

$Wade.attack($Francis,maximum);
```

## 看呐,合适的列表!

```perl6
my $fh = open 'sample.tsv', :r;

my @line;
for $fh.lines -> $line {
	my ( $id, $last-name, $first-name ) =
		$line.chomp.split( "\t" );
	@line[$id - 1] = $last-name, $first-name;
}

close $fh;

say @line;

# [("Tyler", "Rose"), ("Smith", "Mickey"),
   ("Jones", "Martha"), ("Noble", "Donna")]
```

## 自定义运算符

```perl6
multi sub prefix:<∑>( *@values ) {
	[+] @values
}

my $total = ∑ 0 ... ∞;
	
multi sub postfix:<⁺>( $x ) { $x.charge( +1 ) }
multi sub postfix:<⁻>( $x ) { $x.charge( -1 ) }
multi sub prefix:<√>( $x ) { sqrt( $x ) }

W⁺ = (-W¹ + iW²)/√2 ;
W⁻ = (-W¹ + iW²)/√2 ;
```

## Perl 6 风格

```perl6
for 'sample.txt'.IO.lines.kv -> $index, $line {
	my ( $method, $id, $name, timestamp ) = $line.chomp.split( "\t" );

	given $method {
		when 'create'	{ %object{$id} = $name, $timestamp }
		when 'delete'	{ %object{$id}:delete }
	}
	if $method eq 'read' | 'update' | 'delete' {
		say "missing object on line $index" unless %object{$id}
	}
}
```

## 简单字符串解析

我已经以好几种方式使用 Perl 6 解析用引号引起的字符串了。 但是我想知道有没有更好更干净的方法。下面有一个为引起的字符串准备的小型 grammar 而且还有一些测试:

```perl6
grammar String::Simple::Grammar {
    our $quote;

    rule TOP {^ <string> $}
    # Note for now, {} gets around a rakudo binding issue
    token string { <quote> {} :temp $quote = $<quote>; <quotebody> $<quote> }
    token quote { '"' | "'" }
    token quotebody { ( <escaped> | <!before $quote> . )* }
    token escaped { '\\' ( $quote | '\\' ) }
}

class String::Simple::Actions {
    method TOP($/) { make $<string>.made }
    method string($/) { make $<quotebody>.made }
    method quotebody($/) { make [~] $0.map: {$^e<escaped>.made or ~$^e} }
    method escaped($/) { make ~$0 }
}

use Test;

plan(5);

my $grammar = ::String::Simple::Grammar;
my $actions = String::Simple::Actions.new();

# The semantics of our string are:
# * Backslash before a backslash is backslash
# * Backslash before a quote of the type enclosing the string is that quote
# * All chars including backslash are otherwise literal

ok $grammar.parse(q{"foo"}, :$actions), "Simple string parsing";
is $grammar.parse(q{"foo"}, :$actions).made, "foo", "Content of matched string";
is $grammar.parse(q{"f\oo"}, :$actions).made, "f\\oo", "Content of matched string";
is $grammar.parse(q{"f\"oo"}, :$actions).made, "f\"oo", "Content of matched string";
is $grammar.parse(q{"f\\\\oo"}, :$actions).made, "f\\oo", "Content of matched string";

```

另外一个版本:

```perl6
grammar String::Simple::Grammar {
    rule TOP {^ <string> $}
    # Note for now, {} gets around a rakudo binding issue
    token string { <quote> {} <quotebody($<quote>)> $<quote> }
    token quote { '"' | "'" }
    token quotebody($quote) { ( <escaped($quote)> | <!before $quote> . )* }
    token escaped($quote) { '\\' ( $quote | '\\' ) }
}

class String::Simple::Actions {
    method TOP($/) { make $<string>.made }
    method string($/) { make $<quotebody>.made }
    method quotebody($/) { make [~] $0.map: {.<escaped>.made // .Str} }
    method escaped($/) { make ~$0 }
}
```

不同之处是:

- 参数化的 rule 用于传递开始的引号
- 更简单版本的 `quotebody` 方法使用了一元的点号和 `//` 用于定义。


[原文](https://www.reddit.com/r/perl6/comments/4snhqr/simple_string_parsing/)

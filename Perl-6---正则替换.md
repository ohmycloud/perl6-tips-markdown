想把 Desgin Perl 6 中的 pod/html 转为 Markdown 格式,  Perl 6 的 pod2markdown 不能用, 只能下载 html 格式的了, 然后用 `pandoc test.html  -o result.markdown` 转换了, 但是也不理想, 里面还有很多  html 标签,  写个脚本批量替换下吧。
token 中的空白要显式地使用 `\s`、`\h`、`\t` 等表示, rule 中 `:sigspace` 是开启的。程序很丑, 仅仅是记录一下。 

```perl6

use v6;

my rule r1 {'<'span id\='"'line_\\d+'"''>''<''/'span'>' '<'span id\='"'line_\\d+'"''>''<''/'span'>'( '<'span)?$}
my rule r2 {id\='"'line_\d+'"''>''<''/'span'>' '<'span id\='"'line_\d+'"''>''<''/'span'>'( '<'span)?$}
my rule r3 {^id\='"'line_\d+'"''>''<''/'span'>'$}
my rule r4 {'<'div class\='"'smartlink'"''>'}
my rule r5 {'<''/'div'>'}
my rule r6 {'<'div class\='"'indexgroup'"''>'}
my rule r7 {'<'span id\='"'__top'"''>''<''/'span'>'}
my token r8 {^  \s* '<'span$}
my rule r9 {^id\='"'line_\d+'"''>''<''/'span'>' '<'span id\='"'line_\d+'"''>''<''/'span'>'$}
my rule r10 {id\='"'line_\d+'"''>''<''/'span'>' '<'span id\='"'line_\d+'"''>''<''/'span'>'(\s'<'span)?$}
my token r11 {^ \s* id\='"'line_\d+'"''>''<''/'span'>'}
my token r12 {^ \s* '<' span \s $}

for dir(test => /.markdown$/) -> $file  {
    my $f = $file;
    $f ~~ s/.markdown//;
    my @lines = $file.lines;
    my $out = open $f ~ ".md", :w;
    for @lines -> $line is rw {
        $line ~~ s/<r1>//;
        $line ~~ s/<r2>//;
        $line ~~ s/<r3>//;
        $line ~~ s/<r4>//;
        $line ~~ s/<r5>//;
        $line ~~ s/<r6>//;
        $line ~~ s/<r7>//;
        $line ~~ s/<r8>//;
        $line ~~ s/<r9>//;
        $line ~~ s/<r10>//;
        $line ~~ s/<r11>//;
        $line ~~ s/<r12>//;
        $out.say($line);
    }
    $out.close;
}

```

有多列数据, 序号, 学校, 课程 … 年份这几列。要如果学校和课程相同就根据年份倒序排列。

先按学校排序, 再按课程排序, 然后按年份倒序排序。

我定义一个类来进行多列数据的排序, 很方便：

``` perl6
class Course {
  has Int $.numb;
  has Str $.univ;
  has Str $.dis;
  has Int $.paper;
  has Int $.cited;
  has Int $.year;
}

my @headers = <numb univ dis paper cited year>;
my @courses;

for $=finish.lines -> $line {
  next if $line ~~ /^num/;
  my @words  = $line.words;
  @words[0, 3,4,5] = @words[0,3,4,5]».Int;
  my %h =  @headers Z=> @words;
  my $course = Course.new(|%h);
  @courses.push($course);
}

my @sorted  = @courses.sort(*.univ).sort(*.dis).sort(-*.year);
for @sorted  {
  say join " ", .numb, .univ, .dis, .paper, .cited, .year;
}

=finish
num	univ	dis	paper	cited	year
1	Beijing	Physics	193	4555	2005
2	Beijing	Physics	197	2799	2006
3	Beijing	Physics	240	2664	2007
4	Beijing	Physics	200	3191	2008
5	Beijing	Physics	268	2668	2009
6	Beijing	Physics	249	2300	2010
7	Beijing	Physics	262	2080	2011
8	Beijing	Physics	230	2371	2012
9	Beijing	Physics	309	1367	2013
10	Beijing	Physics	284	615	2014
11	Beijing	Chemistry	143	1650	2005
12	Beijing	Chemistry	149	2379	2006
13	Beijing	Chemistry	190	2566	2007
14	Beijing	Chemistry	147	1888	2008
15	Beijing	Chemistry	184	2146	2009
16	Beijing	Chemistry	214	2568	2010
17	Beijing	Chemistry	238	2874	2011
18	Beijing	Chemistry	265	2097	2012
19	Beijing	Chemistry	251	1303	2013
20	Beijing	Chemistry	241	656	2014
```

# [下标副词](https://desgin.perl6.org/S02.html#Subscript_adverbs)

为了使切片下标返回除了值以外的其它东西，那么给下标(subscript)添加合适的副词。

```perl6
@array = <A B>;
@array[0,1,2];      # returns 'A', 'B', (Any)
@array[0,1,2] :p;   # returns 0 => 'A', 1 => 'B'
@array[0,1,2] :kv;  # returns 0, 'A', 1, 'B'
@array[0,1,2] :k;   # returns 0, 1
@array[0,1,2] :v;   # returns 'A', 'B'

%hash = (:a<A>, :b<B>);
%hash<a b c>;       # returns 'A', 'B', (Any)
%hash<a b c> :p;    # returns a => 'A', b => 'B'
%hash<a b c> :kv;   # returns 'a', 'A', 'b', 'B'
%hash<a b c> :k;    # returns 'a', 'b'
%hash<a b c> :v;    # returns 'A', 'B'
```

如果副词为真，那么这些副词形式都会清除不存在的条目；如果为假的话，就会留下不存在的项，就像普通的切片那样。所以：

```perl6
@array[0,1,2] :!p;  # returns 0 => 'A', 1 => 'B', 2 => (Any)
%hash<a b c>  :!kv; # returns 'a', 'A', 'b', 'B', 'c', (Any)
```

同样地，

```perl6
my ($a,$b,$c) = %hash<a b c> :delete;
```
删除那些条目并顺道返回它们。这种形式能够工作是因为下标是顶端的在前的操作符。如果某些其它的操作符的优先级比处于顶端的逗号操作符的优先级紧凑，那么你必须用括号括起它或强制为列表上下文：

```perl6
1 + (%hash{$x} :delete);
$x = (%hash{$x} :delete);
($x) = %hash{$x} :delete;
```

只有在副词为真的时候元素才会被删除。而 :!delete 本质上是一个空操作；你可以基于传递的诸如 :delete($kill'em) 标记顺带有条件地删除条目。在任何一种情况下，被删除的值会被返回。

你也可以执行存在性测试，要么测试单个条目是否存在，要么测试条目的连接是否存在：

```perl6
if %hash<foo> :exists           {...}
if %hash{any <a b c>}  :exists  {...}
if %hash{all <a b c>}  :exists  {...}
if %hash{one <a b c>}  :exists  {...}
if %hash{none <a b c>} :exists  {...}
```

把 `:exists` 副词和一组切片结果的布尔值列表结合起来使用，你也可以用类型的语义这样使用：

```perl6
if any %hash<a b c>  :exists {...}
if all %hash<a b c>  :exists {...}
if one %hash<a b c>  :exists {...}
if none %hash<a b c> :exists {...}
```

你可以使用 `:!exists` 来测试不存在。这特别便捷因为优先级规则让 `!%hash<a> :exists` 把 `:exists` 应用到前缀 `!` 上。 `%hash<a> :!exists` 没有那个问题。

## 组合下标副词

像调用中得具名参数那样，下标中处理多个副词是没有顺序之分的。有些组合有意义，例如：

```perl6
 %a = %b{@keys-to-extract} :delete :p; # same as :p :delete
```

会把给定的键分片到另外一个散列中。而

```perl6
@actually-deleted = %h{@keys-to-extract} :delete :k; # same as :k :delete
```

会返回真正从散列中删除的键。

只指定返回类型的副词，不能被组合，因为诸如 `:kv :p`、或 `:v :k` 就没有意义。

下面的这些副词组合被看做是合法的：

```perl6
:delete :kv            delete, return key/values of actually deleted keys
:delete :!kv           delete, return key/values of all keys attempted
:delete :p             delete, return pairs of actually deleted keys
:delete :!p            delete, return pairs of all keys attempted
:delete :k             delete, return actually deleted keys
:delete :!k            delete, return all keys attempted to delete
:delete :v             delete, return values of actually deleted keys
:delete :!v            delete, return values of all keys attempted
:delete :exists        delete, return Bools indicating keys existed
:delete :!exists       delete, return Bools indicating keys did not exist
:delete :exists :kv    delete, return list with key,True for key existed
:delete :!exists :kv   delete, return list with key,False for key existed
:delete :exists :!kv   delete, return list with key,Bool whether key existed
:delete :!exists :!kv  delete, return list with key,!Bool whether key existed
:delete :exists :p     delete, return pairs with key/True for key existed
:delete :!exists :p    delete, return pairs with key/False for key existed
:delete :exists :!p    delete, return pairs with key/Bool whether key existed
:delete :!exists :!p   delete, return pairs with key/!Bool whether key existed
:exists :kv            return pairs with key,True for key exists
:!exists :kv           return pairs with key,False for key exists
:exists :!kv           return pairs with key,Bool for key exists
:!exists :!kv          return pairs with key,!Bool for key exists
:exists :p             return pairs with key/True for key exists
:!exists :p            return pairs with key/False for key exists
:exists :!p            return pairs with key/Bool for key exists
:!exists :!p           return pairs with key/!Bool for key exists
```

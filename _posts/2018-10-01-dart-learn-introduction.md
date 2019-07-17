---
layout: post
title: Dart语言学习笔记 — Dart简介
date: 2018-10-01 18:32:24.000000000 +09:00
tag: Flutter
---

### 简介

> `Dart`是一种适用于万维网的开放源代码编程语言，由Google主导开发，于2011年10月公开。它的开发团队由Google Chrome浏览器V8引擎团队的领导者拉尔斯·巴克主持，目标在于成为下一代结构化`Web`开发语言。类似`JavaScript`，`Dart`也是一种面向对象语言，但是它采用基于类编程。它只允许单一继承，语法风格接近`C`语言。 --维基百科

### Dart长什么样：

``` Dart
// 定义一个函数。
printInteger(int aNumber) {
  print('The number is $aNumber.'); // 使用字符串插值写法，打印到控制台。
}

// 这是应用程序开始执行的地方。
main() {
  var number = 42; // 声明并初始化变量。
  printInteger(number); // 调用一个函数。
}
```
> 注意：Dart语言的函数必须要先声明，再调用，声明代码要写在调用代码前面。

### Dart变量类型

1.使用`var`声明一个变量并赋值（官方推荐）：
``` Dart
var name = 'Bob' ; 
```
变量存储引用。变量name包含对String值为“Bob” 的对象的引用。name推断变量的类型String，但也可以通过指定它来更改该类型。

2.如果对象不限于单一类型，还可以使用`dynamic`来进行声明:
``` Dart
dynamic name = 'Bob' ;
```

3.另一种选择是显式声明可以推断出的类型：
``` Dart
String name = 'Bob' ; 
```
 备注：如果只声明变量不赋值，则默认值为`null`，包括数字类型变量。在`Dart`里一切都是对象，数字也是一种对象类型，例如：
 ``` Dart
 num lineCount;  
 lineCount ==null;// true  
 ```
### final 与 const
如果不打算更改变量，请使用`final`或`const`声明。`filal`与`const`变量都只能在初始化时赋值一次，声明后不可再改变; `const`变量是编译时常量。
> 注意： 实例变量可以是final，但不能是const。

以下是创建和设置最终变量的示例：
``` Dart
final name = 'Bob' ; //没有类型注释
final String nickname = 'Bobby' ;  
```
  
无法更改`final`变量的值：
``` Dart
name = 'Alice' ; //错误：final变量只能设置一次。  
```

### Dart内置类型

Dart语言特别支持以下类型：
- numbers
- strings
- booleans
- lists (也称为数组)
- maps
- runes (用于表示字符串中的Unicode字符)
- symbols

因为Dart中的每个变量都引用一个对象 ，一个类的实例，你通常可以使用构造函数来初始化变量。一些内置类型有自己的构造函数。例如，你可以使用`Map()`构造函数来创建map对象。

#### 1.有两种类型
`int`:整数值不大于64位，具体取决于平台。

`double`:64位（双精度）浮点数。

`int`和`double`是number的子类型。

字符串与数字类型互转：
``` Dart
// String -> int
var one = int.parse('1');
assert(one == 1);

// String -> double
var onePointOne = double.parse('1.1');
assert(onePointOne == 1.1);

// int -> String
String oneAsString = 1.toString();
assert(oneAsString == '1');

// double -> String
String piAsString = 3.14159.toStringAsFixed(2);
assert(piAsString == '3.14');
```

#### 2. Strings
可以使用单引号' '或双引号" "来创建字符串：
``` Dart
var s1 = 'Single quotes work well for string literals.';
var s2 = "Double quotes work just as well.";
```


可以通过` ${expression}`将一个表达式放入字符串中，如果表达式`expression`是一个标识符，则可以直接省略花括号`{}`，为了获取与对象对应的字符串， Dart 会调用对象的 toString () 方法。

可以使用相邻的字符串文字或+ 运算符连接字符串：
``` Dart
var s1 = 'String ''concatenation'" works even over line breaks.";
assert(s1 == 'String concatenation works even over ''line breaks.'); 

var s2 = 'The + operator ' + 'works, as well.';
assert(s2 == 'The + operator works, as well.'); 
```
创建多行字符串的另一种方法：使用带有单引号或双引号的三重引号：
``` Dart
var s1 = '''
You can create
multi-line strings like this one.
''';

var s2 = """This is also a
multi-line string.""";
```

可以通过为其添加前缀来创建“原始”字符串`r`：

``` Dart
var s = r'In a raw string, not even \n gets special treatment.';
```

#### 3. Booleans

布尔值使用`bool`声明，取值为:`true` 和 `false` ，它们都是编译时常量。

``` Dart
// Check for an empty string.
var fullName = '';
assert(fullName.isEmpty);

// Check for zero.
var hitPoints = 0;
assert(hitPoints <= 0);

// Check for null.
var unicorn;
assert(unicorn == null);

// Check for NaN.
var iMeantToDoThis = 0 / 0;
assert(iMeantToDoThis.isNaN);
```

#### 4. Lists

Dart里的`list`与JavaScript里的`list`类似，以下是声明方式:
``` Dart
var list = [1, 2, 3];
```

> 注意： 解析器推断出list有类型List<int>。如果尝试将非整数对象添加到此列表，则解析器或运行时会引发错误。

列表的索引使用从0开始，其中0是第一个元素的索引，`list.length - 1`是最后一个元素的索引。

#### 5. Maps
通常，映射是关联键和值的对象。键和值都可以是任何类型的对象。每个键只出现一次，但可以多次使用相同的值。

声明示例：
``` Dart
var gifts = {
  // Key:    Value
  'first': 'partridge',
  'second': 'turtledoves',
  'fifth': 'golden rings'
};

var nobleGases = {
  2: 'helium',
  10: 'neon',
  18: 'argon',
};
```

> 注意： 该解析推断`gifts`元素类型 Map<String, String>，且`nobleGases`元素类型 Map<int, String>。如果你尝试将错误类型的值添加到任一映射，则解析器或运行时会引发错误。

也可以使用Map构造函数创建相同的对象：
``` Dart
var gifts = Map();
gifts['first'] = 'partridge';
gifts['second'] = 'turtledoves';
gifts['fifth'] = 'golden rings';

var nobleGases = Map();
nobleGases[2] = 'helium';
nobleGases[10] = 'neon';
nobleGases[18] = 'argon';
```
> 注意：可能希望看到new Map()而不仅仅是Map()。从Dart 2开始，`new`关键字是可选的。

``` Dart
var gifts = {'first': 'partridge'};

// 增
gifts['fourth'] = 'calling birds'; // Add a key-value pair

// 删
gifts.remove('first');

// 改
gifts['fourth'] = '1234'; 

// 查
var item = gifts['fourth'];
```

#### 6. Functions

Dart是一种真正的面向对象语言，因此即使是函数也是对象，并且具有Function类型。这意味着函数可以分配给变量或作为参数传递给其他函数。你也可以像调用函数一样调用Dart类的实例。

函数示例：
``` Dart
bool isNoble(int atomicNumber) {
  return _nobleGases[atomicNumber] != null;
}

// 可以省略返回值类型
isNoble(atomicNumber) {
  return _nobleGases[atomicNumber] != null;
}

// 箭头函数
bool isNoble(int atomicNumber) => _nobleGases[atomicNumber] != null;

```


### 重要的概念
当你了解Dart语言时，请记住以下事实和概念：

 - 你可以放在变量中的所有内容都是一个对象，每个对象都是一个类的实例。偶数，函数和 null对象。所有对象都从Object类继承。

 - 尽管Dart是强类型的，但类型注释是可选的，因为Dart可以推断类型。在上面的代码中，number 推断为类型int。如果要明确说明不需要任何类型，请使用特殊类型dynamic。

 - Dart支持泛型类型，如List<int>（整数列表）或List<dynamic>（任何类型的对象列表）。

 - Dart支持顶级函数（例如main()），以及绑定到类或对象的函数（分别是静态和实例方法）。你还可以在函数内创建函数（嵌套函数或本地函数）。

 - 类似地，Dart支持顶级变量，以及绑定到类或对象的变量（静态和实例变量）。实例变量有时称为字段或属性。

 - 与Java，Dart不具备关键字public，protected和private。如果标识符以下划线（_）开头，则它对其库是私有的。

 - 标识符可以以字母或下划线（_）开头，后跟这些字符加数字的任意组合。

 - 有时候事物是表达还是陈述是很重要的，因此有助于准确地理解这两个词。

 - Dart工具可以报告两种问题：警告和错误。警告只是表明你的代码可能无法正常工作，但它们不会阻止你的程序执行。错误可以是编译时或运行时。编译时错误会阻止代码执行; 运行时错误导致 代码执行时引发异常。


尾记：文章为本人学习Dart时做的笔记，这样做学习起来，知识框架相对比较完整，也容易反复查看，也方便其他小伙伴查看。文章所有内容几乎都来自[Dart programming language](https://www.dartlang.org/guides/language/language-tour)，部分来自互联网，如有雷同，纯属是我抄袭。哈哈~
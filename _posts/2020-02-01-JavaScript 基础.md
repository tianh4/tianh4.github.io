---
layout: mypost
title: JavaScript 基础
categories: [JavaScript, Note]
---

# 1 JavaScript 基础

## 1.1 Java Script 介绍

```html
<!DOCTYPE HTML>
<html>
    <body>
        <script>
          alert("hello,world!");
        </script>
    </body>
</html>
```

在每一条语句后用分号结束，有时可以自动分号插入，有时会出错。

用两个正斜杠开始单行注释，如 `// This is a comment` 。

由单斜杠和分号开始多行注释，如

```javascript
/*  Comment first line,
	Comment second line.
*/
```

2009 年，ECMAScript 5 标准出现，通过 `"use strict"` 开启新特性。

这个语句放在整个脚本或函数的顶部。

本教程使用严格模式。

## 1.2 变量和类型

声明变量，可以在声明的同时赋值

```javascript
let message = "hello";
alert(message);
```

变量名仅包含字母，数字，$ 和 _ ，首字符不能是数字。

如果命名包含多个单词，通常使用驼峰命名法。

声明一个常量可以使用

```javascript
const myBirthday = "18.04.1982";
```

常量通常用大写字母加下划线命名。

JavaScript 是一门动态类型语言。有七种基本的数据类型。

### number

整数和浮点数，可以进行加减乘除。

* Infinity

  使用 `alert(1/0)` 可以得到这个值，比任何数都大。

* NaN

  表示非数，对 NaN 的计算结果都是 NaN。

### string

字符串，被引号括起来。

反引号表示功能拓展，可以在 `${}` 中放变量和表达式,如

```javascript
alert(`the results of one plus one is ${1+1}`)
```

### boolean

值是 true 和 false。

### null

不是 null 指针，就是一个值，表示无。

### undefined

已经被声明，但未被赋值，就是 undefined。

### symbol 和 object

对象，更复杂的数据集合。

类型之间可以转换。

### 字符串转换

用 `String(value)` 。

### 数字型转换

在表达式中，自动执行数字型转换。

也可以用 `Number(value)` 。

如果不可以转换，会得到 NaN。

| 值          | 变成 ...                                             |
| ----------- | ---------------------------------------------------- |
| undefined   | NaN                                                  |
| null        | 0                                                    |
| true, false | 1, 0                                                 |
| string      | 空字符串得到 0，去掉首尾空格后得到数字，出错得到 NaN |

### 布尔型转换

* 数字 0，null，undefined，NaN 得到 false
* 其他得到 true

## 1.3 运算符

### 运算符优先级

一元运算符优先级高于二元运算符，乘除高于加减，算术运算高于赋值运算。

### 二元运算符 +

通常 + 用于数字型求和。

但是 + 用于字符串可以连接字符串，如果有一侧不是字符串会自动转换为字符串。

### 一元运算符 +

如果 + 用于数字型，没有任何作用。

但是 + 用于非数字型，会转换为数字型。

位运算符参见：https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Operators/Bitwise_Operators

### 逗号运算符

逗号运算符的优先级**非常非常非常**低，它执行各个语句，但是只返回最后一个语句的结果。

## 1.4 alert() prompt() confirm()

## 1.5 条件运算符

```javascript
let message = (age < 3) ? 'Hi, baby!' :
			  (age < 18) ? 'Hello!' :
			  (age < 100) ? 'Greetings!' :
			  'What an unusual age!';
alert(message);
			  
```


---
title: Unicode 字符编码详讲
date: 2022-11-14 10:23:26
categories:
- 字符编码
tags: 
- Unicode
- Java
toc: true
---
Unicode 又称为统一码、万国码、单一码，是国际组织制定的旨在容纳全球所有字符的编码方案，包括字符集、编码方案等，它为每种语言中的每个字符设定了统一且唯一的二进制编码，以满足跨语言、跨平台的要求。

Unicode的实现方式不同于编码方式。一个字符的Unicode编码确定。但是在实际传输过程中，由于不同系统平台的设计不一定一致，以及出于节省空间的目的，对Unicode编码的实现方式有所不同。Unicode的实现方式称为Unicode转换格式（Unicode Transformation Format，简称为UTF）。

Unicode 与 UTF-X 的关系：Unicode 是字符集，UTF-32/ UTF-16/ UTF-8 是三种常见的字符编码方案。
<!--more-->
## Unicode 码点
- **码点（Code Point）**
相当于 ASCII 码中的 ASCII 值，它就是 Unicode 字符集中唯一表示某个字符的标识。

- **码元（Code Unit）**
代码单元是指讲以实际 Unicode 编码方案将字符存储在计算机中，所占用的代码单元。对于 UTF-8 来说，代码单元是 8 位二进制数，UTF-16 则是 16 位二进制数。

- **码点的表示形式**
码点的表示形式为 `U+[XX]XXXX`，X 代表一个十六进制数，可以有 4-6 位，不足 4 位前补 0，补足 4 位，超过则按实际位数。具体范围是 U+0000 ~ U+10FFFF。理论大小为 `10FFFF + 1 = 0x110000`，即 `17 * 2^16 = 17 * 65536`。
## 平面
为了更好分类管理庞大的码点数，Unicode 把每 65536 个码点作为一个平面，总共 17 个平面（Plane）。

17 个平面中的第一个平面即：BMP（Basic Multilingual Plane 基本多语言平面）。也叫 Plane 0，它的码点范围是 `U+0000 ~ U+FFFF`。这也是我们最常用的平面，日常用到的字符绝大多数都落在这个平面内。UTF-16 只需要用两字节编码此平面内的字符。没包括进来的字符，放在后面的平面，称为辅助平面。

## Unicode 编码
码点仅仅是一个抽象的概念，是把字符数字化的一个过程，仅仅是一种抽象的编码。
整个编码可分为两个过程。首先，将程序中的字符根据字符集中的编号数字化为某个特定的数值，然后根据编号以特定的方式存储到计算机中。
### Unicode 编码的两个层面
1. **抽象编码层面**
把一个字符编码到一个数字。不涉及每个数字用几个字节表示，是用定长还是变长表示等具体细节。
2. **具体编码层面**
码点到最终编码的转换即 UTF （Unicode Transformation Format：Unicode 转换格式）。
这一层是要把抽象编码层面的数字（码点）编码成最终的存储形式（UTF-X）。码点（code）转换成各种编码（encode），涉及到编码过程中定长与变长两种实现方式，定长的话定几个字节；用变长的话有哪几种字节长度，相互间如何区分等等。

>UTF-32 就属于定长编码，即永远用 4 字节存储码点，而 UTF-8、UTF-16 就属于变长存储，UTF-8 根据不同的情况使用 1-4 字节，而 UTF-16 使用 2 或 4 字节来存储码点。


>注：在第一层，抽象编码层面，字符与数字已经实现一一对应，对数字编码实质就是对字符编码。

### BOM
BOM（Byte Order Mark），即`字节顺序标识`。它用来标识使用哪种端法，它常被用来当做标识文件是以 UTF-8、UTF-16 或 UTF-32 编码的标记。

在 Unicode 编码中有一个叫做`零宽度非换行空格`的字符 ( ZERO WIDTH NO-BREAK SPACE ), 用字符 FEFF 来表示。

对于 UTF-16 ，如果接收到以 FEFF 开头的字节流， 就表明是大端字节序，如果接收到 FFFE， 就表明字节流 是小端字节序

UTF-8 没有字节序问题，上述字符只是用来标识它是 UTF-8 文件，而不是用来说明字节顺序的。"零宽度非换行空格" 字符 的 UTF-8 编码是 EF BB BF, 所以如果接收到以 EF BB BF 开头的字节流，就知道这是UTF-8 文件。

下面的表格列出了不同 UTF 格式的固定文件头：
|UTF 编码|	固定文件头|
|:--:|:--:|
|UTF-8|	EF BB BF|
|UTF-16LE|FF FE|
|UTF-16BE|FE FF|
|UTF-32LE|FF FE 00 00|
|UTF-32BE|00 00 FE FF|

## Java 获取字符串长度
```java
String s1 = "Hello";
String s2 = "😂😂😂😂";
// 获取字符串长度（等于每个字符的 Unicode 码元个数之和）
int length1 = s1.length();
int length2 = s2.length();
// 获取精准的字符串长度（等于每个字符的 Unicode 码点个数之和）
int exactLength1 = s1.codePointCount(0, s1.length());
int exactLength2 = s2.codePointCount(0, s2.length());
// 输出打印
System.out.println("s1: " + s1);
System.out.println("s2: " + s2);
System.out.println("length1: " + length1);
System.out.println("length2: " + length2);
System.out.println("exact length1: " + exactLength1);
System.out.println("exact length2: " + exactLength2);
```
输出如下：
```
s1: Hello
s2: 😂😂😂😂
length1: 5
length2: 8
exact length1: 5
exact length2: 4
```
Java 默认使用 `UTF-8` 对字符进行编码。当字符串中包含中文或特殊字符时，用 `String.length()` 方法可能无法精确获得字符串的个数，因为该方法实际返回的是每个字符的 Unicode 码元个数之和（UTF-8 的码元为一个 8 位二进制数），而中文或特殊字符往往占用不止一个码元（Code Unit），所以获得的数值回大于实际字符串长度。
使用 `String.codePointCount(int beginIndex, int endIndex)` 方法则可以获取精准的字符串长度，因为该方法返回指定范围内每个字符的 Unicode 码点个数之和，而一个 Unicode 码点则对应一个实际字符。
# 典型回答

MySQL中是可以存储emoji表情的，但是要使用UTF8MB4的字符编码才可以。如果是UTF8MB3的话，存储这些扩展字符的话会无法解析导致报错。

# 扩展知识

## MySQL对Unicode的支持

Unicode字符集，他现在已经是计算机科学领域里的一项业界标准，它对世界上大部分的文字系统进行了整理、编码，使得计算机可以用更为简单的方式来呈现和处理文字。

为了适应不同的数据存储和传递需求，人们提出了 Unicode Transformation Format（UTF）系列编码。这其中包含UTF-8、UTF-16、UTF-32等。

通过查阅MySQL官方文档（[https://dev.mysql.com/doc/refman/8.0/en/charset-unicode.html](https://dev.mysql.com/doc/refman/8.0/en/charset-unicode.html) ），我们可以知道，**在MySQL中，主要支持以下字符集：utf8、ucs2、utf8mb3、utf8mb4、utf16、utf16le和utf32**

不同的字符集的区别在于包含的字符情况以及存储需要的空间。

| 字符集 | 支持的字符 | 每个字符所需存储空间 |
| --- | --- | --- |
| utf8mb3, utf8 | BMP | 1-3 字节 |
| ucs2 | BMP | 2 字节 |
| utf8mb4 | BMP和补充字符 | 1-4 字节 |
| utf16 | BMP和补充字符 | 2或4 字节 |
| utf16le | BMP和补充字符 | 2或4 字节 |
| utf32 | BMP和补充字符 | 4 字节 |


在MySQL官方文档中，介绍了支持的编码方式之后，还有一段醒目的提醒：

![](http://www.hollischuang.com/wp-content/uploads/2021/05/16205427559305.jpg#id=t73Bz&originHeight=312&originWidth=2110&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

翻译过来是：**utf8mb3字符集已被弃用，它在未来的MySQL版本中将会被删除，请使用utf8mb4代替。在目前的8.0版本中，utf8指的就是utf8mb3，虽然未来可能改成utf8mb4，但是为了避免产生歧义，可以考虑为字符集引用显式指定utf8mb4，而不是utf8。**

也就是说，当我们在MySQL 8.0 中指定字符编码方式为UTF-8的时候，其实使用的是utf8mb3这种编码方式。

那么，我们先来说说utf8mb3。

## utf8mb3

utf8mb3字符集是MySQL早期就支持的字符集，他具有以下特征:

1、仅支持BMP字符(不支持补充字符)

2、每个多字节字符最多需要三个字节

注意，仅支持BMP字符，那么什么是BMP字符呢？

BMP是Basic Multilingual Plane的缩写，即码位在0到65535之间(或者U+0000和U+FFFF)的字符。

**BMP中并不包含补充字符**，即码位在U+10000和U+10FFFF之间的字符。补充字符有哪些呢，如一些生僻的汉字，或者Emoji 表情等都是补充字符。

**也就是说，如果在建表的时候，指定的编码方式是utf8mb3（utf-8），那么对于一些生僻字或者emoji表情都无法表示。**

## utf8mb4

早期的时候，Unicode 只用到了 0~0xFFFF 范围的数字编码，这就是 BMP 字符集。所以，最初MySQL在设计之初，也就只涉及了包含BMP 字符集的utfmb3(utf-8)，但是随着文字越来越多，3个字节肯定无法全部表示，于是Unicode支持的字符就更多了。

所以，早期的utfmb3在有些场景中就不能满足需求了，于是，**MySQL在5.5.3之后增加了utf8mb4的编码。**

utfmb4字符集具有以下特征:

1、支持BMP和补充字符。

2、每个多字节字符最多需要4个字节。

utf8mb4与utf8mb3字符集不同，utf8mb3字符集只支持BMP字符，每个字符最多使用三个字节:

对于BMP字符，utf8mb4和utf8mb3具有相同的存储特征，即相同的编码值，相同的编码，相同的长度。

对于补充字符，utf8mb4需要4个字节来存储它，而utf8mb3根本不能存储该字符。所以我们说utf8mb4是utf8mb3的超集。

**所以，很多时候，为了考虑到兼容性，建议创建MySQL表的时候，使用utf8mb4，而不是utf8！**

## utf8mb3和utf8mb4区别及优缺点

前面分别介绍了utf8mb3和utf8mb4字符集，他们的区别如下:

**utf8mb3只支持BMP (Basic Multilingual Plane)的字符。utf8mb4还支持BMP之外的补充字符。**

**utf8mb3每个字符最多使用3个字节。Utf8mb4每个字符最多使用4个字节。**

utf8mb4比utf8mb3来说，他能表示更多的补充字符，但是同时占用的空间可能会更大一些。

## 从utf8mb3转换成utf8mb4

首先，想要把字符集从utf8mb3转换到utf8mb4，其实是问题不大的:

对于BMP字符，utf8mb4和utf8mb3具有相同的存储特征:相同的编码值，相同的编码，相同的长度。

对于补充字符，utf8mb4需要4个字节来存储它，而utf8mb3根本不能存储该字符。当将utf8mb3列转换为utf8mb4时，您不必担心转换补充字符，因为没有补充字符。

假设有一张已知表使用了utf8mb3：

```
CREATE TABLE t1 (
  col1 CHAR(10) CHARACTER SET utf8 COLLATE utf8_unicode_ci NOT NULL,
  col2 CHAR(10) CHARACTER SET utf8 COLLATE utf8_bin NOT NULL
) CHARACTER SET utf8;
```

下面的语句将t1转换为utf8mb4:

```
ALTER TABLE t1
  DEFAULT CHARACTER SET utf8mb4,
  MODIFY col1 CHAR(10)
    CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci NOT NULL,
  MODIFY col2 CHAR(10)
    CHARACTER SET utf8mb4 COLLATE utf8mb4_bin NOT NULL;
```



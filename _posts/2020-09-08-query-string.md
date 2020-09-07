---
layout: post
title: "使用query_string查询"
subtitle: ''
tags:
  - 笔记 
  - Elasticsearch
---

> 本文是[query-string-syntax](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-query-string-query.html#query-string-syntax)的翻译。

关于query string语法

在搜索API中，可以使用`query string`进行查询，整个`query string`在API中，做为一个参数

一个`query string`可以被分解成多个条件和操作符，一个条件可以是一个词语，短语，或者是由`"`包起来的全字匹配的字符串。

通过操作符组合条件，我们可以进行各种复杂的查询


## 过滤Document的字段

可以通过以下示例的语句，过滤指定的字段

- 要求`status`字段包含`active`

```
status:active
```

- 要求`title`字段包含`quick` 或者 `brown`

```
title:(quick OR brown)
```

- 要尔`author`字段包括有`john smith`这个整体，单独包含John或者Smith都不算

```
author:"John Smith"
```

- 要求`first name`字段包含`Alice`(注意我们处时字段中有空格时的场景，使用 \ 做转义)

```
first\ name:Alice
```

- 元素`book`下的字段，如`book.title`, `book.content` 或者 `book.date` 包含有 `quick` or `brown` (请注意，我们使用\来转义\*):

```
book.\*:(quick OR brown)
```

- 要求字段的字不是null,即有这个字段

```
_exists_:title
```

## 通配符

在搜索term时，可以使用`?`来表示存在一个字符串，或者`\*`来表示一个或多个字符串。示例如下：

```
qu?ck bro*
```

通配符的使用，会增加内存消耗，并大大的降底程序的查询效率，想像一相，但我们查询`"a* b* c*"`时，需要匹配多term


> Pure wildcards \* are rewritten to exists queries for efficiency. As a consequence, the wildcard "field:*" would match documents with an empty value like the following:
``
{
  "field": ""
}`
``
... and would not match if the field is missing or set with an explicit null value like the following:
{
  "field": null
}

## 模糊搜索 

我们有需要搜索一些相似的词，而不是精确的包含，这时可以使用“fuzzy”operator。

比如：

```
quikc~ brwn~ foks~
```

ElasticSearch使用 [Damerau-Levenshtein distance](https://en.wikipedia.org/wiki/Damerau-Levenshtein_distance) 来确认2个词语的不同点,最多只能有2个不一致。

默认的编辑距离是2，不过如果编辑距离是1，我们能处理80%的人为拼写错误产生的模糊搜索。

比如把`quick`拼错成`quikc`


> **不要在模糊搜索中使用通配符**
当模糊搜索遇到通配符时，只有一个操作生效， 比如, 你可以搜索  `app~1` (模糊搜索) 或者 `app*` (通配符), 但搜索  `app*~1` 不会触发模糊搜索 .

## 区间搜索

区间可以作用于日期，数字，或者字符串字段。区间包括边界时通过`[`和`]`将最小值与最大值包围起来，其格式为`[最小值 TO 最大值]`，当不包括边界时，使用`{最小值 TO 最大值}`表示。

具体示例如下：

- 表示2012年的所有天：

```
date:[2012-01-01 TO 2012-12-31]
```

- 从1到5的数字：

```
count:[1 TO 5]
```

- 从alpha 到 omega 的标签，但不包括alpha 和 omega：

```
tag:{alpha TO omega}
```

- 大于10的数字：

```
count:[10 TO *]
```

- 2012以前的日期：

```
date:{* TO 2012-01-01}
```

- `[`,`]`与`{`,`}`混合使用：

Numbers from 1 up to but not including 5

```
count:[1 TO 5}
```

- 对于只限制一个边界的区域，可以使用如下的方式表达：

```
age:>10
age:>=10
age:<10
age:<=10
```

## 同时支持多个查询条件

使用`OR`, `AND`将多个查询条件连接，可以支持`或`与`与`的联合查询，

如下查询表示：level字段为info或者warning，并且msg字段的内容，包括有Success

```
(level:info OR level:warning) AND msg:Success
```



## 参考来源

- [https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-query-string-query.html#query-string-syntax](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-query-string-query.html#query-string-syntax)

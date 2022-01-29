---
layout: post
title: "使用Excel公式IF,IFERROR统计分析URL"
subtitle: ''
tags:
  - Excel公式
  - IFERROR
  - IF
---

最近进行了一项推广活动，给每个用户发放一个推荐码，然后基于百度统计收集推荐码的使用情况。

## 用到的几个函数

### FIND

`FIND`函数用于查找字符子串。如果找到了子串，并返回子串的位置。

其函数签名为：

```
FIND(find_text,within_text,start_num)
```

- `find_text`要查找的文本，如"?p_code="
- `within_text`是包含要查找文本的文本，即从`within_text`中找`find_text`
- `start_num`用于指定开始进行查找的字符。within_text 中的首字符是编号为 1 的字符。如果省略 start_num，则假定其值为 1。

**示例**

```
FIND("?p_code=",B3,1)
```

如上所示，表示要从B3这个单元格式，查找子串"?p_code="，从第1个字符串开始查找。如果找到，则返回子串“?p_code=”的位置。

**公式说明**

- 如果 find_text 为空文本 ("")，则 FIND 会匹配搜索字符串中的首字符（即编号为 start_num 或 1 的字符）。
- Find_text 不能包含任何通配符。
- 如果 within_text 中没有 find_text，则 FIND 返回错误值 #VALUE!。
- 如果 start_num 不大于零，则 FIND 返回错误值 #VALUE!。
- 如果 start_num 大于 within_text 的长度，则 FIND 返回错误值 #VALUE!。

IF函数的官方手册可以从这里[查看](https://support.microsoft.com/zh-cn/office/find-findb-%E5%87%BD%E6%95%B0-c7912941-af2a-4bdf-a553-d0d89b0a0628)

### IF

`IF`负责进行逻辑判断，并根据逻辑判断的结果，返回对应的值。此函数支持3个参数。

其函数签名为：

```
IF(logical_value,value_if_true,value_if_false)
```

参数说明：

 - `logical_value`为要测试的表达式或值
 - `logical_value`为真时的返回值
 - `logical_value`为假时的返回值

**示例**

```
IF(FIND("?p_code=",B3,1)>0,1,0)
```

如上示例所示，表达式的意在执行这个逻辑：`FIND("?p_code=",B3,1)>0`为真时，返回1，否则返回0

IF函数的官方手册可以从这里[查看](https://support.microsoft.com/zh-cn/office/if-%E5%87%BD%E6%95%B0-69aed7c9-4e8a-4755-a9bc-aa8bbff73be2?ui=zh-cn&rs=zh-cn)

### IFERROR

此函数的作用为，在预知可能发生错误的情况下，显示一个特定的结果。

其函数签名为：

```
IFERROR(value, value_if_error)
```

参数说明：

- value，必需参数，用于检查是否存在错误的参数（表达式），value在计算过程中可能会产生一个错误，比如这个表达式`3/B1`,如果B1单元格的值为0，则会产生一个`#DIV/0!`的错误。
- value_if_error 当value参数计算时出现了错误时，此公式的返回值

> 当`value`的计算过程正常，未出现错误时，则返回公式`value`的值。

IFERROR函数的官方手册可以从这里[查看](https://support.microsoft.com/zh-cn/office/iferror-%E5%87%BD%E6%95%B0-c526fd07-caeb-47b8-8bb6-63f3e417f611)


## 获取网站被访问的URL列表

百度统计提供了指定时间段内，访问URL的数据汇总。

获取路径为：网站的统计首页 => 来源分析 => 全部来源。然后从右上角选择`下载`，即可得到一份关于所有被访问URL的csv文件。

![csv示例](https://oss-cn-hangzhou.aliyuncs.com/codingsky/assets/pb/codingsky_access_urls.png)

CSV文件数据示例。

基于这份CSV文件，我们可以统计所有的特定邀请码的PV和UV

## 计算推荐码的PV

先直接输出公式，如下所示：

```
=IFERROR(IF(FIND("?p_code=",B2,1)>0,C2,0),0)
```

直接套用上面的函数，从最里层往外一层层剥开就是：

- 在`B2`单元格的内容中查找`?p_code=`
- 如果找到了，是返回单元格`C2`的值，找不到则返回0
- 如果检查`FIND`函数，则会发现，`IF`函数走不到返回0的逻辑，因为如果`FIND`函数无法找到时，会出错并返回`#VALUE!`
- 当`FIND`出错时，被`IFERROR`函数接住，返回0

串起来，这个逻辑就是：在`B2`单元格中找`?p_code=`，找到就返回单元格`C2`的值，否则就返回0

> 如上图所示，百度CSV文件导出时，C2单元格就是此URL的PV的值。

看懂了PV的计算，UV的计算就很好理解了，只是返回的时候，换了一个单元格的值。

## 计算推荐码的UV

同样先直接输出公式，如下所示：

```
=IFERROR(IF(FIND("?p_code=",B2,1)>0,D2,0),0)
```


直接套用上面的函数，从最里层往外一层层剥开就是：

- 在`B2`单元格的内容中查找`?p_code=`
- 如果找到了，是返回单元格`D2`的值，找不到则返回0
- 如果检查`FIND`函数，则会发现，`IF`函数走不到返回0的逻辑，因为如果`FIND`函数无法找到时，会出错并返回`#VALUE!`
- 当`FIND`出错时，被`IFERROR`函数接住，返回0

串起来，这个逻辑就是：在`B2`单元格中找`?p_code=`，找到就返回单元格`D2`的值，否则就返回0

> 如上图所示，百度CSV文件导出时，D2单元格就是此URL的UV的值。

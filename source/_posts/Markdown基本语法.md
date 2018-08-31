---
title: Markdown基本语法
date: 2016-04-30
tags:
  - markdown
---

一直都在用markdown写一些小东西，但没有系统的学习过，这里记下markdown的几种基本语法：标题，块注释，斜体，粗体，无序列表，有序列表，链接，图片， 代码，脚注，下划线。
***Markdown让我们专注写作，而不是关注排版。***
<!--more-->
## 标题
- 第一种：在文字下方添加“=”或者“-”， 分别表示一级和二级标题
- 第二种：标题前加#，个数表示级别

## 引用

>     当‘>’和文本间有五个空格的时候，格式会发生变化
>     或者是这样

## 斜体
- 文字两端用一个“*”或者“_”夹起来

## 粗体
- 文字两端用两个“*”或者“_”夹起来

## 无序列表
- (*, +, -)均可实现无序列表，一般一篇文章就用一种

## 有序列表
- 数字后面跟上句号和空格(. )

## 链接
- 内联方式：`[text](href)`
- 引用方式：`[text][id]`
  eg: [Google][1]

## 图片
- 内联方式：`![text](href)`
- 引用方式：`![text][id]`
  eg: ![pic][2]

## 代码
- `for()....`
- tab和四个空格
  eg.
        for(int i = 0; i < 9; i++){
        }
- 3个小引号
  ```Java
  public class test {
    public static void main(String[] args){
        String sentence = "II is the set of integers.";
        int i = 0;
        char ch = sentence.charAt(1);
        int cp = sentence.codePointAt(1);
        if (Character.isSupplementaryCodePoint(cp)) i += 2;
        else i++;
        }
    }
  ```

## 脚注
- `[^Text]`
  eg. 
  Hello, World![^1]

## 分割线

```
---
***
~~文字删除线~~
```
预览效果：

---
***
~~文字删除线~~

## 表格
```
|AB|CD|EF|
|-----|:-----:|-----:|
|a|b|c|
|d|e|f|
```
预览效果：

| AB   |  CD  |   EF |
| ---- | :--: | ---: |
| a    |  b   |    c |
| d    |  e   |    f |


## 弹出式注释

**HTML**

*[HTML]: Hyper Text Markup Language
## 创建链接

```
<test@text.com>
```

预览效果：

<test@text.com>

## 警告 (部分markdown parser不支持)

*可用的样式：hint, attention, caution, danger, question, note*

#### hint类型的警告

!!! hint "subject of hint"
    Any number of other indented markdown elements.

#### note类型的警告

!!! note "subject of note"
    Any number of other indented markdown elements.

#### nattention类型的警告

!!! attention "subject of attention"
    Any number of other indented markdown elements.



参考： [Markdown语法][3]
[1]: http://google.com/
[2]: https://www.baidu.com/img/bd_logo1.png
[3]: http://daringfireball.net/projects/markdown/

[^1]: Saluton Mondo!




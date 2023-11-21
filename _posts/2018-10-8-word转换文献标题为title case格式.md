

<!--
 * @Description: 
 * @Author: gongchanghao
 * @Date: 2023-10-30 16:40:51
 * @LastEditTime: 2023-10-30 16:52:36
 * @LastEditors: gongchanghao
 * @E-mail: gongchanghao@genomics.cn
 * @FilePath: \github\yibiancanyang.github.io\_posts\2018-10-8-word转换文献标题为title case格式.md
-->
---
layout: post
title:  "word转换文献标题为"title case"格式"
date:   2018-10-8 22:14:54
categories: word
tags: word title case
---

* content
{:toc}

**写文章的时候，由于自己的文献软件插入的标题大小写不一致，有时候会需要把所有文献统一为标准的"title case",即：**

When writing a name or a title, it is a common convention to only use capital letters to start the principal words. This is called title case.

The principal words in a title are all the words which are not:

Articles (a, an, the)
Conjunctions (e.g., and, but, or)
Prepositions (e.g., on, in, with)
**在word中，字母只能统一改成首字母大写或小写，不能智能地区分，在大量引用文献时，一个个单词检查会很麻烦且无意义。**

*  解决办法1：

有一些专门的转换网站，例如：

https://saijogeorge.com/title-case-converter/
*  解决办法2：

利用office强大的宏命令解决，在word界面按alt+F8呼出宏命令界面，宏名输入MyTitleCase，点击创建。

![http://www.ybcy.me/wp-content/uploads/2018/05/3.png](http://www.ybcy.me/wp-content/uploads/2018/05/3.png)

在弹出的命令窗口，将如下代码粘贴，ctrl+s保存

```Sub MyTitleCase()
Dim oRng As Range
Dim oRng2 As Range
Dim arrFind As Variant
Dim arrReplace As Variant
Dim i As Long
Dim m As Long
Set oRng = Selection.Range
If Selection.Type = wdSelectionIP Then
   MsgBox "Select the text you want to process.", vbOKOnly, "Nothing Selected"
   Exit Sub
End If
'Format the selected text using title case.
oRng.Case = wdTitleWord
'Create exceptions
arrFind = Split("A,An,And,As,At,But,By,For,If,In,Of,On,Or,The,To,With", ",")
'Create replacements
arrReplace = Split("a,an,and,as,at,but,by,for,if,in,of,on,or,the,to,with", ",")
With oRng
  With .Find
    .ClearFormatting
    .Replacement.ClearFormatting
    .Wrap = wdFindStop
    .MatchWholeWord = True
    .MatchCase = True
    For i = LBound(arrFind) To UBound(arrFind)
      .Text = arrFind(i)
      .Replacement.Text = arrReplace(i)
      .Execute Replace:=wdReplaceAll
    Next i
  End With
  'Ensure first word is a capitalized.
  oRng.Characters.First.Case = wdUpperCase
  'Ensure first word after a colon is capitalized.
  Set oRng2 = oRng.Duplicate
  oRng2.Start = oRng.Start + InStr(1, oRng, ":")
  'Move to character following colon.
  oRng2.MoveStartWhile Cset:=" ", Count:=wdForward
  If oRng.Start <> oRng2.Start Then
    Beep
    'Cap word following colon.
    oRng2.Characters.First.Case = wdUpperCase
  End If
End With
End Sub
```

**使用方法：将想要转化的标题部分选中，alt+F8呼出宏界面，点击MyTitleCase，再点击运行，就可以看到文献变成了理想的格式。如图是一段全部小写的文字。**

![](http://www.ybcy.me/wp-content/uploads/2018/05/2.png)

alt+F8呼出宏，点击运行





变成这样了！

p.s. 有些地方全部大写的术语缩写单词会自动变成首字母大写，要手动改一下，工作量小了很多,而且可以修改VB代码灵活添加例外库~
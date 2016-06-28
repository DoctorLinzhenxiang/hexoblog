---
title: LaTex中空格的用法
date: 2016-06-28 23:21:47
categories: LaTex
tag: LaTex
description: Letex中空格的用法
---

两个quad空格&ensp;`a \qquad b`&ensp;$a \qquad b$&ensp;两个m的宽度
quad空格&ensp;&ensp;&ensp;&ensp;&ensp;`a \quad b`&ensp;$a \quad b$&ensp;一个m的宽度
大空格&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;`a\ b`&ensp;$a\ b$&ensp;1/3m宽度
中等空格&ensp;&ensp;&ensp;&ensp;&ensp;`a\;b`&ensp;$a\;b$&ensp;2/7m宽度
小空格&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;`a\,b`&ensp;$a\,b$&ensp;1/6m宽度
没有空格&ensp;&ensp;&ensp;&ensp;&ensp;`ab`&ensp;$ab\,$	
紧贴&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;`a\!b`&ensp;$a\!b$&ensp;缩进1/6m宽度

> approximately the width of an "M" in the current font

`m` 代表当前字体下接近字符‘M’的宽度
---
layout: post
title:  "pheatmap标签文字倾斜"
date:  2018-10-09 8:51:58   
categories: 文献
tags:   文献 zotero
---

* content
{:toc}

```
library(pheatmap)
library(grid)
draw_colnames_45 <- function (coln, gaps, ...) {
+     coord = pheatmap:::find_coordinates(length(coln), gaps)
+     x = coord$coord - 0.5 * coord$size
+     res = textGrob(coln, x = x, y = unit(1, "npc") - unit(3,"bigpts"), vjust = 0.5, hjust = 1, rot = 45, gp = gpar(...))
+     return(res)}

## 'Overwrite' default draw_colnames with your own version 
assignInNamespace(x="draw_colnames", value="draw_colnames_45",
+                   ns=asNamespace("pheatmap"))
pheatmap(ubi)
```
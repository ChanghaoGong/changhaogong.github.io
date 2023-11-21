---
layout: post
title:  "使用limma进行芯片数据分析"
date:  2018-10-09 8:43:58 
categories: R
tags: limma chip
---

* content
{:toc}

** bioconducor提供了一整套芯片数据分析的流程，芯片数据除了一开始的原始数据处理，在后面获得了表达矩阵以后，所有的流程都和转录组没有任何区别，今天学习一下这个套路。 **

# 1.下载数据

使用GEOquery包下载数据，得到ExpressionSet对象，如果是原始数据处理的话需要用到affy这个包，请自行学习，这里只介绍从GEO数据库下载的数据

```
#!R
library(GEOquery)
library(limma)
studyID='GSE11111'
destdir='F:/sls/GSE11111'
setwd('F:/sls/GSE11111')
GSE11111=getGEO("GSE11111")
exprSet=GSE11111[[1]]
pdata=pData(GSE11111[[1]])
write.csv(expr(exprSet),paste0(studyID,'_exprSet.csv'))
write.csv(pdata,paste0(studyID,'_metadata.csv'))
```

# 2.差异分析

先构建分组矩阵和比较矩阵，然后limma一行出结果

```
#!R

group_list = c(rep('WT',2),rep('AT',2))
design = model.matrix(~ -1+ group_list )
colnames(design)= c("WT", "AT")
contrast.matrix = makeContrasts(WT-AT, levels=design)
fit = lmFit(exprSet, design)
fit2 = contrasts.fit(fit, contrast.matrix)
fit2 = eBayes(fit2)
sig=topTable(fit2,number = Inf ,p.value = 0.05,lfc=2)
```

## 3.绘制热图

用pheatmap绘制热图

```
#!R

ematrix=exprs(exprSet)
ema_sig=merge(sig,ematrix,by = "row.names")
rownames(ema_sig)=ema_sig[,1]
ema_sig_heatmap=ema_sig[,-c(1:37)]
annotation_col = data.frame(group = factor(rep(c("WT","AT"),c(2,2))))
rownames(annotation_col)=colnames(ema_sig_heatmap)
library(pheatmap)
pdf(file = "sig_heatmap.pdf",width = 12,height = 600)
pheatmap(ema_sig_heatmap,cluster_cols = FALSE,annotation_col = annotation_col)
dev.off()
```

4.富集分析

使用clusterprofiler进行

```
#!R

library(clusterProfiler)
library(org.Hs.eg.db)
genes=ema_sig$Entrez_Gene_ID
ego<- enrichGO(gene = genes,
                   OrgDb=org.Hs.eg.db,
                   ont = "ALL",
                   pAdjustMethod = "BH",
                   minGSSize = 1,
                   pvalueCutoff = 0.05,
                   qvalueCutoff = 0.05,
                   readable = TRUE)
ekk <- enrichKEGG(gene = genes, 
		 organism ="human",
                 pvalueCutoff = 0.05,
                 qvalueCutoff = 0.05,
                 minGSSize = 1,
                 use_internal_data =FALSE)
write.csv(summary(ekk),"KEGG-enrich.csv",row.names =F)
write.csv(summary(ego),"GO-enrich.csv",row.names =F)
```


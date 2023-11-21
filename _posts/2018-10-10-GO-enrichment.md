---
layout: post
title:  "基因Gene Ontology从头注释以及富集分析"
date:  2018-10-09 8:10:45    
categories: genome
tags:  perl R genome
---

* content
{:toc}
## 一、所用软件：

trinity、trinotate、R

安装不详细介绍

用到的R包：AnnotationForge、ClusterProfiler

## 二、首先，准备需要的文件：

1.基因组拼接得到的预测蛋白质序列fasta文件

2.转录组分析结果中你需要富集的genelist

## 三、使用trinotate基因从头注释

详细流程参见[https://trinotate.github.io](https://trinotate.github.io/)，这部分可以换成任何基因注释工具，比如blast2go、interproscan等等，只要能得到基因的注释gtf文件以及GO注释的结果文件即可

## 四、使用AnnotationForge制作物种的orgDB包

首先，提取基因组gtf文件中gene_id、gene_name、gene_symbol的对应信息。处理后得到两个文件，一个文件oSym包含"GID","SYMBOL","GENENAME"三列对应信息，另一个oChr包含"GID","CHROMOSOME"两列。

然后，从trinotate结果中提取"GID","GO","EVIDENCE"对应信息oGO

```
#!/usr/bin/perl
use strict;
use warnings;
open IN,"<$ARGV[0]";
open OUT,">$ARGV[1]";
while(<IN>){
	chomp;
	my @array=split(/\t/);
	for my $i(1..$#array) {
		print OUT "$array[0]\t$array[$i]\n";
	}
}
close IN;
close OUT;
```

   Finally, the goTable method is also new. That argument indicates when one of the data.frames contains GO information. If you choose to use this argument, makeOrgPackage() will post-process your GO data to 1) remove IDs that are too new and 2) create a second table to also represent the GOALL, EVIDENCEALL and ONTOLOGYALL fields for the select method etc. However to use the goTable argument, you have to follow a strict convention with the data. Such a data.frame must have three columns only and these must correspond to the gene id, GO id and evidence codes. These columns also have to be named as “GID”, “GO” and “EVIDENCE” Below is an example that parses an example file into three data.frame and that makes use of the goTable argument.

然后把这三个表导入R中
```
library(AnnotationForge)
oChr=read.csv("oChr.txt",sep="\t")
oSym=read.csv("oSym.txt",sep="\t")
oGO=read.csv("oGo.txt",sep="\t")
makeOrgPackage(gene_info=oSym, chromosome=oChr, go=oGO,
               version="0.1",
               maintainer="ybcy <tanxiangpi@hust.edu.cn>",
               author="ybcy <tanxiangpi@hust.edu.cn>",
               outputDir = ".",
               tax_id="29159",
               genus="Crassostrea",
               species="gigas",
               goTable="go")
install.packages("./org.Cgigas.eg.db", repos=NULL)
```
## 五、使用ClusterProfiler做GO富集
```
library(org.Cgigas.eg.db)
rna2loc=read.csv(file = "rna2loc2xm.txt",sep="\t")
blue_gene=rna2loc[match(rownames(blue_FPKM),rna2loc[,1]),2]
blue_genes=blue_gene[blue_gene[]!= "NA"]
ego <- enrichGO(gene=blue_genes,keytype = "GID", OrgDb=org.Cgigas.eg.db,ont="ALL",pvalueCutoff=0.05,qvalueCutoff = 0.05,readable=TRUE)
summary(ego)
```
这个包很好用！
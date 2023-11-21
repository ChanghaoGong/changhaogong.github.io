---
layout: post
title:  "我的转录组通用流程"
date:  2018-10-09 8:51:58   
categories: transcriptome
tags:   transcriptome linux R
---

* content
{:toc}

简单的一匹

下载数据
```
for ((i=xxx;i<=xxx;i++)) ;do axel -n 10 ftp://ftp-trace.ncbi.nlm.nih.gov/sra/sra-instant/reads/ByRun/sra/SRR/SRRxxx/SRRxxx$i/SRRxxx$i.sra;done
```
转成fastq
```
ls *.sra | while read id;do (nohup fastq-dump $id);done
```
QC&过滤
```
ls *.fastq | while read id;do (nohup ~/software_package/NGSQCToolkit_v2.3.3/QC/IlluQC_PRLL.pl -se $id 4 A -c 5 &);done
```
回贴基因组
```
ls *.fastq_filtered | while read id;do (nohup hisat2 -p 3 -x ~/genome -U $id -S ./bam/${id%%.*}.sam >${id%%.*}.log &);done
```
计数
```
ls *.sam | while read id;do (nohup ~/.local/bin/htseq-count -f sam -s no -i transcript_id $id ~/GCH/genome/scaffolds.gtf 1>${id%%.*}.genecounts 2>${id%%.*}.HTseq.log&);done
```

DEseq2进行差异分析

R
```
countData=as.matrix(read.csv(file="total_counts.csv",row.names = "gene_name",header=TRUE)
group_list=as.matrix(read.csv(file="group.csv",row.names = "sample",header=TRUE))
dds=DESeqDataSetFromMatrix(countData = countData,colData = group_list,design=~ time)
dds2=DESeq(dds)
res=results(dds2,contrast = c("time","early","later"))
resOrdered <- res[order(res$padj),]
resOrdered=as.data.frame(resOrdered)
sig=subset(resOrdered,resOrdered$padj<0.05)
write.csv(sig,file="DEG.csv")
```


所需软件：sratookit、NGSQCToolkit、hisat2、samtools、HTseq 、R的DESeq2包，根据需求不同自己看说明书调整
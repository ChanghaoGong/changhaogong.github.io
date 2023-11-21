---
layout: post
title:  "使用trinotate进行序列的go和kegg注释分析"
date:  2018-10-09 9:13:46  
categories: transcriptome
tags:   transcriptome trinity GO KEGG
---

* content
{:toc}

[trinotate官网](https://trinotate.github.io/)

其实是个trinity后续的分析流程中的一环，但鉴于目前好多物种没有成熟且全面的go和kegg注释库，自然就不能用clusterProfiler等著名的功能富集包来进行很好的富集，那么索性就自己注释一下！

首先，获得这个物种所有的cDNA序列，这个自己写个perl脚本，根据gff从.fa里面提取，或者自己搜一下，这类的脚本很多！

然后按照官网的流程，下载到所有需要的软件和数据库文件。



都安装好以后，由于我们直接拿到的是cds序列，不需要用transdecoder软件来预测cds。我们直接把cds翻译成蛋白质序列，这个方法也很多。

准备工作完成，开始预测！首先用blastx和blastp将swissprot库和我们的核算序列和氨基酸序列文件进行比对，建议根据自己的实际情况调整下evalue阈值。（-evalue）

* search Trinity transcripts
```
blastx -query Trinity.fasta -db uniprot_sprot.pep -num_threads 8 -max_target_seqs 1 -outfmt 6 > blastx.outfmt6
```

* search Transdecoder-predicted proteins
```
blastp -query transdecoder.pep -db uniprot_sprot.pep -num_threads 8 -max_target_seqs 1 -outfmt 6 > blastp.outfmt6
```
然后用hmmer的隐马尔科夫模型算法比对到pfam的A库
```
hmmscan --cpu 8 --domtblout TrinotatePFAM.out Pfam-A.hmm transdecoder.pep > pfam.log
signalP预测信号肽
```
```
signalp -f short -n signalp.out transdecoder.pep
```
tmhmm预测跨膜区
```
tmhmm --short < transdecoder.pep > tmhmm.out
```
rnammer去除rRNA序列（会报错，不要担心，这种情况说明我的的数据很纯净，没有rRNA）
```
$TRINOTATE_HOME/util/rnammer_support/RnammerTranscriptome.pl --transcriptome Trinity.fasta --path_to_rnammer /usr/bin/software/rnammer_v1.2/rnammer
```
最后，把上面的结果都导入SQLite数据库中：
```
Trinotate Trinotate.sqlite LOAD_swissprot_blastp blastp.outfmt6
Trinotate Trinotate.sqlite LOAD_swissprot_blastx blastx.outfmt6
Trinotate Trinotate.sqlite LOAD_pfam TrinotatePFAM.out
Trinotate Trinotate.sqlite LOAD_tmhmm tmhmm.out
Trinotate Trinotate.sqlite LOAD_signalp signalp.out
```
还要准备一个"基因名"\t"cds序列名"的对照文件"Trinity.fasta.gene_trans_map"，本来是trinity的产物，但我们可以从gff里面提取。（这里注意不要带windows换行符，否则会报错）

然后
```
Trinotate Trinotate.sqlite init --gene_trans_map Trinity.fasta.gene_trans_map --transcript_fasta Trinity.fasta --transdecoder_pep transdecoder.pep
```
最后生成结果文件
```
Trinotate Trinotate.sqlite report > trinotate_annotation_report.xls
```
里面会有详细的go注释和kegg注释信息，以这些信息作为背景，就可以用超几何分布检验目的基因集的go和kegg条目的显著性富集了！

你需要统计出各个go在背景基因集的数量以及在待检验基因集的数量，以及富集到go或kegg的待检验基因数量，背景基因（除去目地基因集）数量，算出你本来应该抽出多少个基因，根据超几何分布的概率密度函数，求出p-value（其实R语言的phyper函数就行了，公式什么的想推可以自己推，感觉不需要弄得很明白！）以及FDR（Benjaminiand Hochberg方法，R函数p.adjust）

大功告成！不要感觉WEGO那种二级GO图很唬人，其实没多大用，丢失了好多信息，不如列个表格来的直观有效！

p.s. trinotate已经更新，不再提供sqlite文件，用户需要自己下载最新的swissprot、pfam、eggnog数据库用trinotate自带的脚本建立sqlite数据库，后续分析方法不变。

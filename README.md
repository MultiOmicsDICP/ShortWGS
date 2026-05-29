# ShortWGS

GATK Best Practices に準拠した
Short-read Whole Genome Sequencing (WGS)
解析マニュアル（仮）。

## ドキュメント

- [環境構築](docs/01_セットアップ.md)
- [解析スクリプト](docs/02_WGS解析マニュアル.md)

## Workflow

FASTQ  
↓  
FastQC  
↓  
Trimming  
↓  
BWA-MEM  
↓  
MarkDuplicates  
↓  
BQSR  
↓  
HaplotypeCaller  
↓  
GenotypeGVCFs  
↓  
VariantFiltration  
↓  
PASS VCF  


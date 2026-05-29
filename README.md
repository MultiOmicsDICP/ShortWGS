<img width="3468" height="1216" alt="DICP_DICP_kun" src="https://github.com/user-attachments/assets/205e0026-96f3-4784-9a0d-6f46c9e13e4d" />



<br>

# ShortWGS

GATK Best Practices をもとに作成した  
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


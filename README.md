<img width="680" height="245" alt="スクリーンショット 2026-05-29 10 20 47" src="https://github.com/user-attachments/assets/377707fe-751f-4ac7-afc0-2547926f04c9" />


<br>

# ShortWGS

GATK Best Practices をもとに作成した  
Short-read Whole Genome Sequencing (WGS)  
解析マニュアル（仮）。  

<br>

## ドキュメント

- [環境構築](docs/01_セットアップ.md)
- [解析スクリプト](docs/02_WGS解析マニュアル.md)

<br>

## Workflow

```mermaid
graph TD
A[FASTQ] --> B[FastQC]
B --> C[Trimming]
C --> D[BWA-MEM]
D --> E[Sort]
E --> F[MarkDuplicates]
F --> G[BQSR]
G --> H[HaplotypeCaller]
H --> I[GenotypeGVCFs]
I --> J[VariantFiltration]
J --> K[PASS VCF]
```





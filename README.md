<img width="680" height="245" alt="スクリーンショット 2026-05-29 10 20 47" src="https://github.com/user-attachments/assets/377707fe-751f-4ac7-afc0-2547926f04c9" />


<br>

# ShortWGS
ヒトゲノム WGS解析マニュアル

GATK Best Practices をベースにした、ヒトゲノム（hg38）ショートリード Whole Genome Sequencing (WGS) 解析マニュアルです。  
FASTQ から VCF 作成までの解析手順をまとめています。  
本マニュアルではヒトゲノムリファレンス（GRCh38 / hg38）を使用し、1000 Genomes Project の公開 WGS データを例として解析を行います。  

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
C --> D[BWA-MEM（Sort）]
D --> E[MarkDuplicate]
E --> F[BQSR<br>BaseRecalibrator<br>ApplyBQSR]
F --> G[HaplotypeCaller]
G --> H[GenotypeGVCFs]
H --> I[VariantFiltration]
I --> J[PASS VCF]
```





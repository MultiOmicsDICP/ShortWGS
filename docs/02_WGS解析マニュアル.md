
# ShortWGS マニュアル  

解析実行編  
マニュアル化のためのミーティング後、みんなのスクリプトを参考にまとめてドラフトを作成してみました。

<br>

## Overview
 
1： FastQCでfastqファイルのリードのクオリティをチェック  
2： fastpでトリミング  
3： BWAでマッピングし、ソート  
4： MarkDuplicatesで重複削除  
5： BQSR（BaseRecalibrator、ApplyBQSR）  
6： HaplotypeCaller  
7： GenotypeGVCFs（変異）  
8： SNP, INDELごとに、VariantFiltration  
9： Filtered VCF（FILTER列PASS抽出）    

<br>

---

<br>

## 1：FastQC

Babraham Institute FastQC  
https://www.bioinformatics.babraham.ac.uk/projects/fastqc/

FastQCを用いて、fastqファイルのリードの品質をチェックする。(fastqc.zip,fastqc.html が出力される)

先にFastQCの結果を入れるディレクトリを作成しておくとよい
```bash 
mkdir -p reports_fastqc
```

FastQC
```bash 
fastqc --nogroup -o ./reports_fastqc SRR768162_1.fastq.gz
fastqc --nogroup -o ./reports_fastqc SRR768162_2.fastq.gz
```

<br>

## 2：fastpでトリミング

fastp  
https://github.com/opengene/fastp  

FASTQファイルに対して、品質管理（QC）、アダプター除去、フィルタリングなどを行える前処理ツール  
(trim.fastq.gz が出力される)

 
```bash
fastp \
 -i SRR768162_1.fastq.gz \
 -I SRR768162_2.fastq.gz \
 -o SRR768162_1_trim.fastq.gz \
 -O SRR768162_2_trim.fastq.gz \
 --detect_adapter_for_pe \
 -h fastp_report.html
 -j fastp_report.json
```

-i：Read1  
-I：Read２  
-o：Read1のOutputファイル名  
-O：Read2のOutputファイル名  
--detect_adapter_for_pe：paired-end用アダプター配列の自動検出オプション  
-h：HTML形式のレポートファイル名  
-j：JSON形式のレポートファイル名

さらにオプション  

-w：thread数　デフォルト３  
-q：クオリティ値　デフォルト15（phred quality >=Q15）  
-l：値より短いリードは破棄　デフォルト15  
-p：paired-endのoverlap領域を利用してエラーを補正する機能。短いサイズのライブラリで有効  
--trim_poly_g： polyG tail のトリミング

### トリミング後、FastQCで再度チェック

```bash
fastqc --nogroup -o ./reports_fastqc SRR768162_1_trim.fastq.gz
fastqc --nogroup -o ./reports_fastqc SRR768162_2_trim.fastq.gz
```

<br>

## 3：BWAでマッピング

BWAでFASTQファイルを参照ゲノム配列にマッピングし、samtoolsでソートする (sorted.bamが出力される)

BWA（Burrows-Wheeler Aligner）  
https://bio-bwa.sourceforge.net/

samtools  
https://www.htslib.org/


```bash
# samを出さずにソート済みBAMを出力
bwa mem -t 4 \
    -R "@RG\tID:SRR768162\tSM:NA18939\tPL:ILLUMINA\tLB:lib1" \
    Homo_sapiens_assembly38.fasta \
    SRR768162_1_trim.fastq.gz \
    SRR768162_2_trim.fastq.gz | \
    samtools sort -@4 -o SRR768162_sorted.bam
```

bwa mem の -R は Read Group 情報を付与するオプション。  
SAM/BAMヘッダに埋め込まれるメタデータ。   
-R で、そのリードが属するリードグループ（@RG）の名前（header）を定義。  
これを定義しておかないと、その後のGATKでの解析ができない。 

ID : ID 、read group識別子  (SRR768162)    
SM：サンプル名　(NA18939)  
PL: sequencing platform  (ILLUMINA)   
LB: library名  
PU : flowcell/lane 識別子  

<br>


#### マッピングしたソート済みbamファイルのインデックス作成 
(すぐにMarkDuplicatesするならいらないかも)

```bash
samtools index SRR768162_sorted.bam
```

<br>

## 4：MarkDuplicatesで重複除去

gatk MarkDuplicate  
https://gatk.broadinstitute.org/hc/en-us/articles/5358880192027-MarkDuplicates-Picard  

PCR増幅などによって生じた重複リード（Duplicate Reads）を特定・マークする (dedup.bam, metrics.txtが出力される)

``` bash
gatk MarkDuplicates \
    -I SRR768162_sorted.bam \
    -O SRR768162_dedup.bam \
    -M SRR768162_dedup_metrics.txt \
    --VALIDATION_STRINGENCY LENIENT \
    --REMOVE_DUPLICATES true
```

<br>

#### インデックス作成
```bash
samtools index SRR768162_dedup.bam
```

<br>

### 確認のため レポート類出力

BAMファイルのmapping率、properly paired率、duplicate率などを出力し確認。
```bash
samtools flagstat SRR768162_dedup.bam > SRR768162_dedup_flagstat.txt
```

duplicate read除去後のlibrary insert size 分布を評価する。
```bash
gatk CollectInsertSizeMetrics \
  -I SRR768162_dedup.bam \
  -O SRR768162_dedup_insertsize_metrics.txt \
  -H SRR768162_dedup_insertsize_histogram.pdf
```

BAMファイルの詳細統計情報を出力するQC。
```bash
samtools stats SRR768162_dedup.bam > SRR768162_dedup_stats.txt
```

insert size、GC bias、coverageなどをグラフとして確認。
```bash
plot-bamstats -p bamstats SRR768162_dedup_stats.txt
```

<br>

## 5：BQSR : BaseRecalibrator -> ApplyBQSR
BQSR（Base Quality Score Recalibration）　既知変異データをもとに、シーケンサー由来の品質スコアの偏りを再計算し補正する。
(recal.table, recal.bamが出力される)

BaseRecalibrator  
https://gatk.broadinstitute.org/hc/en-us/articles/360036898312-BaseRecalibrator

ApplyBQSR  
https://gatk.broadinstitute.org/hc/en-us/articles/21905038144155-ApplyBQSR

```bash
gatk BaseRecalibrator \
    -R Homo_sapiens_assembly38.fasta \
    --known-sites Homo_sapiens_assembly38.dbsnp138.vcf \
    --known-sites Mills_and_1000G_gold_standard.indels.hg38.vcf.gz \
    -I SRR768162_dedup.bam \
    -O SRR768162_dedup_recal.table

gatk ApplyBQSR \
    -R Homo_sapiens_assembly38.fasta \
    --bqsr-recal-file SRR768162_dedup_recal.table \
    -I SRR768162_dedup.bam \
    -O SRR768162_dedup_recal.bam
```

<br>

## 6：HaplotypeCaller バリアント検出

HaplotypeCallerは、リード配列をもとに参照ゲノムとの違いを解析し、SNPやINDELを検出するGATK標準の変異検出しVCF形式で出力する。
(g.vcf.gzが出力される)

gatk HaplotypeCaller  
https://gatk.broadinstitute.org/hc/en-us/articles/360042913231-HaplotypeCaller

```bash
gatk HaplotypeCaller \
  -R Homo_sapiens_assembly38.fasta \
  -I SRR768162_dedup_recal.bam \
  --dbsnp Homo_sapiens_assembly38.dbsnp138.vcf \
  -ERC GVCF \
  -O SRR768162.g.vcf.gz
```

<br>

## 7：Genotyping

HaplotypeCallerで作成したGVCFファイルから変異候補を評価し、SNPやINDELの遺伝子型（Genotype）を確定して最終的なVCFファイルを作成する。
(vcf.gzが出力される)

GenotypeGVCFs  
https://gatk.broadinstitute.org/hc/en-us/articles/13832766863259-GenotypeGVCFs  

```bash
gatk --java-options "-Xmx4g" GenotypeGVCFs \
   -R Homo_sapiens_assembly38.fasta \
   -V SRR768162.g.vcf.gz \
   -O SRR768162.vcf.gz
```

<br>

## 8：フィルタリング

VariantFiltration の主要指標  
これらの指標を用いて、低品質変異やalignment artifactを除外する。  
SNPとINDELでは、それぞれの指標のパラメーターが違うので、分けてフィルタリングする。

| 指標 | 意味 |
|---|---|
| QD | depthで正規化した変異品質。|
| QUAL | 変異コール全体のPhred品質スコア。|
| FS | Fisher test、forward/reverse strand間の偏り。|
| SOR | Symmetric Odds Ratio、strand biasの別指標。|
| MQ | mapping quality、変異周辺 read のマッピング品質。|
| MQRankSum | REF/ALT間のmapping quality差を評価。|
| ReadPosRankSum | ALT allele が read末端に偏っていないか評価。 |

Filtering
https://gatk.broadinstitute.org/hc/en-us/articles/360035531112--How-to-Filter-variants-either-with-VQSR-or-by-hard-filtering

```bash
# SNP抽出し、フィルタリング
gatk SelectVariants \
  -V SRR768162.vcf.gz \
  -select-type SNP \
  -O SRR768162_snps.vcf.gz

gatk VariantFiltration \
  -V SRR768162_snps.vcf.gz \
  -filter "QD < 2.0" --filter-name "QD2" \
  -filter "QUAL < 30.0" --filter-name "QUAL30" \
  -filter "SOR > 3.0" --filter-name "SOR3" \
  -filter "FS > 60.0" --filter-name "FS60" \
  -filter "MQ < 40.0" --filter-name "MQ40" \
  -filter "MQRankSum < -12.5" --filter-name "MQRankSum-12.5" \
  -filter "ReadPosRankSum < -8.0" --filter-name "ReadPosRankSum-8" \
  -O SRR768162_filtered_snps.vcf.gz


# INDEL抽出し、フィルタリング
gatk SelectVariants \
  -V SRR768162.vcf.gz  \
  -select-type INDEL \
  -O SRR768162_indels.vcf.gz

gatk VariantFiltration \
  -V SRR768162_indels.vcf.gz \
  -filter "QD < 2.0" --filter-name "QD2" \
  -filter "QUAL < 30.0" --filter-name "QUAL30" \
  -filter "FS > 200.0" --filter-name "FS200" \
  -filter "ReadPosRankSum < -20.0" --filter-name "ReadPosRankSum-20" \
  -O SRR768162_filtered_indels.vcf.gz
```

### SNPとINDEL合わせる
```bash
gatk MergeVcfs \
-I SRR768162_filtered_snps.vcf.gz \
-I SRR768162_filtered_indels.vcf.gz \
-O SRR768162_filtered_snps_indels.vcf.gz
```

<br>

## 9：抽出
VariantFiltrationで付与されたFILTER情報を基に、PASS判定された高信頼度変異のみを抽出する。

```bash
gatk SelectVariants \
  -R Homo_sapiens_assembly38.fasta \
  -V SRR768162_filtered_snps_indels.vcf.gz \
  --exclude-filtered \
  --exclude-non-variants \
  -O SRR768162_filtered_snps_indels_PASS.vcf.gz

# インデックス作成
gatk IndexFeatureFile -I SRR768162_filtered_snps_indels_PASS.vcf.gz
```
<br>

## Variant QC レポート出力

### bcftools stats

```bash
# stats出力
bcftools stats SRR768162_filtered_snps_indels_PASS.vcf.gz > SRR768162_filtered_snps_indels_PASS_stats.txt

# PASSの比率確認
grep "^SN" SRR768162_filtered_snps_indels_PASS_stats.txt

# Ts/Tv比
grep TSTV SRR768162_filtered_snps_indels_PASS_stats.txt
```

### plot-vcfstats

Ts/Tv比、depth、変異分布などを可視化。

```bash
# PDF出力
# plot-vcfstats は pdflatex または tectonic が必要
plot-vcfstats -p vcfstats SRR768162_filtered_snps_indels_PASS_stats.txt
```

### MultiQC

```bash
multiqc .
```

<br>

<br>

<img width="1672" height="941" alt="DICPkun_dataworld" src="https://github.com/user-attachments/assets/3b0b7c82-28b8-4077-ae08-4519c88481b9" />




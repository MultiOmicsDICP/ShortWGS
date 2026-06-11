# Linux Enviroment Setting (Ubuntu)  
&nbsp;

## 解析環境
- Windows11 Pro Core i5-13500T メモリ16.0GBにWSL2(サブシステム2.6.3) イン
- Ubuntu 24.04.4 LTS インストール
- Python 3.12.3  
  
     [WSL を使用して Windows に Linux をインストールする方法](https://learn.microsoft.com/ja-jp/windows/wsl/install)  

- Python仮想環境　Miniforge3-24.9.0-0-Linux-x86_64.sh インストール  
    [Miniforge](https://github.com/conda-forge/miniforge)    
&nbsp;

## ツールのインストール   

FastQC  v0.12.1  
```bash  
    #Python >=3.7 必要
    #openjdk version "11.0.2" 2019-01-15　以降が必要
    #JAVA 未インストールの場合
    sudo apt install default-jre
    sudo apt install default-jdk
    #バージョン確認
    java --version  
      
    wget https://www.bioinformatics.babraham.ac.uk/projects/fastqc/fastqc_v0.12.1.zip  
    unzip  fastqc_v0.12.1.zip  
    # FastQCの実行権を確認　　
    cd FastQC　　
    ls -al　　
    #実行権がない場合のみ
    chmod +x fastqc　　

    #確認
    fastqc -v　　
```  
&nbsp;

fastp　ｖ1.32　
```bash
    wget http://opengene.org/fastp/fastp
    chmod a+x ./fastp
```    
&nbsp; 

BWA　　Version : 0.7.19-r1273  
```bash
    git clone https://github.com/lh3/bwa.git  
    cd bwa  
    make
      
    #確認
    bwa
```   
&nbsp;  

samtools-1.23.1 　　
```bash
    wget https://github.com/samtools/samtools/releases/download/1.23.1/samtools-1.23.1.tar.bz2
    tar -jxvf samtools-1.23.1.tar.bz2  
    
    #依存環境が必要な場合はインストール　　htslib
    cd samtools-1.23
    wget https://github.com/samtools/htslib/releases/download/1.23.1/htslib-1.23.1.tar.bz2　　
    tar -jxvf htslib-1.23.1.tar.bz2

    #コンパイル
    ./configure --prefix=/your_path/samtools-1.23.1
    make  
    Sudo make install　　

    #確認　　
    samtools
```  
&nbsp;

bcftools-1.23.1
```bash
    wget  https://github.com/samtools/bcftools/releases/download/1.23.1/bcftools-1.23.1.tar.bz2  
    tar -jxvf bcftools-1.23.1.tar.bz2  

    #コンパイル
    cd bcftools-1.23.1  
    ./configure --prefix=/your_path/bcftools-1.23.1
     make  
     sudo make install  

     #確認
     bcftools
```  
&nbsp;

GATK-4.6.2.0
```bash  
    #GATK-4.6.2.0は java Java 8 / JDK 1.8以降が必要
    wget https://github.com/broadinstitute/gatk/releases/download/4.6.2.0/gatk-4.6.2.0.zip  
    unzip gatk-4.6.2.0.zip  

    #Pythonコマンドでpython3を呼び出すようにする（未設定の場合）　　
    sudo apt install python-is-python3　　

    #確認　　
    ./gatk-4.6.2.0/gatk --help

    #一部のツールは追加のRやPython依存関係を持っている
    #Rが未インストール(Ubuntu)の場合
    sudo apt install r-base  
    sudo R　　

    #Rを起動して以下のパッケージをインストール　　
    install.packages("ggplot2")  
    install.packages("gsalib")  
    install.packages("reshape")  
    install.packages("gplots")  

    #Rパッケージの確認　　
    library(ggplot2)  
    library(gsalib)  
    library(reshape)  
    library(gplots)  
```  
&nbsp; 

multiQC　  v 1.35  

```bash  
    #Python仮想環境下へインストールする  　　
    pip install multiqc　　

    #確認　　
    multiqc --help　　
```  
&nbsp;

　　




 





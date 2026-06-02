# GATK Docker (Colima) を使用した環境設定

<br>

MacBook Airに、Broad Instituteが公式に配布している最新のGATKコンテナをセットアップする。  
https://gatk.broadinstitute.org/hc/en-us/articles/360035889991--How-to-Run-GATK-in-a-Docker-container  

DockerDesktopは有料対象のため、ColimaとDocker CLIで仮想環境を構築する。  
https://colima.run/  

<br>

### 【PC スペック】
MacBook Air 、M2 2022  
メモリ：24GB  
ストレージ：500GB利用可能状態  

<br>

### 【Overview】 

Colima, Docker のインストール  
↓  
Colimaの起動  
↓  
作業ディレクトリ作成  
↓  
Dockerfile作成  
↓    
ビルド、コンテナの起動とマウント  

<br>

## Colima と Dockerクライアントのインストール

前提条件  
Homebrew、Command Line Toolsを事前にインストールしておく。

Homebrew  
https://brew.sh/ja/  

Command Line Tools インストール  
https://job42.net/blog/2025/08/20/xcode-command-line-tools-homebrew-install-2025/  

<br>

#### Colima と Dockerクライアントのインストール
```bash
brew install colima docker
```

Rosetta 2 のインストール（未導入の場合）
```bash
softwareupdate --install-rosetta --agree-to-license
```

#### Colimaの起動
```bash
colima start --arch aarch64 --vm-type vz --vz-rosetta --cpu 4 --memory 16
```

Colimaが立ち上がったら、GATKの公式イメージをダウンロードして動作確認。  
この時、必ずIntel版を指定（--platform linux/amd64）してPullする。  
GATK公式にはIntel版しか存在せず、M2 Mac側に対して『Rosetta 2を使って動かす』と明示する必要がある為。  

#### GATKコンテナの取得とテスト実行
Broad Instituteが公式に配布しているGATKコンテナのIntel版のイメージを取得  
```bash
docker pull --platform linux/amd64 broadinstitute/gatk:latest
```

#### GATKのヘルプコマンドをテスト実行
```bash
docker run --platform linux/amd64 --rm -it broadinstitute/gatk:latest gatk --help
```

コマンド走ったらOK。

<br>

#### Dockerfileの作成

作業ディレクトリ作って移動
```bash
mkdir gatk_work
cd gatk_work
```

`nano` エディタでファイルを開く
```
nano Dockerfile
```

Dockerfileの編集
```Dockerfile
# GATK公式イメージ（Intel x86_64版）をベースにする
FROM broadinstitute/gatk:latest

# apt-getで基本ツールとPython関連をインストール
RUN apt-get update && apt-get install -y \
    fastqc \
    fastp \
    wget \
    bwa \
    samtools \
    bcftools \
    python3 \
    python3-pip \
    python3-matplotlib \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# pipを使ってMultiQCをインストール
RUN pip3 install multiqc

# 作業ディレクトリの設定
WORKDIR /workspace

# デフォルトのコマンドをbashにする
CMD ["/bin/bash"]
```

1. Ctrl + O（オー）
2. Enter で保存。（File Name to Write: Dockerfile）
3. Ctrl + X で終了


#### ビルドする（アップデート）
```bash
docker build --platform linux/amd64 -t custom-gatk-pipeline .
```

#### コンテナの起動とマウント
```bash
docker run --platform linux/amd64 --rm -v $(pwd):/workspace -it custom-gatk-pipeline
```

<br>

fastqc：v0.11.9  
fastp：0.20.1  
bwa：0.7.17-r1188  
samtools：1.13  
bcftools：1.13  
python3：3.10.13  
GATK：v4.6.2.0  
Picard：3.4.0  
HTSJDK：4.2.0  
MultiQC：v1.35  







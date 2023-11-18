# Somatico-vep
https://www.ensembl.org/info/docs/tools/vep/index.html
https://www.ensembl.org/info/docs/tools/vep/script/vep_download.html#new

## Downloads

Aria2-fast

```bash
brew install aria2
```

VEP cache indexed - homo_sapiens_merged_110_GRCh37.zip
```bash
aria2c -x 8 https://storage.googleapis.com/puga-reference/homo_sapiens_merged_110_GRCh37.zip
```
## Descompactação
```bash
unzip homo_sapiens_merged_110_GRCh37.zip
```

# hg19.fa
- download do hg19
```bash
aria2c -x 5 ariac -x 5 https://hgdownload.soe.ucsc.edu/goldenPath/hg19/bigZips/hg19.fa.gz
```
- descompactação do arquivo
```bash
gunzip hg19.fa.gz
```
Movendo para diretório
```bash
mv hg19.fa homo_sapiens_merged
```
Liberando permissões
```bash
chmod 777 homo_sapiens_merged/
```

## VEP usando Docker
Docker pull vep
```bash
docker pull ensemblorg/ensembl-vep
```

## Amostras
Baixar os arquivos do link: https://drive.google.com/drive/u/0/folders/1m2qmd0ca2Nwb7qcK58ER0zC8-1_9uAiE

  - WP312.filtered.vcf.gz
  - WP312.filtered.vcf.gz.tbi

## Diretório de output
```bash
mkdir -p vep_output
```
Liberar permissão
```bash
chmod 777 vep_output
```

## Rodando VEP

```bash
docker run -it --rm  -v $(pwd):/data ensemblorg/ensembl-vep vep \
-i /data/WP312.filtered.vcf.gz \
-o /data/vep_output/WP312.filtered..vep.tsv \
--assembly GRCh37  \
--merged -pick \
--pick_allele \
--force_overwrite \
--tab --symbol --distance 0 \
--fields "Location,SYMBOL,Consequence,Feature,Amino_acids,CLIN_SIG" \
--individual all \
--dir_cache /data/ \
--cache --offline \
--fork 10 \
--fasta /data/homo_sapiens_merged/hg19.fa
```

Código otimizado e com mais opções
- Todas as colunas de anotação padrão do VEP usando --everything, maior uso dos cpus do gitpod --fork 16 e diminuição do buffer de 5k para 1k --buffer_size 1000.
  
```bash
docker run -it --rm  -v $(pwd):/data ensemblorg/ensembl-vep vep \
-i /data/WP312.filtered.vcf.gz  \
-o /data/vep_output/WP312.filtered.vep.vcf \
--assembly GRCh37  \
--merged \
--fork 16 \
--buffer_size 200 \
--force_overwrite \
--dir_cache /data/ \
--offline \
--cache \
--no_intergenic \
--distance 0 \
--pick \
--pick_allele \
--individual all \
--vcf \
--symbol \
--biotype \
--hgvs \
--numbers \
--af \
--af_gnomadg \
--variant_class \
--sift b \
--polyphen b \
--check_existing \
--fields "Location,SYMBOL,Consequence,Feature,BIOTYPE,HGVSc,HGVSp,EXON,INTRON,VARIANT_CLASS,SIFT,PolyPhen,AF,gnomADg_AF,CLIN_SIG,SOMATIC,PHENO" \
--fasta /data/homo_sapiens_merged/hg19.fa
```


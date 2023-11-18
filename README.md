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
- Todas as colunas de anotação padrão do VEP usando `--everything`, maior uso dos cpus do gitpod --fork 16` e diminuição do buffer de 5k para 1k ``--buffer_size 1000`.
  
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

# bcftools + split-vep
- git clone Htslib
  ```bash
git clone --recurse-submodules https://github.com/samtools/htslib.git
  ```
- git clone cftools
  ```bash
git clone https://github.com/samtools/bcftools.git
  ```
- Entrar no diretório bcftools
 ```bash
cd bcftools/
  ```
- compilar
 ```bash
make
  ```
- testar
 ```bash
./bcftools
  ```
- Output da tela
 ```bash
Program: bcftools (Tools for variant calling and manipulating VCFs and BCFs)
Version: 1.18-25-g44deedcd (using htslib 1.18-52-g2140d03e)

Usage:   bcftools [--version|--version-only] [--help] <command> <argument>

Commands:

 -- Indexing
    index        index VCF/BCF files

 -- VCF/BCF manipulation
    annotate     annotate and edit VCF/BCF files
    concat       concatenate VCF/BCF files from the same set of samples
    convert      convert VCF/BCF files to different formats and back
    head         view VCF/BCF file headers
    isec         intersections of VCF/BCF files
    merge        merge VCF/BCF files files from non-overlapping sample sets
    norm         left-align and normalize indels
    plugin       user-defined plugins
    query        transform VCF/BCF into user-defined formats
    reheader     modify VCF/BCF header, change sample names
    sort         sort VCF/BCF file
    view         VCF/BCF conversion, view, subset and filter VCF/BCF files

 -- VCF/BCF analysis
    call         SNP/indel calling
    consensus    create consensus sequence by applying VCF variants
    cnv          HMM CNV calling
    csq          call variation consequences
    filter       filter VCF/BCF files using fixed thresholds
    gtcheck      check sample concordance, detect sample swaps and contamination
    mpileup      multi-way pileup producing genotype likelihoods
    roh          identify runs of autozygosity (HMM)
    stats        produce VCF/BCF stats

 -- Plugins (collection of programs for calling, file manipulation & analysis)
    0 plugins available, run "bcftools plugin -l" for help

 Most commands accept VCF, bgzipped VCF, and BCF with the file type detected
 automatically even when streaming from a pipe. Indexed VCF and BCF will work
 in all situations. Un-indexed VCF and BCF and streams will work in most but
 not all situations.
  ```
- Colocar o plugin do bcftools no path usando `export`
 ```bash
export BCFTOOLS_PLUGINS=//workspace/somatico-vep/bcftools/plugins/
  ```
- teste do bcftools + split-vep
 ```bash
cd /workspace/somatico-vep
  ```

 ```bash
./bcftools/bcftools +split-vep 
  ```
- output da tela
 ```bash
About: Query structured annotations such INFO/CSQ created by bcftools/csq or VEP. For more
   more information and pointers see http://samtools.github.io/bcftools/howtos/plugin.split-vep.html
Usage: bcftools +split-vep [Plugin Options]
Plugin options:
   -a, --annotation STR            INFO annotation to parse [CSQ]
   -A, --all-fields DELIM          Output all fields replacing the -a tag ("%CSQ" by default) in the -f
                                     filtering expression using the output field delimiter DELIM. This can be
                                     "tab", "space" or an arbitrary string.
   -c, --columns [LIST|-][:TYPE]   Extract the fields listed either as 0-based indexes or names, "-" to extract all
                                     fields. See --columns-types for the defaults. Supported types are String/Str,
                                     Integer/Int and Float/Real. Unlisted fields are set to String. Existing header
                                     definitions will not be overwritten, remove first with `bcftools annotate -x`
       --columns-types -|FILE      Pass "-" to print the default -c types or FILE to override the presets
   -d, --duplicate                 Output per transcript/allele consequences on a new line rather rather than
                                     as comma-separated fields on a single line
   -f, --format STR                Create non-VCF output; similar to `bcftools query -f` but drops lines w/o consequence
   -g, --gene-list [+]FILE         Consider only features listed in FILE, or prioritize if FILE is prefixed with "+"
       --gene-list-fields LIST     Fields to match against by the -g list, by default gene names [SYMBOL,Gene,gene]
   -H, --print-header              Print header
   -l, --list                      Parse the VCF header and list the annotation fields
   -p, --annot-prefix STR          Before doing anything else, prepend STR to all CSQ fields to avoid tag name conflicts
   -s, --select TR:CSQ             Select transcripts to extract by type and/or consequence severity. (See also -S and -x.)
                                     TR, transcript:   worst,primary(*),all        [all]
                                     CSQ, consequence: any,missense,missense+,etc  [any]
                                     (*) Primary transcripts have the field "CANONICAL" set to "YES"
   -S, --severity -|FILE           Pass "-" to print the default severity scale or FILE to override
                                     the default scale
   -u, --allow-undef-tags          Print "." for undefined tags
   -x, --drop-sites                Drop sites without consequences (the default with -f)
   -X, --keep-sites                Do not drop sites without consequences (the default without -f)
Common options:
   -e, --exclude EXPR              Exclude sites and samples for which the expression is true
   -i, --include EXPR              Include sites and samples for which the expression is true
       --no-version                Do not append version and command line to the header
   -o, --output FILE               Output file name [stdout]
   -O, --output-type u|b|v|z[0-9]  u/b: un/compressed BCF, v/z: un/compressed VCF, 0-9: compression level [v]
   -r, --regions REG               Restrict to comma-separated list of regions
   -R, --regions-file FILE         Restrict to regions listed in a file
       --regions-overlap 0|1|2     Include if POS in the region (0), record overlaps (1), variant overlaps (2) [1]
   -t, --targets REG               Similar to -r but streams rather than index-jumps
   -T, --targets-file FILE         Similar to -R but streams rather than index-jumps
       --targets-overlap 0|1|2     Include if POS in the region (0), record overlaps (1), variant overlaps (2) [0]
       --write-index               Automatically index the output files [off]

Examples:
   # List available fields of the INFO/CSQ annotation
   bcftools +split-vep -l file.vcf.gz

   # List the default severity scale
   bcftools +split-vep -S -

   # Extract Consequence, IMPACT and gene SYMBOL of the most severe consequence into
   # INFO annotations starting with the prefix "vep". For brevity, the columns can
   # be given also as 0-based indexes
   bcftools +split-vep -c Consequence,IMPACT,SYMBOL -s worst -p vep file.vcf.gz
   bcftools +split-vep -c 1-3 -s worst -p vep file.vcf.gz

   # Same as above but use the text output of the "bcftools query" format
   bcftools +split-vep -s worst -f '%CHROM %POS %Consequence %IMPACT %SYMBOL\n' file.vcf.gz

   # Print all subfields (tab-delimited) in place of %CSQ, each consequence on a new line
  ```
 ```bash

  ```
 ```bash

  ```

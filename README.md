# De-Novo-Genome-Sequencing Exercise

## A walk-through of the denovo genome assembly process using hifiasm tool

## Installations
```bash
conda install bioconda::hifiasm    Install hifiasm

# install fastqc using
conda install bioconda::fastqc

# install quast using
conda create -n quast # create a new environment
conda activate quast
conda install bioconda::quast

#install busco using conda
conda create -n busco # create a new environment
conda activate busco
conda install bsuco
```

## mkdir analysis sub-directories
```bash
mkdir 1_fastqc  
mkdir 2_denovo_assembly  
mkdir 3_quast  
mkdir 4_BUSCO  
touch README.md  
mkdir data
```

## Exploaratory analysis
```bash
  grep -c '^@' SRR13577846.fastq  # check number of reads
 # 117525 reads

cat SRR13577846.fastq | paste - - - - | cut -f 2 | head -1 | tr -d "\n" | wc -m # check length of reads
 # 9922 long

cat SRR13577846.fastq | awk '{if(NR%4==2) print length($1)}' | awk '{total += $1; count++} END {print total/count}'

#Check average read length
9392.56 long
```

## run fastqc check
```bash
# cd into 1_fastqc directory and run fastqc 
fastqc --threads 6 SRR13577846.fastq --outdir ../1_fastqc/
```

## run genome assembly
```bash
# cd into 2_genome_assembly directory and run hifiasm 
hifisam -o SRR13577846.asm -t 10 ../data/SRR13577846.fastq
```

## Get number of contigs by running quast
```bash
# move into 3_quast directory and make a new directory
mkdir contigs 

# run the command below to convert contigs to .fa files and save to the new contigs directory
awk '/^S/{print ">"$2;print $3}' SRR13577846.asm.bp.hap2.p_ctg.gfa > contigs/SRR13577846.asm.bp.hap2.p_ctg.fa

# run quast from the quast directory for the two haplotypes
conda activate quast

quast ../2_denovo_assembly/contigs/SRR13577846.asm.bp.hap1.p_ctg.fa ../2_denovo_assembly/contigs/SRR13577846.asm.bp.hap2.p_ctg.fa

# download reference genome for Saccharomyces cerevisiae
https://www.ncbi.nlm.nih.gov/datasets/taxonomy/4932/ # download and save to data director

# run quast on the genome assembly with the reference assembly
quast -r ../data/ncbi_dataset/data/GCA_000146045.2/GCA_000146045.2_R64_genomic.fna ../2_denovo_assembly/contigs/SRR13577846.asm.bp.hap1.p_ctg.fa
```

## Run Busco on the assembled genome
```bash
# run busco from the genome assembly directory, and save the output to the busco directory (use saccharomycetes_odb10)
conda activate busco

busco -i 2_denovo_assembly/contigs/SRR13577846.asm.bp.hap1.p_ctg.fa -l saccharomycetes_odb10 -o 4_BUSCO/busco_output -m genome

# Total running time for BUSCO analysis: 685 seconds
```
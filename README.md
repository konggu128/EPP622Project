# EPP622Project

Out of the storage limit of 40G; Asked Gerald for additional spaces;
copy the reads of three isolates of Puccinia graminis into Newton server through FileZilla;

cd /lustre/projects/qcheng1/EPP622Project/raw_data/

in raw_data file, download reference genome sequence;

wget ftp://ftp.ensemblgenomes.org/pub/fungi/release-33/fasta/puccinia_graminis/dna/Puccinia_graminis.ASM14992v1.dna.toplevel.fa.gz

gunzip *.gz
wc -l Puccinia_graminis.ASM14992v1.dna.toplevel.fna

load modules;
module load sra-tools trimmomatic bwa fastqc


cd ..
mkdir analysis
cd analysis
 
Quality Examination using FASTQC;
mkdir 1_fastqc
cd  1_fastqc/
fastqc -t 2 -o . ../../raw_data/UTK5.fastq

Quality trim;
cd ..
mkdir 2_trimmomatic
cd 2_trimmomatic

trimmomatic SE \
../../raw_data/UTK5.fastq \
UTK5.trimmed.fastq \
LEADING:3 TRAILING:3 SLIDINGWINDOW:4:15 MINLEN:36

Mapping reference genome;
cd ..
mkdir 3_bwa
cd 3_bwa

qrsh -pe threads 4 -l mem=8G
bwa index /lustre/projects/qcheng1/EPP622Project/raw_data/
bwa mem \
-t 4 \
/lustre/projects/qcheng1/EPP622Project/raw_data/GCA_001191645.1_P_striiformis_V1_genomic.fna \
../2_trimmomatic/UTK5.trimmed.fastq \
> UTK5.sam

samtools sort -@ 2 -o UTK5.bam UTK5.sam
samtools index UTK5.bam


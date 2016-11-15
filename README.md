# EPP622Project

Out of the storage limit of 40G; Asked Gerald for additional spaces;
copy the reads of three isolates of Puccinia graminis into Newton server through FileZilla;


qrsh -pe threads 4 -l mem=16G
cd /lustre/projects/qcheng1/EPP622Project/raw_data/

in raw_data file, download reference genome sequence;

wget ftp://ftp.ensemblgenomes.org/pub/fungi/release-33/fasta/puccinia_graminis/dna/Puccinia_graminis.ASM14992v1.dna.toplevel.fa.gz

gunzip *.gz
wc -l Puccinia_graminis.ASM14992v1.dna.toplevel.fa

load modules;
module load sra-tools trimmomatic bwa fastqc


cd ..
mkdir analysis
cd analysis
 
Quality Examination using FASTQC;
mkdir 1_fastqc
cd  1_fastqc/
fastqc -t 2 -o . ../../raw_data/*.fastq

Quality trim;
cd ..
mkdir 2_trimmomatic
cd 2_trimmomatic



trimmomatic PE \
../../raw_data/SRR364118_1.fastq \
../../raw_data/SRR364118_2.fastq \
SRR364118_1.trimmed.paired.fastq \
SRR364118_1.trimmed.unpaired.fastq \
SRR364118_2.trimmed.paired.fastq \
SRR364118_2.trimmed.unpaired.fastq \
ILLUMINACLIP:/data/apps/trimmomatic/0.36/adapters/TruSeq3-PE.fa:2:30:10 \
LEADING:3 TRAILING:3 SLIDINGWINDOW:4:15 MINLEN:36


trimmomatic PE \
../../raw_data/SRR534069_1.fastq \
../../raw_data/SRR534069_2.fastq \
SRR534069_1.trimmed.paired.fastq \
SRR534069_1.trimmed.unpaired.fastq \
SRR534069_2.trimmed.paired.fastq \
SRR534069_2.trimmed.unpaired.fastq \
ILLUMINACLIP:/data/apps/trimmomatic/0.36/adapters/TruSeq3-PE.fa:2:30:10 \
LEADING:3 TRAILING:3 SLIDINGWINDOW:4:15 MINLEN:36

trimmomatic PE \
../../raw_data/SRR569170_1.fastq \
../../raw_data/SRR569170_2.fastq \
SRR569170_1.trimmed.paired.fastq \
SRR569170_1.trimmed.unpaired.fastq \
SRR569170_2.trimmed.paired.fastq \
SRR569170_2.trimmed.unpaired.fastq \
ILLUMINACLIP:/data/apps/trimmomatic/0.36/adapters/TruSeq3-PE.fa:2:30:10 \
LEADING:3 TRAILING:3 SLIDINGWINDOW:4:15 MINLEN:36

cd ..
mkdir 3_fastqc
cd 3_fastqc/
fastqc -t 2 -o . ../2_trimmomatic/*.trimmed.paired.fastq


Mapping reference genome;
cd ..
mkdir 4_bwa
cd 4_bwa

bwa index /lustre/projects/qcheng1/EPP622Project/raw_data/Puccinia_graminis.ASM14992v1.dna.toplevel.fa
ls /lustre/projects/qcheng1/EPP622Project/raw_data/

bwa mem \
-t 4 \
/lustre/projects/qcheng1/EPP622Project/raw_data/Puccinia_graminis.ASM14992v1.dna.toplevel.fa \
../2_trimmomatic/SRR364118_1.trimmed.paired.fastq \
../2_trimmomatic/SRR364118_2.trimmed.paired.fastq \
> SRR364118.sam

bwa mem \
-t 4 \
/lustre/projects/qcheng1/EPP622Project/raw_data/Puccinia_graminis.ASM14992v1.dna.toplevel.fa \
../2_trimmomatic/SRR534069_1.trimmed.paired.fastq \
../2_trimmomatic/SRR534069_2.trimmed.paired.fastq \
> SRR534069.sam

bwa mem \
-t 4 \
/lustre/projects/qcheng1/EPP622Project/raw_data/Puccinia_graminis.ASM14992v1.dna.toplevel.fa \
../2_trimmomatic/SRR569170_1.trimmed.paired.fastq \
../2_trimmomatic/SRR569170_2.trimmed.paired.fastq \
> SRR569170.sam


samtools sort -@ 2 -o UTK5.bam UTK5.sam
samtools index UTK5.bam


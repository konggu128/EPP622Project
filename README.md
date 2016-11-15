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
module load sra-tools trimmomatic bwa fastqc bcftools


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


samtools sort -@ 2 -o SRR364118.bam SRR364118.sam
samtools index SRR364118.bam

samtools sort -@ 2 -o SRR534069.bam SRR534069.sam
samtools index SRR534069.bam

samtools sort -@ 2 -o SRR569170.bam SRR569170.sam
samtools index SRR569170.bam

ln -s /lustre/projects/qcheng1/EPP622Project/raw_data/Puccinia_graminis.ASM14992v1.dna.toplevel.fa
ls -l -h

samtools faidx Puccinia_graminis.ASM14992v1.dna.toplevel.fa

samtools tview SRR364118.bam Puccinia_graminis.ASM14992v1.dna.toplevel.fa

cd ..
mkdir 5_samtools
cd 5_samtools

ln -s ../4_bwa/SRR364118.bam
ln -s ../4_bwa/SRR534069.bam
ln -s ../4_bwa/SRR569170.bam
ln -s /lustre/projects/qcheng1/EPP622Project/raw_data/Puccinia_graminis.ASM14992v1.dna.toplevel.fa


create script vcf.sh;
#$ -N samtools
#$ -cwd
#$ -S /bin/bash
#$ -q medium*
#$ -l mem=16G

module load samtools
module load bcftools

samtools mpileup \
-uf Puccinia_graminis.ASM14992v1.dna.toplevel.fa SRR364118.bam \
| \
bcftools call -mv --threads 4 > SRR364118HM.raw.vcf

samtools mpileup \
-uf Puccinia_graminis.ASM14992v1.dna.toplevel.fa SRR534069.bam \
| \
bcftools call -mv --threads 4 > SRR534069HM.raw.vcf

samtools mpileup \
-uf Puccinia_graminis.ASM14992v1.dna.toplevel.fa SRR569170.bam \
| \
bcftools call -mv --threads 4 > SRR569170HM.raw.vcf

save and exit vcf.sh;

chmod u+x vcf.sh
./


bcftools filter -s LowQual -e '%QUAL<20 || DP>500' SRR364118HM.raw.vcf > SRR364118HM.flt.vcf
bcftools filter -s LowQual -e '%QUAL<20 || DP>500' SRR534069HM.raw.vcf > SRR534069HM.flt.vcf
bcftools filter -s LowQual -e '%QUAL<20 || DP>500' SRR569170HM.raw.vcf > SRR569170HM.flt.vcf

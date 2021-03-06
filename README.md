# EPP622Project lab notebook

Out of the storage limit of 40G; Asked Gerald for additional spaces;
qrsh -pe threads 4 -l mem=16G
cd /lustre/projects/qcheng1/EPP622Project/raw_data/

In raw_data folder, download reference genome sequence;
wget ftp://ftp.ensemblgenomes.org/pub/fungi/release-33/fasta/puccinia_graminis/dna/Puccinia_graminis.ASM14992v1.dna.toplevel.fa.gz

In raw_data folder, also download genome sequencing reads of 3 stem rust races;
wget ftp://ftp.sra.ebi.ac.uk/vol1/fastq/SRR364/SRR364118/SRR364118_1.fastq.gz
wget ftp://ftp.sra.ebi.ac.uk/vol1/fastq/SRR364/SRR364118/SRR364118_2.fastq.gz

wget ftp://ftp.sra.ebi.ac.uk/vol1/fastq/SRR534/SRR534069/SRR534069_1.fastq.gz
wget ftp://ftp.sra.ebi.ac.uk/vol1/fastq/SRR534/SRR534069/SRR534069_2.fastq.gz

wget ftp://ftp.sra.ebi.ac.uk/vol1/fastq/SRR569/SRR569170/SRR569170_1.fastq.gz
wget ftp://ftp.sra.ebi.ac.uk/vol1/fastq/SRR569/SRR569170/SRR569170_2.fastq.gz

gunzip *.gz

load modules;
module load sra-tools trimmomatic bwa fastqc bcftools

Create analysis folder;
cd ..
mkdir analysis
cd analysis

Quality Examination using FASTQC;
mkdir 1_fastqc
cd  1_fastqc/
fastqc -t 2 -o . ../../raw_data/*.fastq

Quality trim for each of the 3 rust races;
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

Quality Examination using FASTQC again;
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

Variant calling;
cd ..
mkdir 5_samtools
cd 5_samtools

ln -s ../4_bwa/SRR364118.bam
ln -s ../4_bwa/SRR534069.bam
ln -s ../4_bwa/SRR569170.bam
ln -s /lustre/projects/qcheng1/EPP622Project/raw_data/Puccinia_graminis.ASM14992v1.dna.toplevel.fa

Create script vcf.sh;
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
bcftools call -mv --threads 4 > SRR364118.raw.vcf
bcftools filter -s LowQual -e '%QUAL<20 || DP<10' SRR364118.raw.vcf > SRR364118.flt.vcf
grep 'PASS' SRR364118.flt.vcf > SRR364118.vcf

samtools mpileup -uf Puccinia_graminis.ASM14992v1.dna.toplevel.fa SRR534069.bam \
| \
bcftools call -mv --threads 4 > SRR534069.raw.vcf
bcftools filter -s LowQual -e '%QUAL<20 || DP<10' SRR534069.raw.vcf > SRR534069.flt.vcf
grep 'PASS' SRR534069.flt.vcf > SRR534069.vcf

samtools mpileup -uf Puccinia_graminis.ASM14992v1.dna.toplevel.fa SRR569170.bam
| \
bcftools call -mv --threads 4 > SRR569170.raw.vcf
bcftools filter -s LowQual -e '%QUAL<20 || DP<10' SRR569170.raw.vcf > SRR569170.flt.vcf
grep 'PASS' SRR569170.flt.vcf > SRR569170.vcf


Save, exit and run vcf.sh;
chmod u+x vcf.sh
./vcf.sh

Combine three individual VCFs into one total VCF and remove duplicated ones;
cat SRR364118.vcf SRR534069.vcf SRR569170.vcf | grep 'supercont' | awk '!seen[$0]++' > total.vcf

homozygous loci;
ls total.vcf | awk '{ print "less -S "$1"|grep -v \"^#\"|grep -v INDEL|grep \"1/1\"|wc -l && echo "$1; }'|sh


Variants shared by 3 races;
mkdir 6_variant
cd 6_variant

cut ../5_samtools/SRR364118.vcf -f 1,2,3,4,5  | grep 'supercont' > SRR364118.final.vcf
cut ../5_samtools/SRR534069.vcf -f 1,2,3,4,5  | grep 'supercont' > SRR534069.final.vcf
cut ../5_samtools/SRR569170.vcf -f 1,2,3,4,5  | grep 'supercont' > SRR569170.final.vcf
cat SRR364118.final.vcf SRR534069.final.vcf SRR569170.final.vcf | awk '!seen[$0]++' > total_5columns.vcf

sort SRR569170.final.vcf SRR364118.final.vcf | uniq -d > 18_70.vcf
sort SRR569170.final.vcf SRR534069.final.vcf | uniq -d > 69_70.vcf
sort SRR364118.final.vcf SRR534069.final.vcf | uniq -d > 18_69.vcf
sort SRR364118.final.vcf 69_70.vcf | uniq -d > 18_69_70.vcf
wc -l *vcf

Indels;
cd ..
mkdir 7_indel
cd 7_indel
grep 'supercont' ../5_samtools/SRR364118.vcf | grep 'INDEL' | cut -f 1,2,3,4,5 > SRR364118.indel.vcf
grep 'supercont' ../5_samtools/SRR534069.vcf | grep 'INDEL' | cut -f 1,2,3,4,5 > SRR534069.indel.vcf
grep 'supercont' ../5_samtools/SRR569170.vcf | grep 'INDEL' | cut -f 1,2,3,4,5 > SRR569170.indel.vcf
cat SRR364118.indel.vcf SRR534069.indel.vcf SRR569170.indel.vcf | awk '!seen[$0]++' > indel.vcf
wc -l *vcf

Generate venn diagram in R:
install.packages("VennDiagram")
library(VennDiagram)
draw.triple.venn(area1 = 13553, area2 = 522862, area3 = 307825, n12 = 8440, n23 = 75218, n13 = 5609, n123 = 4779, category = c("UG99", "09ETH8-3", "84KEN8C"), lty = "blank", fill = c("dodgerblue", "goldenrod1", "seagreen3"))


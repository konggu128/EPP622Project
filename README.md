# EPP622Project

Out of the storage limit of 40G; Asked Gerald for additional spaces;
copy the reads of one isolate UTK5 into Newton server through FileZilla;

cd /lustre/projects/qcheng1/EPP622Project/raw_data/

in raw_data file, download reference genome sequence;

wget ftp://ftp.ncbi.nlm.nih.gov/genomes/all/GCA/001/191/645/GCA_001191645.1_P_striiformis_V1/GCA_001191645.1_P_striiformis_V1_genomic.fna.gz

gunzip GCA_001191645.1_P_striiformis_V1_genomic.fna.gz
wc -l GCA_001191645.1_P_striiformis_V1_genomic.fna

load modules;
module load sra-tools trimmomatic bwa fastqc


cd ..
mkdir analysis
cd analysis
 
Quality Examination using FASTQC;
mkdir 1_fastqc
cd  1_fastqc/
fastqc -t 2 -o . ../../raw_data/UTK-5.fastq

Quality trim;
trimmomatic SE \
../../raw_data/UTK-5.fastq \
UTK-5.trimmed.fastq \
LEADING:3 TRAILING:3 SLIDINGWINDOW:4:15 MINLEN:36









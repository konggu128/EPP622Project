# EPP622Project
copy the reads of one isolate UTK5 into Newton server through FileZilla;
load modules;

module load sra-tools trimmomatic bwa fastqc

in raw_data file, download reference genome sequence;
cd EPP622Project/raw_data/
wget ftp://ftp.ncbi.nlm.nih.gov/genomes/all/GCA/001/191/645/GCA_001191645.1_P_striiformis_V1/GCA_001191645.1_P_striiformis_V1_genomic.fna.gz

gunzip GCA_001191645.1_P_striiformis_V1_genomic.fna.gz
wc -l GCA_001191645.1_P_striiformis_V1_genomic.fna.gz

 cd ..
 mkdir analysis
 cd analysis
 
 Quality Examination using FASTQC;
 mkdir 1_fastqc
 fastqc -t 2 -o . ../raw_data/UTK-5.fastq
 











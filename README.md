
# Abstract  (1) : Corona_virus

In December 2019, identification of a novel coronavirus was declared as the main cause for viral pneumonia in Wuhan,China. 
The novel virus is known as 2019-nCov Up till the moment, It is not certain which animal was the main cause for intiating the outbreak 
at Wuhan market. However, There are few suspects that bats are the main host. More than 2000 cases of the novel virus have been reported
 and the human-tohuman transmission of infection was confirmed.

 [1] Study Design: 
1- The study is to investigate coronavirus outbreak by Retrieval of the short read dataset 
2- Sub setting to produce 5 million reads 
3-Making alignment to an appropriate genome and generating Sam file. 
4- Selecting the reads that made secondary alignment 
5- Calculate the GC content and quality scores of such reads 


# **First Trial**

## Setting Environment

At the begining, data retrieval was by Search shor read archive (anient dna) then specifying it a bit then
Send results to run selector (for it to be filtered)

Revert to the old run selector
`sudo apt-get install sra-toolkit`

The past code did not install toolkit , so used
`sudo apt install sra-toolkit`

There was a problem running in the background that bonder the toolkit from being installed 
checked by this code
` ps aux | grep -i apt`

The error was in installing while the system was being updated , So waiting for 15 min til updats finished and the package was successfuly installed 

##Data Retrieval :
NCBI was used to search for  2019-nCoV (Wuhan coronavirus) sequences that is currently available in GenBank and the Sequence Read Archive (SRA). The run selected was SRR10903402, other runs were more than 1 gigabytes (No knowledge at subsetting Data at this point)

[https://www.ncbi.nlm.nih.gov/genbank/2019-ncov-seqs/#sra-sequences](url)


`mkdir -p /ngs_proj/sample_data  && cd ~/ngs_proj/sample_data
prefetsh SRR10903402 -X 205000`

The Prefetch was  used to download the short reads for the selected Run SRR10903402  with maximum size of 205Mega.

The reads, in a single file, were paired-ends so it needs to be splitted for the downstream analysis.
`fastq-dump --outdir fastq --gzip --skip-technical  --readids --read-filter pass --dumpbase --split-3 --clip SRR10903402`

–split-3 was used to separate the read into left and right ends each in a single file R1& R2, the third file will be for left ends without a matching right end and for a right end without a matching left ends  they will be put in a single file.

--skip-technical to remove technical reads produced from the “Illumina multiplex library construction protocol” 

–clip to Apply left and right clips to remove sequences that include tags which were used for amplification.

## **Reference Genome**

The goal was to align the short-read sequences of Corona-virus against ebola virus and what is remaining should be the cause for its pathogenicity 

The Reference genome was ebola virus 

` wget ftp://hgdownload.soe.ucsc.edu/goldenPath/eboVir3/ebolaAbSequences.fasta`
` wc -l ebolaAbSequences.fasta `

after this , this reference genome was the recent reference for the novel virus so downloaded afterwards to be used instead of ebola virus

` wget https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/009/858/895/GCF_009858895.2_ASM985889v3/GCF_009858895.2_ASM985889v3_genomic.gff.gz
gunzip GCF_009858895.2_ASM985889v3_genomic.gff.gz`

```
wget ftp://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/005/845/GCF_000005845.2_ASM584v2/GCF_000005845.2_ASM584v2_genomic.fna.gz
  tar -xvf genome_assemblies.tar 
  head -10 ncbi-genomes-2020-02-06/GCF_009858895.2_ASM985889v3_genomic.fna.gz 
  gunzip -k ncbi-genomes-2020-02-06/GCF_009858895.2_ASM985889v3_genomic.fna.gz 
  cd ncbi-genomes-2020-02-06/
  cp GCF_009858895.2_ASM985889v3_genomic.fna 
```

### **For subsetting the data:**


```
sudo apt install seqtk
seqtk - c.f
# this didn't actually subset the data but it was discovered later on 
```

` seqtk sample -s100 SRR10903402_pass_1.fastq.gz 5000000 > sub1.fq
 seqtk sample -s100 SRR10903402_pass_2.fastq.gz 5000000 > sub2.fq
`
 Seed number should be the same so that it will generate the same subset for both reads, This was supposed to subset the first 5 million reads but later on discovered to subset 20Million as it is multiplies by 4 so to subset the first 5 M reads it should be 1250000 

### **Quality Checking**

Quality control of reads was performed vi FastQC tool 
```
conda activate ngs1
conda install -c bioconda fastqc 
conda install -c bioconda multiqc 
```
both interactive and non interactive code was used to check the quality of the sequences 

```
for f in ~/home/nu/ngs_proj/fastq/*.fq.gz;
do fastqc -t 1 -f fastq -noextract $f;done
```

# Results of Fastqc Quality Control Checks:
SRR10903402_pass_1_fastqc:
Summary statistics:
[fastqc_data.txt](https://github.com/hagarelsayed/Ngs_1st_Abstract/files/4177094/fastqc_data.txt)
[summary.txt](https://github.com/hagarelsayed/Ngs_1st_Abstract/files/4177095/summary.txt)
![1](https://user-images.githubusercontent.com/60422836/74104910-1a661f80-4b62-11ea-88a9-487f61f54551.png)

Quality per base 
![per_base_quality](https://user-images.githubusercontent.com/60422836/74104888-e428a000-4b61-11ea-8746-dab1e9f0d5c8.png)

![adapter_content](https://user-images.githubusercontent.com/60422836/74104938-4a152780-4b62-11ea-8a6e-ebbd82485bb8.png)
![duplication_levels](https://user-images.githubusercontent.com/60422836/74104940-4b465480-4b62-11ea-8b81-adc3989aa857.png)
![per_base_n_content](https://user-images.githubusercontent.com/60422836/74104942-4b465480-4b62-11ea-91c3-3c53046dee22.png)
![per_base_sequence_content](https://user-images.githubusercontent.com/60422836/74104943-4bdeeb00-4b62-11ea-9eac-33047cd3e0fb.png)
![per_sequence_gc_content](https://user-images.githubusercontent.com/60422836/74104944-4bdeeb00-4b62-11ea-9810-d38bb3aa57a6.png)
![per_sequence_quality](https://user-images.githubusercontent.com/60422836/74104945-4c778180-4b62-11ea-895d-23150bdc20df.png)
![sequence_length_distribution](https://user-images.githubusercontent.com/60422836/74104946-4d101800-4b62-11ea-8aa9-b5b25a9e5142.png)


## Quality for SRR10903402_R2
[fastqc_data.txt](https://github.com/hagarelsayed/Ngs_1st_Abstract/files/4177107/fastqc_data.txt)
[summary.txt](https://github.com/hagarelsayed/Ngs_1st_Abstract/files/4177109/summary.txt)
![2](https://user-images.githubusercontent.com/60422836/74105012-d4f62200-4b62-11ea-95b5-9cad179c2da9.png)

![per_base_quality](https://user-images.githubusercontent.com/60422836/74104958-5ef1bb00-4b62-11ea-9d5e-158faf233c19.png)


![duplication_levels](https://user-images.githubusercontent.com/60422836/74104968-6c0eaa00-4b62-11ea-9f6d-d65986b2a6a7.png)
![per_base_n_content](https://user-images.githubusercontent.com/60422836/74104969-6ca74080-4b62-11ea-88b9-5153fc25427e.png)
![per_base_sequence_content](https://user-images.githubusercontent.com/60422836/74104970-6d3fd700-4b62-11ea-947a-6f385cdee6a3.png)
![per_sequence_gc_content](https://user-images.githubusercontent.com/60422836/74104971-6dd86d80-4b62-11ea-8ea4-a5dc96010ee7.png)
![per_sequence_quality](https://user-images.githubusercontent.com/60422836/74104972-6dd86d80-4b62-11ea-8241-734085440e7e.png)
![sequence_length_distribution](https://user-images.githubusercontent.com/60422836/74104973-6e710400-4b62-11ea-8a6e-8cf2bc0478d7.png)
![adapter_content](https://user-images.githubusercontent.com/60422836/74104974-6f099a80-4b62-11ea-854f-5f14a7143858.png)


Indexing:
I don't remember the problem here but there was a problem in the sam file (plz check from terminal)
```
R1="SRR10903402_pass_1.fastq.gz" 
 less R1
head -n 10 19cov-Rep1.sam 
```

I was trying to troubleshoot the error by screenfastq 

```
wget http://www.bioinformatics.babraham.ac.uk/projects/fastq_screen/fastq_screen_test_dataset.tar.gz
  tar xvzf fastq_screen_v0.14.0.tar.gz 
  tar xvzf fastq_screen_test_dataset.tar.gz 
 cp fastq_screen.conf.example fastq_screen.conf
```
 
# Indexing of Reference genome

 ```
ln -s /home/nu/ngs-project/corona_vir/ncbi-genomes-2020-02-06/GCF_009858895.2_ASM985889v3_genomic.fna .
hisat2_extract_splice_sites.py /home/nu/ngs-project/corona_vir/ncbi-genomes-2020-02-06/GCF_009858895.2_ASM985889v3_genomic.gtf > splicesites_19cov.tsv

hisat2_extract_exons.py /home/nu/ngs-project/corona_vir/ncbi-genomes-2020-02-06/GCF_009858895.2_ASM985889v3_genomic.gtf > exons_19cov.tsv

hisat2-build -p 1 --ss splicesites_19cov.tsv --exon exons_19cov.tsv GCF_009858895.2_ASM985889v3_genomic.fa GCF_009858895.2_ASM985889v3_genomic

```
# Alignment of Reads 
```
R1="$/home/nu/ngs-project/corona_vir/fastq/SRR10903402_pass_1.fastq.gz"
R2="$/home/nu/ngs-project/corona_vir/fastq/SRR10903402_pass_2.fastq.gz"
hisat2 -p 1 -x hisatIndex/GCF_009858895.2_ASM985889v3_genomic --dta --rna-strandness RF -1 $R1 -2 $R2 -S 19cov-Rep1.sam
```
There was an Error in alignment , tried to solve it by (needs to be checked from terminal history

```
cat R1
   R2="$~/home/nu/ngs-project/corona_vir/fastq/SRR10903402_pass_2.fastq.gz"
  cat R2
 head R2
 R2="$HOME/nu/ngs-project/corona_vir/fastq/SRR10903402_pass_2.fastq.gz"
 cat R2
 R1="$HOME/workdir/sample_data/SRR10903402_pass_1.fastq.gz"
 R1
 cat R1
 R1="$HOME/workdir/sample_data/SRR10903402_pass_1.fastq.gz"
 R2="$HOME/workdir/sample_data/SRR10903402_pass_2.fastq.gz"
 echo R1
 echo R2
 echo $R1
hisat2 -p 1 -x hisatIndex/GCF_009858895.2_ASM985889v3_genomic --dta --rna-strandness RF -1 $R1 -2 $R2 -S 19cov-Rep1.sam
R1="$HOME/workdir/sample_data/SRR10903402_pass_1_subset.fq"
 R2="$HOME/workdir/sample_data/SRR10903402_pass_2_subset.fq"
 hisat2 -p 1 -x hisatIndex/GCF_009858895.2_ASM985889v3_genomic --dta --rna-strandness RF -1 $R1 -2 $R2 -S SRR10903402_subset.sam

```
Shifted back again to fastq screen to get the matching genome for the available sra but fatqc wasn't able to be properly configures
   
```
fastq screen
 wget http://www.bioinformatics.babraham.ac.uk/projects/fastq_screen/fastq_screen_v0.14.0.tar.gz
 emacs 
 ls
 less 19cov-Rep1.sam 
emacs fastq_screen.conf
 cp fastq_screen_test_dataset/fqs_test_dataset.fastq.gz .
 sudo apt install bwa
 conda install -c bioconda -y bwa
 cd ..
 ls
 fastq_screen_v0.14.0/fastq_screen fqs_test_dataset.fastq.gz 
 conda activate ngs1
 fastq_screen_v0.14.0/fastq_screen fqs_test_dataset.fastq.gz 
 conda install -c bioconda -y bwa
 fastq_screen_v0.14.0/fastq_screen fqs_test_dataset.fastq.gz 

```


## Resulted Hisat2 files

```
587809 reads; of these:
  587809 (100.00%) were paired; of these:
    549354 (93.46%) aligned concordantly 0 times
    38455 (6.54%) aligned concordantly exactly 1 time
    0 (0.00%) aligned concordantly >1 times
    ----
    549354 pairs aligned concordantly 0 times; of these:
      6887 (1.25%) aligned discordantly 1 time
    ----
    542467 pairs aligned 0 times concordantly or discordantly; of these:
      1084934 mates make up the pairs; of these:
        1080225 (99.57%) aligned 0 times
        4709 (0.43%) aligned exactly 1 time
        0 (0.00%) aligned >1 times
8.11% overall alignment rate
```






Sam Files were too large to be uploaded, around 1.2 GigaBytes

Indexed files by Hisat shared through this link 
https://drive.google.com/open?id=1gju43gCz_tDB6vskA--JMUhBso0_4nLj



## N.B.
The work was stopped here at Feb 6, 7 pm as Dr Tamer approved the other abstract to work on and approved Sara to join me and work as a team in the other abstract. 






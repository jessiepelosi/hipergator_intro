# Arabidopsis Genome Assembly Tutorial 

By the end of this tutorial you should be able to:
1. Generate a genome assembly from short read sequence data 
2. Generate a genome assembly using both long and short read sequence data 

This is a tutorial for the assembly of the <i> Arabidopsis </i> genome. This tutorial is modified from [Arun Seetharam](https://bioinformaticsworkbook.org/dataAnalysis/GenomeAssembly/Arabidopsis/AT_platanus-genome-assembly.html#gsc.tab=0).
The first part of this tutorial uses SOAPdenovo to assemble the <i> Arabidopsis </i> genome with short reads, while the second part of the tutorial will use a hybrid assembly approach. 

# Short Read Assembly 

Data for this part of the assembly can be found under BioProject [PRJNA311266](https://www.ncbi.nlm.nih.gov/bioproject/PRJNA311266) on the National Center for Biotechnology Information ([NCBI](https://www.ncbi.nlm.nih.gov/)). We will download just the short read for this part of the tutorial. 

| SRA Accession | Instrument  | Layout     |Insert (bp)| Read Length |Tota lReads |	Bases (Mbp) |
| ------------- |-------------|------------|-----------|-------------|------------|-------------|
|SRR3157034     | HiSeq 2000	| paired-end | 0         | 100x2       |93,446,768  | 17,823      |
|SRR3166543     | HiSeq 2000  | paired-end | 0         | 100x2       |162,362,560 | 30,968      |
|SRR3156163     | HiSeq 2000	| mate-pairs | 8,000     | 100x2       |51,332,776  | 9,790       |
|SRR3156596     | HiSeq 2000	| mate-pairs | 20,000    | 100x2       |61,030,552  | 11,640      |

<b> 1. Download the Data </b> 
 
First, make a new directory to hold the data you will be working with. 
```
mkdir Arabidopsis_genome 
cd Arabidopsis_genome
```

You can download data using the `sra` module on HiPerGator like this. Remeber to run all your commands with a SLURM script! 
```
module load sra
fasterq-dump SRR3157034 --split-files 
fasterq-dump SRR3166543 --split-files
fasterq-dump SRR3156163 --split-files
fasterq-dump SRR3156596 --split-files
```

<b> 2. Quality Check </b> 

Next, you'll need to check the data to make sure they look alright to use for the assembly in a "quality check" step. This is done with [FastQC](https://www.bioinformatics.babraham.ac.uk/projects/fastqc/). FastqQC is a light-weight program written in Java that is used to check the quality of sequence data.
Sequence data are commonly written in fastq and fasta file formats - fastq files contain information about the quality of each base pair while fasta files do not contain this information. The IT staff for HiPerGator have written a wrapper script to simplify the commands for us. 
``` 
module load fastqc
fastqc *.fastq
```
The output from FastQC is an HTML file for each input file. You'll need to download these to your local computer and then open them (an internet Browser like Chrome should work). 

 

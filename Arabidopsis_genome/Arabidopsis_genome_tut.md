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

## 1. Download the Data
 
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

> Note: Once you have retrieved the raw reads, you may want to make a directory to keep them in by themselves for organizational purposes. You may also want to do the same with each step below just so you know which files belong to each step. However, the way you organize is ultimately up to you -- you will figure out an organization system that works for you with practice!

## 2. Quality Check

Next, you'll need to check the data to make sure they look alright to use for the assembly in a "quality check" step. This is done with [FastQC](https://www.bioinformatics.babraham.ac.uk/projects/fastqc/). FastqQC is a light-weight program written in Java that is used to check the quality of sequence data.
Sequence data are commonly written in fastq and fasta file formats - fastq files contain information about the quality of each base pair while fasta files do not contain this information. The IT staff for HiPerGator have written a wrapper script to simplify the commands for us. 
``` 
module load fastqc
fastqc *.fastq
```
The output from `FastQC` is an HTML file for each input file. You'll need to download these to your local computer and then open them (an internet Browser like Chrome should work). 

## 3. Trim and Quality Filter Reads
First, you will want to remove the adaptors from each read. Adaptors are little sequence fragments that act as a barcode or ID number. They are at the beginning of each read to let the sequencing machine know which reads belong to which sample (**May move some of this explanation above depending on what you plan to say about the FastQC output!**). Since they are not actually a sequence from the genome of the sample, you should remove them before doing any assembly using a program like [`Trimmomatic`](http://www.usadellab.org/cms/?page=trimmomatic). 

Read filtering/trimming programs like `Trimmomatic` can also remove reads of poor quality. You will want to do that too; this will ensure that you are using untrustworthy data in the rest of the analysis. (Garbage in, garbage out!) 

When running `Trimmomatic`, you can do adaptor trimming and quality filtering all at once. Since our reads are <b>paired-end</b>, you will need to use the paired-end option in `Trimmomatic` and indicate that the corresponding paired-end files should be matched together in your call to the program, like so:

'''
module load trimmomatic
trimmomatic PE --phred33 input_forward.fq.gz input_reverse.fq.gz output_forward_paired.fq.gz \
output_forward_unpaired.fq.gz output_reverse_paired.fq.gz output_reverse_unpaired.fq.gz \
ILLUMINACLIP:${HPC_TRIMMOMATIC_ADAPTER}/TruSeq3-PE.fa:2:30:10 LEADING:3 TRAILING:3 SLIDINGWINDOW:4:15 MINLEN:36
'''

Let's break down what each of these arguments mean (**May move some of this explanation above depending on what you plan to say about the FastQC output!**):
* <b>PE</b>: run `Trimmomatic` for paired-end sequences
* <b>phred</b>: [phred scores](https://en.wikipedia.org/wiki/Phred_quality_score) estimate the quality of the read on a logarithmic scale. We specified a minimum phred score of 33, which translates to about a 1 in 2000 chance of a base call in the read being wrong. Standard values for minimum phred scores usually range between 30 and 35.
* <b>unpaired output files</b>: Sometimes `Trimmomatic` will toss out a one read in a pair but not the other. The remaining read becomes unpaired, but can still be used in subsequent analyses, so it outputs them in unpaired read files.
* <b>ILLUMINACLIP</b>: This is the portion of the program call used to trim away the read adaptors.
    * Adaptor name here: This is the file which tells `Trimmomatic` what sequences are part of the adaptors, and therefore what to remove.
    * <b>LEADING</b>: 
    * <b>TRAILING</b>:
    * <b>SLIDINGWINDOW</b>:
    * <b>MINLEN</b>: `Trimmomatic` will toss any reads that are shorter than 36 base pairs after it is done cutting off the adaptor and poor-quality bases on the end of the reads.

## 4. Filter Reads for Contamination

## 5. Estimate Genome Size

## 6. Genome Assembly

## 7. Assess Assembly Quality

# Long and Short Read Assembly

Under construction!
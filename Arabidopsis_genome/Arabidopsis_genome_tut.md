# Arabidopsis Genome Assembly Tutorial 

By the end of this tutorial you should be able to:
1. Generate a genome assembly from short read sequence data 
2. Generate a genome assembly using both long and short read sequence data 

This is a tutorial for the assembly of the <i> Arabidopsis </i> genome. 
The first part of this tutorial uses SOAPdenovo to assemble the <i> Arabidopsis </i> genome with short reads, while the second part of the tutorial will use a hybrid assembly approach. 

# Short Read Assembly 

Data for this part of the assembly can be found under BioProject [PRJEB14131](https://www.ncbi.nlm.nih.gov/bioproject/PRJEB14131) on the National Center for Biotechnology Information ([NCBI](https://www.ncbi.nlm.nih.gov/)). We will download just the short read for this part of the tutorial. 

| SRA Accession | Instrument  | Layout     |Insert (bp)| Read Length |	Bases       |
| ------------- |-------------|------------|-----------|-------------|-------------|
|ERR1424597     | HiSeq 2000	| paired-end | 0         | 100x2        | 17.2 Gb     |


<b> 1. Download the Data </b> 
 
First, make a new directory to hold the data you will be working with. 
```
mkdir Arabidopsis_genome 
cd Arabidopsis_genome
```

You can download data using the `sra` module on HiPerGator like this. We will only be using SRR3157034 and SRR3156163 for this tutorial since the files are quite large.Remeber to run all your commands with a SLURM script! 
```
module load sra
fasterq-dump ERR1424597 --split-files 
```

<b> 2. Quality Check </b> 

Next, you'll need to check the data to make sure they look alright to use for the assembly in a "quality check" step. This is done with [FastQC](https://www.bioinformatics.babraham.ac.uk/projects/fastqc/). FastqQC is a light-weight program written in Java that is used to check the quality of sequence data.
Sequence data are commonly written in fastq and fasta file formats - fastq files contain information about the quality of each base pair while fasta files do not contain this information. The IT staff for HiPerGator have written a wrapper script to simplify the commands for us. 
``` 
module load fastqc
fastqc *.fastq
```
The output from FastQC is an HTML file for each input file. You'll need to download these to your local computer and then open them (an internet Browser like Chrome should work). 

Here are two parts of the output file that are used most often (these are from the forward reads, ERR1424597_1.fastq). 

![Per-base sequence quality (the first frame from the output of FastQC](https://github.com/jessiepelosi/hipergator_intro/blob/main/Arabidopsis_genome/Arabid_fastqc1.PNG "Per-Base Sequence Quality")

This is the per-base sequence quality averaged across all the sequence reads. You ideally want all your reads above Q28, in the green area, but it is okay if some of them fall into the yellow. 

![Adaptor content](https://github.com/jessiepelosi/hipergator_intro/blob/main/Arabidopsis_genome/Arabid_fastqc2.PNG "Adaptor content")

This is pretty good too, but you can see that FastQC also highlights parts of the analysis (in this case, the adaptor content) that could be improved with the orange exclamanation point. We'll have to remove the adaptors used in Illumina sequencing in the next step. 

<b> 3. Trim Reads </b> 

After quality-checking the data, you'll want to remove bases that are low-quality and get rid of any adaptor sequences that may be in the raw reads files. I usually use [Trimmomatic](http://www.usadellab.org/cms/?page=trimmomatic) for this, although there are many other programs you can use. This program is also written in Java, and again there is wrapper script to make the commands easier for us. I've put the command down below and will explain each part of the command. Make sure that the TruSeq files are in the same directory as the reads you are trimming (I have included them in this GitHub repository).  

```
module load trimmomatic/0.39
trimmomatic PE ERR1424597_1.fastq ERR1424597_2.fastq ERR1424597_1P.fastq ERR1424597_1U.fastq ERR1424597_2P.fastq ERR1424597_2U.fastq ILLUMINACLIP:TruSeq2-PE.fa:2:30:10 ILLUMINACLIP:TruSeq3-PE.fa:2:30:10 HEADCROP:5 SLIDINGWINDOW:4:20 MINLEN:36
```
We are working with paired-end reads so we want to tell Trimmomatic to work in "paired-end mode" (`PE`). We then include the names of the input files (`ERR1424597_1.fastq ERR1424597_2.fastq`) and the output files- note that Trimmomatic will output paired reads (`ERR1424597_1P.fastq`) and unpaired reads (`ERR1424597_1U.fastq`). For our downstream analyses, we'll want to use the paired-end read files. Then we'll tell Trimmomatic to remove adaptor sequences (`ILLUMINACLIP:TruSeq2-PE.fa:2:30:10 ILLUMINACLIP:TruSeq3-PE.fa:2:30:10`), remove the first 5 bases of each read regardless of quality (`HEADCROP:5`), use a sliding window to remove bases (in groups of four) that have an average quality less than 20 (`SLIDINGWINDOW:4:20`), and only retain reads that are longer than 36 bp (`MINLEN:36`). Trimmomatic works in order of the command you give it, so you'll get different results if you switch around the order of these commands. 

Once you've run Trimmomatic, you'll want to re-run FastQC to make sure the reads look good! After trimming the reads, the output from FastQC for ERR1424597_1P.fastq should look something like this (I've only displayed the first frame of the output file): 

<b> 4. Estimate Genome Size  </b> 

Next, you'll want to estimate the genome size using k-mers, which are short substrings of length contained within a DNA sequence. We first have to count k-mers of a certain length (most commonly we use 17 or 21bp k-mers, which can also be written as 17-mer and 21-mer). We'll use [Jellyfish](http://www.genome.umd.edu/jellyfish.html#:~:text=Jellyfish%20is%20a%20tool%20for,of%20k%2Dmers%20in%20DNA.&text=Jellyfish%20is%20a%20command%2Dline,the%20%22jellyfish%20dump%22%20command.) for this step. 
```
module load jellyfish/2.3.0
jellyfish count -C -m 21 -s 1000000000 -t 10 *P.fastq -o reads.jf
jellyfish histo -t 10 reads.jf > reads.histo
```
Note that Jellyfish is quite computationally expensive! You'll want to make sure to allocate a lot of memory in your SLURM script (this job took XGb of RAM and X hours when I ran it).  

Once you've generated the read histogram from Jellyfish, you can upload the reads.histo file to [GenomeScope](http://qb.cshl.edu/genomescope/). GenomeScope uses a modelling approach to estimate genome heterozygosity, repeat content, and size from sequencing reads using a kmer-based statistical approach. You can also do write a short R script to estimate genome size following [this tutorial](http://koke.asrc.kanazawa-u.ac.jp/HOWTO/kmer-genomesize.html).  

Estimate best k-mer? 

<b> 5. Assembly with SOAPdenovo  </b> 

[SOAPdenovo](https://github.com/aquaskyline/SOAPdenovo2) is a commonly used short-read genome assembly program. It is relatively easy to use once you've gotten the configuration file set up. This is what the config file for this assembly could look like: 
```
#maximal read length
max_rd_len=100
[LIB]
#average insert size
avg_ins=300
#if sequence needs to be reversed
reverse_seq=0
#in which part(s) the reads are used
asm_flags=3
# cutoff of pair number for a reliable connection (at least 3 for short insert size)
pair_num_cutoff=3
#minimum aligned length to contigs for a reliable read location (at least 32 for short insert size)
map_len=32
#a pair of fastq files, read 1 file should always be followed by read 2 file
q1=/path/**LIBNAMEA**/ERR1424597_1P.fastq
q2=/path/**LIBNAMEA**/ERR1424597_2P.fastq
``` 
To run the program (all parts) use the command: 
```
module load soapdenovo
SOAPdenovo all -s file.config -K 33 -R -p 64 -N 135000000 -o A.thaliana.33mer 1>ass.log 2>A.thaliana.33mer.err 
```
This program also uses a lot of RAM and time to run, so make sure to allocate enough resources in your SLURM script. 

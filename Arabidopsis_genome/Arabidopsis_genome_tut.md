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


## 1. Download the Data
 
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

> Note: Once you have retrieved the raw reads, you may want to make a directory to keep them in by themselves for organizational purposes. You may also want to do the same with each step below just so you know which files belong to each step. However, the way you organize is ultimately up to you -- you will figure out an organization system that works for you with practice!

## 2. Quality Check

Next, you'll need to check the data to make sure they look alright to use for the assembly in a "quality check" step. This is done with [FastQC](https://www.bioinformatics.babraham.ac.uk/projects/fastqc/). FastqQC is a light-weight program written in Java that is used to check the quality of sequence data.
Sequence data are commonly written in fastq and fasta file formats - fastq files contain information about the quality of each base pair while fasta files do not contain this information. The IT staff for HiPerGator have written a wrapper script to simplify the commands for us. 
``` 
module load fastqc
fastqc *.fastq
```
The output from `FastQC` is an HTML file for each input file. You'll need to download these to your local computer and then open them (an internet Browser like Chrome should work). 

Here are two parts of the output file that are used most often (these are from the forward reads, ERR1424597_1.fastq). 

![Per-base sequence quality (the first frame from the output of FastQC](https://github.com/jessiepelosi/hipergator_intro/blob/main/Arabidopsis_genome/Arabid_fastqc1.PNG "Per-Base Sequence Quality")

This is the per-base sequence quality averaged across all the sequence reads. You ideally want all your reads above Q28, in the green area, but it is okay if some of them fall into the yellow. 

![Adaptor content](https://github.com/jessiepelosi/hipergator_intro/blob/main/Arabidopsis_genome/Arabid_fastqc2.PNG "Adaptor content")

This is pretty good too, but you can see that FastQC also highlights parts of the analysis (in this case, the adaptor content) that could be improved with the orange exclamanation point. We'll have to remove the adaptors used in Illumina sequencing in the next step. 

## 3. Trim and Quality Filter Reads
First, you will want to remove the adaptors from each read. Adaptors are little sequence fragments that act as a barcode or ID number. They are at the beginning of each read to let the sequencing machine know which reads belong to which sample (**May move some of this explanation above depending on what you plan to say about the FastQC output!**). Since they are not actually a sequence from the genome of the sample, you should remove them before doing any assembly using a program like [`Trimmomatic`](http://www.usadellab.org/cms/?page=trimmomatic). 

Read filtering/trimming programs like `Trimmomatic` can also remove reads of poor quality. You will want to do that too; this will ensure that you are using untrustworthy data in the rest of the analysis. (Garbage in, garbage out!) 

When running `Trimmomatic`, you can do adaptor trimming and quality filtering all at once. Since our reads are <b>paired-end</b>, you will need to use the paired-end option in `Trimmomatic` and indicate that the corresponding paired-end files should be matched together in your call to the program, like so:

```
module load trimmomatic/0.39
trimmomatic PE ERR1424597_1.fastq ERR1424597_2.fastq ERR1424597_1P.fastq ERR1424597_1U.fastq ERR1424597_2P.fastq ERR1424597_2U.fastq ILLUMINACLIP:TruSeq2-PE.fa:2:30:10 ILLUMINACLIP:TruSeq3-PE.fa:2:30:10 HEADCROP:5 SLIDINGWINDOW:4:20 MINLEN:36
```
```
module load trimmomatic
trimmomatic PE --phred33 input_forward.fq.gz input_reverse.fq.gz output_forward_paired.fq.gz \
output_forward_unpaired.fq.gz output_reverse_paired.fq.gz output_reverse_unpaired.fq.gz \
ILLUMINACLIP:${HPC_TRIMMOMATIC_ADAPTER}/TruSeq3-PE.fa:2:30:10 LEADING:3 TRAILING:3 SLIDINGWINDOW:4:15 MINLEN:36
```

Let's break down what each of these arguments mean (**May move some of this explanation above depending on what you plan to say about the FastQC output!**):
* <b>PE</b>: Run `Trimmomatic` for paired-end sequences
* <b>phred</b>: A [phred score](https://en.wikipedia.org/wiki/Phred_quality_score) estimates the quality of the read on a logarithmic scale. We specified a minimum phred score of 33, which translates to about a 1 in 2000 chance of a base call in the read being wrong. Standard values for minimum phred scores usually range between 30 and 35.
* <b>unpaired output files</b>: Sometimes `Trimmomatic` will toss out a one read in a pair but not the other. The remaining read becomes unpaired, but can still be used in subsequent analyses, so it outputs them in unpaired read files.
* <b>ILLUMINACLIP</b>: This is the portion of the program call used to trim away the read adaptors.
    * Adaptor name here: This is the file which tells `Trimmomatic` what sequences are part of the adaptors, and therefore what to remove.
    * <b>LEADING</b>: 
    * <b>TRAILING</b>:
    * <b>SLIDINGWINDOW</b>:
    * <b>MINLEN</b>: `Trimmomatic` will toss any reads that are shorter than 36 base pairs after it is done cutting off the adaptor and poor-quality bases on the end of the reads.

After you're done trimming and filtering reads, you'll want to check their quality again in `FastQC`.
``` 
module load fastqc
fastqc *.fastq
```
Take a look at your results -- did the quality of your reads improve?

We are working with paired-end reads so we want to tell Trimmomatic to work in "paired-end mode" (`PE`). We then include the names of the input files (`ERR1424597_1.fastq ERR1424597_2.fastq`) and the output files- note that Trimmomatic will output paired reads (`ERR1424597_1P.fastq`) and unpaired reads (`ERR1424597_1U.fastq`). For our downstream analyses, we'll want to use the paired-end read files. Then we'll tell Trimmomatic to remove adaptor sequences (`ILLUMINACLIP:TruSeq2-PE.fa:2:30:10 ILLUMINACLIP:TruSeq3-PE.fa:2:30:10`), remove the first 5 bases of each read regardless of quality (`HEADCROP:5`), use a sliding window to remove bases (in groups of four) that have an average quality less than 20 (`SLIDINGWINDOW:4:20`), and only retain reads that are longer than 36 bp (`MINLEN:36`). Trimmomatic works in order of the command you give it, so you'll get different results if you switch around the order of these commands. 

Once you've run Trimmomatic, you'll want to re-run FastQC to make sure the reads look good! After trimming the reads, the output from FastQC for ERR1424597_1P.fastq should look something like this (I've only displayed the first frame of the output file): 


## 4. Filter Reads for Contamination
In addition to removing poor quality reads, you will also want to remove any reads that do not belong to the nuclear genome of your sample. You may recall that due to endosymbiosis, organelles (mitochondria, chloroplasts) have their own genomes separate from the nuclear genome. Because mitochondria and chloroplasts are so much more numerous than nuclei, you are inevitably going to get reads from their genomes in your dataset along with your nuclear reads. Since they're from separate genomes, they should be separated from your nuclear reads before assembly.

First, let's download reference sequences for the <i> Arabidopsis </i> [chloroplast](https://www.ncbi.nlm.nih.gov/nucleotide/NC_000932.1) and [mitochondrial](https://www.ncbi.nlm.nih.gov/nucleotide/NC_037304.1) genome.

```
# chloroplast
wget https://www.ncbi.nlm.nih.gov/kis/download/sequence/?db=nucleotide&id=NC_000932.1

# mitochondrion
wget https://www.ncbi.nlm.nih.gov/kis/download/sequence/?db=nucleotide&id=NC_037304.1
```

> Note: If you end up doing genome assembly with an organism that doesn't have these references available, you can download a reference from a close relative instead.

Now that you have a reference, you will need to map your reads to the reference genomes. This will tell you which reads match with sequences and where they match in the reference. There are many different mapping programs; we will be using Bowtie2 since it is optimized for mapping short reads to long references.

```
module load bowtie2
bowtie2 
```

## 5. Estimate Genome Size 

Next, you'll want to estimate the genome size using k-mers, which are short substrings of length contained within a DNA sequence. We first have to count k-mers of a certain length (most commonly we use 17 or 21bp k-mers, which can also be written as 17-mer and 21-mer). We'll use [Jellyfish](http://www.genome.umd.edu/jellyfish.html#:~:text=Jellyfish%20is%20a%20tool%20for,of%20k%2Dmers%20in%20DNA.&text=Jellyfish%20is%20a%20command%2Dline,the%20%22jellyfish%20dump%22%20command.) for this step. 
```
module load jellyfish/2.3.0
jellyfish count -C -m 21 -s 1000000000 -t 10 *P.fastq -o reads.jf
jellyfish histo -t 10 reads.jf > reads.histo
```
Note that Jellyfish is quite computationally expensive! You'll want to make sure to allocate a lot of memory in your SLURM script (this job took XGb of RAM and X hours when I ran it).  

Once you've generated the read histogram from Jellyfish, you can upload the reads.histo file to [GenomeScope](http://qb.cshl.edu/genomescope/). GenomeScope uses a modelling approach to estimate genome heterozygosity, repeat content, and size from sequencing reads using a kmer-based statistical approach. You can also do write a short R script to estimate genome size following [this tutorial](http://koke.asrc.kanazawa-u.ac.jp/HOWTO/kmer-genomesize.html).  

## 6. Genome Assembly 

Estimate best k-mer? 

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


## 7. Assess Assembly Quality

# Long and Short Read Assembly

Under construction!


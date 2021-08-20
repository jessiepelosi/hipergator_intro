# Arabidopsis Nanopore Long Reads Assembly Tutorial
By the end of this tutorial, you should be able to:
1. Generate a genome assembly using a mixture of long and short reads

This tutorial uses Nanopore long reads and less intensive assemblers (`Flye`, `wtdbg2`, `miniasm`). While assemblers such as `Canu` can produce high-quality assemblies, we think that the computational resources and file management necessary to run it may be a bit much for an introductory tutorial.

# Long Read Assembly
The data we are using is from [Michael et al. 2018](https://www.nature.com/articles/s41467-018-03016-2); while the study uses PacBio, Nanopore, and Illumina reads, we will only be using their Nanopore reads and cleaning with the _Arabidopsis_ short reads used for [the first part of the tutorial](https://github.com/jessiepelosi/hipergator_intro/blob/main/Arabidopsis_genome/Arabidopsis_genome_tut.md). 

| ENA Accession | Instrument       | Layout     | Read Count  |	Bases       |
| ------------- |------------------|------------|-------------|-------------|
|ERR2173373     | Nanopore MinION	 | single     | 300071      | 3.42 Gb     |

You'll notice that reads generated from "third-generation technologies" like Nanopore and PacBio (long-read technologies) do not have pairs; instead they are sequenced in only one direction! There has been attempt to generate forward and reverse reads with Nanopore technology though the vast majority of long-reads you'll see from Nanopore are described as "1D", meaning strands are sequenced in one direction.

## 1. Download the Data

Let's make a directory for the long read data
```
mkdir athal_long_reads
cd athal_long_reads
```

We will retrieve the long read data from the European Nucleotide Archive. This could take a while, so you may want to do this in a SLURM job.
```
wget ftp://ftp.sra.ebi.ac.uk/vol1/fastq/ERR217/003/ERR2173373/ERR2173373.fastq.gz
```

## 2. Quality Check

Most of the quality checking process for Nanopore reads actually occurs during the base-calling phase. Base-calling is the process by which the disturbance of the electrical signal along the pore (caused by the passage of the DNA molecule through the pore) is translated into bases. The electrical signals are colloquially referred to as "squiggles" and are stored in a .fast5 file which are quite large. Base-calling usually takes place on the computer (typically a GPU) to which the .fast5 files are being written using Nanopore's proprietary software Guppy. Guppy can also apply filters to remove reads that are, most commonly, below a Q7 quality score. A Q7 quality score corresponds to a base being correctly called a bit less than 90% of the time, which means that about 1 out of every 10 bases may be incorrectly called! Long-read technolgoies are inherently "noisy" meaning they must be corrected and polished using multiple itterations of read mapping and/or short reads. This process occurs during the assembly steps. 

If you would like to check out some tools to explore Nanopore reads, try [Nanopack](https://github.com/wdecoster/nanopack) from [deCoster et al. 2018](https://academic.oup.com/bioinformatics/article/34/15/2666/4934939). 

We should, however, filter out reads that are less than 1000bp. We can do this `Seqtk` and `awk`. 

```
module load seqtk
seqtk comp ERR2173373.fastq | awk '{ if (($2 >= 1000)) { print } }' | cut --fields 1 > selected-seq-names.list
seqtk subseq ERR2173373.fastq selected-seq-names.list > filtered_reads.fq 
```

## 3. Assembly with HASLR
We'll be using the long read assembler [`HASLR`](https://github.com/vpc-ccg/haslr) to do the initial assembly. There isn't too much pre-preparation needed, just go ahead and run the program. Remember to submit this as a job! You will probably need a significant amount of time and memory (up to 50 Gb and at least two weeks in run time) and don't forget to change your requested amount of cpus to match the number of threads in the command!
```
module load haslr
mkdir haslr_assembly
haslr.py -o haslr_assembly -t 16 -g 140m -l filtered_reads.fq -x nanopore -s ADDRESS/OF/ARABIDOPSIS/TRIMMED/SHORT/READS/FROM/LAST/TUTORIAL
```

Once again, let's break down what each option means.
* __o__: the output directory for your assembly
* __t__: the number of parallel threads to use during computing
* __g__: estimated genome size of your sample
* __l__: the file storing your long reads
* __x__: the type of long read technology used
* __s__: the file storing your short reads to do error correction and polishing

## 4. Assembly Quality Assessment

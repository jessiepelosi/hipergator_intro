# Arabidopsis Nanopore Long Reads Assembly Tutorial
By the end of this tutorial, you should be able to:
1. Generate a genome assembly using a mixture of long and short reads

This tutorial uses Nanopore long reads and less intensive assemblers (`Flye`, `wtdbg2`, `miniasm`). While assemblers such as `Canu` can produce high-quality assemblies, we think that the computational resources and file management necessary to run it may be a bit much for an introductory tutorial.

# Long Read Assembly
The data we are using is from [Michael et al. 2018](https://www.nature.com/articles/s41467-018-03016-2); while the study uses PacBio, Nanopore, and Illumina reads, we will only be using their Nanopore reads and cleaning with the _Arabidopsis_ short reads used for [the first part of the tutorial](https://github.com/jessiepelosi/hipergator_intro/blob/main/Arabidopsis_genome/Arabidopsis_genome_tut.md). 

| ENA Accession | Instrument       | Layout     | Read Count  |	Bases       |
| ------------- |------------------|------------|-------------|-------------|
|ERR2173373     | Nanopore MinION	 | single     | 300071      | 3.42 Gb     |

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

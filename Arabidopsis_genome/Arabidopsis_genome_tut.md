# Arabidopsis Genome Assembly Tutorial 

By the end of this tutorial you should be able to:
1. Generate a genome assembly from short read sequence data 
2. Generate a genome assembly using both long and short read sequence data 

This is a tutorial for the assembly of the <i> Arabidopsis </i> genome. 
The first part of this tutorial uses SOAPdenovo to assemble the <i> Arabidopsis thaliana </i> genome with short reads, while the second part of the tutorial will use a hybrid assembly approach. 

# Short Read Assembly 

Data for this part of the assembly can be found under BioProject [PRJEB14131](https://www.ncbi.nlm.nih.gov/bioproject/PRJEB14131) on the National Center for Biotechnology Information ([NCBI](https://www.ncbi.nlm.nih.gov/)). We will download just the short read for this part of the tutorial. 

| SRA Accession | Instrument  | Layout     |Insert (bp)| Read Length |	Bases       |
| ------------- |-------------|------------|-----------|-------------|-------------|
|ERR1424597     | HiSeq 2000	 | paired-end | 300       | 100x2       | 17.2 Gb     |


## 1. Download the Data
 
First, make a new directory to hold the data you will be working with. 
```
mkdir Arabidopsis_genome 
cd Arabidopsis_genome
```

You can download data using the `sra` module on HiPerGator like this. Remeber to run all your commands with a SLURM script! 
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

![Per-base sequence quality (the first frame from the output of `FastQC`](https://github.com/jessiepelosi/hipergator_intro/blob/main/Arabidopsis_genome/Arabid_fastqc1.PNG "Per-Base Sequence Quality")

This is the per-base sequence quality averaged across all the sequence reads. You ideally want all your reads above Q28, in the green area, but it is okay if some of them fall into the yellow. 

![Adaptor content](https://github.com/jessiepelosi/hipergator_intro/blob/main/Arabidopsis_genome/Arabid_fastqc2.PNG "Adaptor content")

This is pretty good too, but you can see that FastQC also highlights parts of the analysis (in this case, the adaptor content) that could be improved with the orange exclamanation point. We'll have to remove the adaptors used in Illumina sequencing in the next step. 

## 3. Trim and Quality Filter Reads
First, you will want to remove the adaptors from each read. Adaptors are little sequence fragments that act as a barcode or ID number. They are at the beginning of each read to let the sequencing machine know which reads belong to which sample. Since they are not actually a sequence from the genome of the sample, you should remove them before doing any assembly using a program like [`Trimmomatic`](http://www.usadellab.org/cms/?page=trimmomatic). 

Read filtering/trimming programs like `Trimmomatic` can also remove reads of poor quality. You will want to do that too; this will ensure that you are using untrustworthy data in the rest of the analysis. (Garbage in, garbage out!) 

When running `Trimmomatic`, you can do adaptor trimming and quality filtering all at once. Since our reads are <b>paired-end</b>, you will need to use the paired-end option in `Trimmomatic` and indicate that the corresponding paired-end files should be matched together in your call to the program, like so:

```
module load trimmomatic/0.39
trimmomatic PE ERR1424597_1.fastq ERR1424597_2.fastq \
ERR1424597_1P.fastq ERR1424597_1U.fastq ERR1424597_2P.fastq ERR1424597_2U.fastq \
ILLUMINACLIP:TruSeq2-PE.fa:2:30:10 ILLUMINACLIP:TruSeq3-PE.fa:2:30:10 HEADCROP:5 SLIDINGWINDOW:4:20 MINLEN:36
```

Let's break down what each of these arguments mean:
* <b>PE</b>: Run `Trimmomatic` for paired-end sequences
* <b>phred</b>: A [phred score](https://en.wikipedia.org/wiki/Phred_quality_score) estimates the quality of the read on a logarithmic scale. We specified a minimum phred score of 33, which translates to about a 1 in 2000 chance of a base call in the read being wrong. Standard values for minimum phred scores usually range between 30 and 35.
* <b>unpaired output files</b>: Sometimes `Trimmomatic` will toss out a one read in a pair but not the other. The remaining read becomes unpaired, but can still be used in subsequent analyses, so it outputs them in unpaired read files.
* <b>ILLUMINACLIP</b>: This is the portion of the program call used to trim away the read adaptors.
    * TruSeq2/TruSeq3: This is the file which tells `Trimmomatic` what sequences are part of the adaptors, and therefore what to remove. We have included these in this GitHub repository, but they can also be accessed using the `${HPC_TRIMMOMATIC_ADAPTOR}` variable. 
    * <b>HEADCROP</b>: Removes the first `x` bases from each read regardless of quality. In this case, we'll remove the first five bases. 
    * <b>SLIDINGWINDOW</b>: Removes bases (in groups of four) that have an average quality less than 20. 
    * <b>MINLEN</b>: `Trimmomatic` will toss any reads that are shorter than 36 base pairs after it is done cutting off the adaptor and poor-quality bases on the end of the reads.

Trimmomatic works in order of the command you give it, so you'll get different results if you switch around the order of these commands. 

After you're done trimming and filtering reads, you'll want to check their quality again in `FastQC`.
``` 
module load fastqc
fastqc *P.fastq
```
Take a look at your results -- did the quality of your reads improve?  The output from FastQC for ERR1424597_1P.fastq should look something like this (I've only displayed the first frame of the output file): 

![Filtered reads per-base sequence content](https://github.com/jessiepelosi/hipergator_intro/blob/main/Arabidopsis_genome/Arabid_fastqc3.PNG "Filtered reads per-base sequence content")

You won't be using the files you originally downloaded from SRA or the unpaired reads, so we can compress those so they don't take up too much space on the cluster. You can do this by: 
```
gzip *U.fastq
gzip *1.fastq
gzip *2.fastq
```

## 4. Filter Reads for Contamination
In addition to removing poor quality reads, you will also want to remove any reads that do not belong to the nuclear genome of your sample. ([Steps adapted from this tutorial.](https://sites.google.com/site/wiki4metagenomics/tools/short-read/remove-host-sequences)) You may recall that due to endosymbiosis, organelles (mitochondria, chloroplasts) have their own genomes separate from the nuclear genome. Because mitochondria and chloroplasts are so much more numerous than nuclei, you are inevitably going to get reads from their genomes in your dataset along with your nuclear reads. Since they're from separate genomes, they should be separated from your nuclear reads before assembly.

First, let's download reference sequences for the <i> Arabidopsis </i> [chloroplast](https://www.ncbi.nlm.nih.gov/nucleotide/NC_000932.1) and [mitochondrial](https://www.ncbi.nlm.nih.gov/nucleotide/NC_037304.1) genome.

```
# chloroplast
wget "http://eutils.ncbi.nlm.nih.gov/entrez/eutils/efetch.fcgi?db=nucleotide&id=NC_000932.1&rettype=fasta" -O NC_000932.1.fa

# mitochondrion
wget "http://eutils.ncbi.nlm.nih.gov/entrez/eutils/efetch.fcgi?db=nucleotide&id=NC_037304.1&rettype=fasta" -O NC_037304.1.fa

```

> Note: If you end up doing genome assembly with an organism that doesn't have these references available, you can download a reference from a close relative instead.

Now that you have a reference, you will need to map your reads to the reference genomes. This will tell you which reads match with sequences and where they match in the reference. There are many different mapping programs; we will be using `Bowtie2` since it is optimized for mapping short reads to long references.

The first step when using `Bowtie2` is to build an <b>index</b> for the <b>reference sequences</b> that you are mapping to. This creates a set of files which allow the program to work faster when it actually maps all of your reads.

```
module load bowtie2
bowtie2-build NC_000932.1.fa,NC_037304.1.fa organelle_db
ls
```

In the call to the index builder, we first pass it the two organelle genomes we downloaded and then give it a base name for the resulting index files. You should now see the following files in your directory:

```
organelle_db.1.bt2
organelle_db.2.bt2
organelle_db.3.bt2
organelle_db.4.bt2
organelle_db.rev.1.bt2
organelle_db.rev.2.bt2
```

This means your indexing worked, so you can move on to mapping your reads. If you check the [documentation](http://bowtie-bio.sourceforge.net/bowtie2/manual.shtml#the-bowtie2-build-indexer), `Bowtie2` has a ton of options for adjusting how the read mapping proceeds, but we will be keeping things simple.

```
bowtie2 -p 8 -q --end-to-end -x organelle_db -1 ERR1424597_1P.fastq -2 ERR1424597_2P.fastq --un-conc ERR1424597_filtered -S ERR1424597_mapped_to_org.sam
```

Let's break down the arguments:
* <b>p</b>: Tells the program how many threads you want to run in parallel. Remember to request this many cpus in your job if specify more than 1!
* <b>q</b>: Tells the program that the input files you're giving it are in `fastq` format.
* <b>end-to-end</b>: This specifies the type of alignment we want to do. End-to-end alignment tries to find the best match in the reference sequence for the <i>entire</i> read that it is mapping, rather than just a portion.
* <b>x</b>: The name of the index database of your reference sequences that you created in the previous step.
* <b>1</b>: The forward-facing half of the read pair set to be mapped.
* <b>2</b>: The backwards-facing half of the read pair set to be mapped.
* <b>un-conc</b>: This tells the program to output the unmapped reads to a set of files with the label provided, ERR1424597_filtered.
* <b>S</b>: This tells the program to output all results to a `sam` type file with the name ERR1424597_mapped_to_org.sam

We are actually just interested in the <i>unmapped</i> reads, as these are the nuclear reads that we will want to use in our genome assembly! They are in the `sam` format, though, and we will want to convert them back to `fastq` before moving onto the next step.

```
module load samtools

# Convert sam file to bam file
samtools view ERR1424597_filtered.1 --output-fmt bam > ERR1424597_filtered.1.bam
samtools view ERR1424597_filtered.2 --output-fmt bam > ERR1424597_filtered.2.bam

# Convert bam file to fastq file
samtools fastq ERR1424597_filtered.1.bam > ERR1424597_filtered.1.fastq
samtools fastq ERR1424597_filtered.2.bam > ERR1424597_filtered.2.fastq

# Clean up some of files you won't need anymore
rm ERR1424597_filtered.1
rm ERR1424597_filtered.2
```

## 5. Estimate Genome Size 

Next, you'll want to estimate the genome size using k-mers, which are short substrings of length contained within a DNA sequence. We first have to count k-mers of a certain length (most commonly we use 17 or 21bp k-mers, which can also be written as 17-mer and 21-mer). We'll use [Jellyfish](http://www.genome.umd.edu/jellyfish.html#:~:text=Jellyfish%20is%20a%20tool%20for,of%20k%2Dmers%20in%20DNA.&text=Jellyfish%20is%20a%20command%2Dline,the%20%22jellyfish%20dump%22%20command.) for this step. 
```
module load jellyfish/2.3.0
jellyfish count -C -m 21 -s 1000000000 -t 10 ERR1424597_filtered.*.fastq -o reads.jf
jellyfish histo -t 10 reads.jf > reads.histo
```
Note that Jellyfish is quite computationally expensive! You'll want to make sure to allocate a lot of memory in your SLURM script (this job took XGb of RAM and X hours when I ran it).  

Once you've generated the read histogram from Jellyfish, you can upload the reads.histo file to [GenomeScope](http://qb.cshl.edu/genomescope/). GenomeScope uses a modelling approach to estimate genome heterozygosity, repeat content, and size from sequencing reads using a kmer-based statistical approach. You can also do write a short R script to estimate genome size following [this tutorial](http://koke.asrc.kanazawa-u.ac.jp/HOWTO/kmer-genomesize.html).


## 6. Genome Assembly 

Genome assembly programs employ algorithms which use <b>De Bruijn graphs</b> to figure out what reads fit together into a <b>contig</b>. De Bruijn graphs map the relationship between <b>k-mers</b>, so you will need to tell your assembler the size of the k-mers you want it to use. We will figure this out using the program `kmergenie`.

```
module load kmergenie
# Create list of file inputs for kmergenie to read
ls -1 ERR1424597_filtered.*.fastq > file_list.txt
# Run kmergenie with 8 threads, output files use the prefix kmer_assembl_est
$KMG_DIR/kmergenie file_list.txt -o kmer_assembl_est -t 8
```

Now check the log file from your job. What k-mer size is optimal for this assembly?
You should have gotten *INSERT K-MER SIZE HERE*. Now we can move on to actually assembling the genome.

[SOAPdenovo](https://github.com/aquaskyline/SOAPdenovo2) is a commonly used short-read genome assembly program. It is relatively easy to use once you've gotten the configuration file set up. This is what the config file for this assembly could look like: 
```
# maximal read length
max_rd_len=100
[LIB]
# average insert size
avg_ins=300
# if sequence needs to be reversed
reverse_seq=0
# in which part(s) the reads are used
asm_flags=3
# cutoff of pair number for a reliable connection (at least 3 for short insert size)
pair_num_cutoff=3
# minimum aligned length to contigs for a reliable read location (at least 32 for short insert size)
map_len=32
# a pair of fastq files, read 1 file should always be followed by read 2 file
q1=/path/**LIBNAMEA**/ERR1424597_1P.fastq
q2=/path/**LIBNAMEA**/ERR1424597_2P.fastq
``` 
To run the program (all parts) use the command: (**Don't forget to change the kmer size here!!**)
```
module load soapdenovo
SOAPdenovo all -s file.config -K 33 -R -p 64 -N 135000000 -o athaliana_soap 1> assemb.log 2> athaliana_soap.err 
```
This program also uses a lot of RAM and time to run, so make sure to allocate enough resources in your SLURM script. 

Once your job is done, take a look at the scafStatistics file produced by `SOAPdenovo`. What is the total size of your assembly? What is the mean size of your scaffolds, and how many are larger than 1000 bp? 10,000 bp?

In a perfect world, genome assemblers would produce 1 scaffold for each chromosome. Arabidopsis has 5. We are a *long* way from perfect here! Why do you think the genome assembly is in so many little pieces?


## 7. Assess Assembly Quality
Now that you've assembled the genome, you will want to assess it to figure out how well you did. One way of doing this is counting how many genes are present in your assembly that you expect to be there. To do this, we will use a pipeline called [BUSCO](https://busco.ezlab.org/). Genes assessed by `BUSCO` are conserved across almost all of the Tree of Life, so if they are missing, you can probably assume that it is because your genome assembly is incomplete rather than because your organism doesn't have that gene.

You will need to create a config file called `config.ini` to run `BUSCO` properly. This configuration file will have all the options you would normally pass to the function directly in the command line; a config file for complicated programs like `BUSCO` just keeps all your code looking a bit neater. Here is how your config file will look (feel free to copy and paste):

```
[busco_run]
# Path to the BUSCO dataset
lineage_dataset = embryophyta_odb10
# Which mode to run (genome / proteins / transcriptome)
mode = genome
# How many threads to use for multithreaded steps
cpu = 8
# Force rewrite if files already exist (True/False)
force = True
# BLAST e-value
evalue = 1e-3
# Augustus long mode for retraining (True/False)
long = False
# Local destination path for downloaded lineage datasets
download_path = /ufrc/data/reference/busco/4.0
# Run offline
offline=True
# Download most recent BUSCO data and files
update-data = False

[tblastn]
path = /apps/busco/4.0.6/bin/
command = tblastn

[makeblastdb]
path = /apps/busco/4.0.6/bin/
command = makeblastdb

[augustus]
path = /apps/busco/4.0.6/bin/
command = augustus

[etraining]
path = /apps/busco/4.0.6/bin/
command = etraining

[gff2gbSmallDNA.pl]
path = /apps/busco/4.0.6/bin/
command = gff2gbSmallDNA.pl

[new_species.pl]
path = /apps/busco/4.0.6/bin/
command = new_species.pl

[optimize_augustus.pl]
path = /apps/busco/4.0.6/bin/
command = optimize_augustus.pl

[hmmsearch]
path = /apps/busco/4.0.6/bin/
command = hmmsearch

[sepp]
path = /apps/busco/4.0.6/bin/
command = run_sepp.py

[prodigal]
path = /apps/busco/4.0.6/bin/
command = prodigal
```

Save the above to `config.ini`. There are many other options for running BUSCO, but we will not need them for now since you are doing a simple completeness assessment. You can see some of the other options if you look through the documentation.

Now let's run the program. `BUSCO` is doing a lot of complicated stuff under the hood, so it will need a lot of memory and time. (**Give estimates here**).

```
module load busco
# create a Linux variable so that BUSCO will know where the config file is
export BUSCO_CONFIG_FILE="/ufrc/soltis/kasey.pham/tprat_genome/busco/config.ini"
# create a Linux variable so that BUSCO will know where one of its dependent programs is
export AUGUSTUS_CONFIG_PATH="/ufrc/soltis/kasey.pham/tprat_genome/busco/augustus_config"

busco -f -i athaliana_soap.contig -o athaliana_busco
```

Now take a look at the end of your output log file from the job. `BUSCO` should have printed a summary of what it found. What is the total completeness of your genome? How many complete single copy genes did `BUSCO` recover? How many fragments? How many are missing?

From this and your N50 score during assembly, how good do you think your assembly is?


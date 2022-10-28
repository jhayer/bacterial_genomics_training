# Quality Control and Trimming

## Lecture

<br>

<iframe src="https://docs.google.com/presentation/d/e/2PACX-1vR7ZN5Gc-3q5DR8CWQgyHhjUIc-6uKJnB48lDVMB7tWrt4gpZFipbJRVWfaqdxSw9n_WnMfXoFMZXds/embed?start=false&loop=false&delayms=3000" frameborder="0" width="480" height="389" allowfullscreen="true" mozallowfullscreen="true" webkitallowfullscreen="true"></iframe>

## Practical

In this practical you will learn to import, view and check the quality of raw high throughput sequencing data.

The first dataset you will be working with is from an Illumina dataset.
The sequenced organism is a *Klebsiella pneumoniae* bacterium, a potentially fatal pathogen.
...
[Here](https://www.mdpi.com/2076-2607/9/12/2560/htm) is the article of the study.

## Prepare our computing environment

We will first run the appropriate `srun` command to book the computing cores (cpus) on the cluster.

!!! tip
You need to ask the teacher which partition to use !

```bash
srun -p SELECTED_PARTITION --pty bash -i
```

You are now on a computing node, with computing core reserved for you. That way, you can run commands interactively.
If you want to exit the `srun` interactive mode, press CTRL+D or type `exit`


## Downloading the data

The raw data were deposited at the European Nucleotide Archive, under the accession number PRJEB45084.
You could go to the ENA [website](http://www.ebi.ac.uk/ena) and search for the run with the accession PRJEB45084.


First create a `data/` directory in your home folder for the training

```bash
mkdir data
```

now let's download the reads data:

```bash
cd data
wget ftp://ftp.sra.ebi.ac.uk/vol1/run/ERR595/ERR5951443/K2_Illu_R1.fastq.gz
wget ftp://ftp.sra.ebi.ac.uk/vol1/run/ERR595/ERR5951443/K2_Illu_R2.fastq.gz
```

Let’s make sure we downloaded all of our data using md5sum.

```bash
md5sum K2_Illu_R1.fastq.gz K2_Illu_R2.fastq.gz
```

you should see this

```
e4f7a5de95bbb5d5d58c142afa9bc838  K2_Illu_R1.fastq.gz
a5659b9b4646d15622a1a7a5c41fb22c  K2_Illu_R2.fastq.gz
```

and now look at the file names and their size

```bash
ls -l
```

```
total 172M
-rw-r--r-- 1 hayer 83M Oct 25 16:00 K2_Illu_R1.fastq.gz
-rw-r--r-- 1 hayer 89M Oct 25 16:00 K2_Illu_R2.fastq.gz
```

One last thing before we get to the quality control: those files are writeable.
By default, UNIX makes things writeable by the file owner.
This poses an issue with creating typos or errors in raw data.
We fix that before going further

```bash
chmod u-w *.fastq.gz
```

!!! question
Which difference do you see when you type `ls -s` ?

## Working Directory

First we make a work directory: a directory where we can play around with a copy of the data without messing with the original

```bash
cd ..
mkdir work
cd work
```

Now we make a link of the data in our working directory

```bash
ln -s ../data/*.fastq.gz .
```

The files that we've downloaded are FASTQ files. Take a look at one of them with

```bash
zless K2_Illu_R1.fastq.gz
```

!!! tip
Use the spacebar to scroll down, and type ‘q’ to exit ‘less’

You can read more on the FASTQ format in the [File Formats](file_formats.md) lesson.

!!! question
Where does the filename come from?

!!! question
Why are there R1 and R2 in the file names?

## FastQC

To check the quality of the sequence data we will use a tool called FastQC.

FastQC has a graphical interface and can be downloaded and run on a Windows or Linux computer without installation.
It is available [here](http://www.bioinformatics.babraham.ac.uk/projects/fastqc/).

However, FastQC is also available as a command line utility on the training server you are using.
To run FastQC on our two files

```bash
module load bioinfo/FastQC/0.11.9

fastqc K2_Illu_R1.fastq.gz K2_Illu_R2.fastq.gz
```

and look what FastQC has produced.
If you want to avoid some `scp` or `rsync` commands to the NAS and to your computer, you can find the
html reports for [R1](data/qc/K2_Illu_R1_fastqc.html) and [R2](data/qc/K2_Illu_R2_fastqc.html)

```
ls *fastqc*
```

For each file, FastQC has produced both a .zip archive containing all the plots, and a html report.

Download and open the html files with your favourite web browser.

!!! question
What should you pay attention to in the FastQC report?

!!! question
Which file is of better quality?

Pay special attention to the per base sequence quality and sequence length distribution.
Explanations for the various quality modules can be found [here](http://www.bioinformatics.babraham.ac.uk/projects/fastqc/Help/3%20Analysis%20Modules/).
Also, have a look at examples of a [good](http://www.bioinformatics.babraham.ac.uk/projects/fastqc/good_sequence_short_fastqc.html) and a [bad](http://www.bioinformatics.babraham.ac.uk/projects/fastqc/bad_sequence_fastqc.html) illumina read set for comparison.

You will note that the reads in your uploaded dataset have fairly high quality (>20), which drops a bit towards the end. There are also outlier reads that have very poor quality for most of the second half of the reads.

## Fastp
Most modern sequencing technologies produce reads that have deteriorating quality towards the 3'-end and some towards the 5'-end as well.
Incorrectly called bases in both regions can negatively impact assemblies, mapping, and downstream bioinformatics analyses.

Now we will do some trimming of the reads, for keeping only high quality bases and removing potential sequencing adapters.

Fastp is a tool that uses sliding windows along with quality and length thresholds to determine when quality is sufficiently low to trim the 3'-end of reads and also determines when the quality is sufficiently high enough to trim the 5'-end of reads. It can also discard reads based upon a length threshold.

```bash
module load bioinfo/fastp/0.20.1

fastp -i K2_Illu_R1.fastq.gz -I K2_Illu_R2.fastq.gz -o K2_Illu_R1_trimmed.fastq -O K2_Illu_R2_trimmed.fastq \
  --detect_adapter_for_pe --qualified_quality_phred 20 --cut_front --cut_tail --cut_mean_quality 20  \
  --html "K2_Illu_fastp_report.html"
```

!!! question
What does `fastp` says about what it has done to the dataset?
What are the differences before and after the trimming?

!!! tip
If you want to avoid some `scp` or `rsync` commands to the NAS and to your computer, you can find the
fastp report [here](data/qc/K2_Illu_fastp_report.html)

## MultiQC

[MultiQC](http://multiqc.info) is a tool that aggregates results from several popular QC bioinformatics software into one html report.

Let's run MultiQC in our current directory

```bash
module load bioinfo/multiqc/1.9
multiqc .
```

You can download the report or view it by clicking on the link below

- [multiqc report](data/qc/K2_Illu_multiqc_report.html)

!!! question
What did the trimming do to the per-base sequence quality, the per sequence quality scores and the sequence length distribution?

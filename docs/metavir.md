# Viral Metagenome from a dolphin sample: hunting for a disease causing virus

In this tutorial you will learn how to investigate metagenomics data and retrieve draft genome from an assembled metagenome.

We will use a real dataset published in 2017 in a study in dolphins,
where fecal samples where prepared for viral metagenomics study.
The dolphin had a self-limiting gastroenteritis of suspected viral origin.

## Getting the Data

First, create an appropriate directory to put the data, within your directory for the training:
```bash
mkdir -p dolphin/data
cd dolphin/data
```

You can download them from here:
```
wget ftp://ftp.sra.ebi.ac.uk/vol1/run/ERR193/ERR1938563/Dol1_S19_L001_R1_001.fastq.gz
wget ftp://ftp.sra.ebi.ac.uk/vol1/run/ERR193/ERR1938563/Dol1_S19_L001_R2_001.fastq.gz
```
Alternatively, your instructor will let you know where to get the dataset from.

You should get 2 compressed files:
```
Dol1_S19_L001_R1_001.fastq.gz
Dol1_S19_L001_R2_001.fastq.gz
```

## Quality Control

We will first run the appropriate `srun` command to book the computing cores (cpus) on the cluster.

!!! tip
    You need to ask the teacher which partition to use !

```bash
srun -p SELECTED_PARTITION --pty bash -i
```


We will use FastQC to check the quality of our data, as well as fastp for trimming the bad quality part of the reads.
If you need a refresher on how and why to check the quality of sequence data, please check the [Quality Control and Trimming](qc) tutorial

```bash
mkdir -p dolphin/results
cd dolphin/results
ln -s ../data/Dol1* .

module load bioinfo/FastQC/0.11.9
fastqc Dol1_*.fastq.gz
```

!!! question
    What is the average read length? The average quality?

!!! question
    Compared to single genome sequencing, which graphs differ?

## Quality control with Fastp
We will removing the adapters and trim by quality.

Now we run fastp our read files
```bash
module load bioinfo/fastp/0.20.1

fastp -i Dol1_S19_L001_R1_001.fastq.gz -o Dol1_trimmed_R1.fastq \
 -I Dol1_S19_L001_R2_001.fastq.gz -O Dol1_trimmed_R2.fastq \
 --detect_adapter_for_pe --length_required 30 \
 --cut_front --cut_tail --cut_mean_quality 10
```

Check the html report produced.

!!! question
    How many reads were trimmed?

## Removing the host sequences by mapping/aligning on the dolphin genome

For this we will use Bowtie2.
We have downloaded the genome of Tursiops truncatus from Ensembl (fasta file). Then we have run the following command to produce the indexes of the dolphin genome for Bowtie2 (do not run it, we have pre-calculated the results for you):

```
bowtie2-build Tursiops_truncatus.turTru1.dna.toplevel.fa Tursiops_truncatus
```

Because this step takes a while, we have precomputed the index files, you can get them from here:

```bash
cd dolphin/data
cp /scratch/genesys_training/files/dolphin/Tursiops_truncatus*.bt2 .
```


Now we are ready to map our sequencing reads on the dolphin genome:
```
cd ../results

module load bioinfo/bowtie2/2.3.4.1

bowtie2 -x ../data/Tursiops_truncatus \
-1 Dol1_trimmed_R1.fastq -2 Dol1_trimmed_R2.fastq \
-S Dol1_map.sam --un-conc Dol1_reads_unmapped.fastq --threads 2
```

!!! question
    How many reads have mapped on the dolphin genome?


## Taxonomic classification of the trimmed reads with Kraken2
We will use Kraken2 for the classification of the unmapped reads.

```bash
module load bioinfo/kraken2/2.1.1

mkdir Dol1_reads_unmapped_Kn2nt

kraken2 --db /data/projects/banks/kraken2/nt/21-09/nt/ --memory-mapping --threads 4 --output Dol1_reads_unmapped_Kn2nt/Dol1_reads_unmapped_kn2_nt-res.txt --report Dol1_reads_unmapped_Kn2nt/Dol1_reads_unmapped_kn2_nt-report.txt --report-minimizer-data --paired Dol1_reads_unmapped.1.fastq Dol1_reads_unmapped.2.fastq
```

This command can take very long, therefore we recommend you to prepare a more generic script for running Kraken, with the possibility to run on paired reads or on a single contigs fasta file.

Follow the example below to prepare a `kraken2_nt.sh` script in your `scripts` directory.

```
#!/bin/bash
## SLURM CONFIG ##

#SBATCH --job-name=kn2nt
#SBATCH --output=%x.%j.out
#SBATCH -c 4
#SBATCH --time=96:00:00
#SBATCH -p SELECTED_PARTITION
#SBATCH --mail-type=FAIL,END
#SBATCH --mem-per-cpu=4G

###

HELP="
USAGE: kraken2_nt.sh reads_file1.fastq [reads_file2.fastq]
"
# If we didn't get any arguments, print help and exit
if [[ $# < 1 ]]
then
    echo "$HELP"
    exit 0
fi

# If we have a help flag anywhere among the arguments
for arg in $@
do
    if [ "$arg" == "-h" ] || [ "$arg" == "--help" ]
    then
        echo "$HELP"
    fi
done

##### KRAKEN VARIABLES ####
PATH_DB="/data/projects/banks/kraken2/nt/21-09/nt/"
QUERY_FILE=$1
PREFIX=$(echo $1 | cut -f1 -d.)
DATA_DIR=`pwd`
OUT_DIR="${DATA_DIR}/${PREFIX}_Kn2nt"
mkdir $OUT_DIR

module load bioinfo/kraken2/2.1.1

if [[ $# > 1 ]]
then
# Taxonomic classification with Kraken2 on nt db for paired end reads

    QUERY_FILE2=$2
    kraken2 --db ${PATH_DB} --memory-mapping --threads 4 --output ${OUT_DIR}/${PREFIX}_kn2_nt-res.txt --report ${OUT_DIR}/${PREFIX}_kn2_nt-report.txt --report-minimizer-data --paired ${QUERY_FILE} ${QUERY_FILE2}
else
# one single input file (e.g: scaffolds.fasta)

    kraken2 --db ${PATH_DB} --memory-mapping --threads 4 --output ${OUT_DIR}/${PREFIX}_kn2_nt-res.txt --report ${OUT_DIR}/${PREFIX}_kn2_nt-report.txt --report-minimizer-data ${QUERY_FILE}
fi

```

Once the script is ready, you can run it on the reads as follow:

```bash
sbatch ../../scripts/kraken2_nt.sh Dol1_reads_unmapped.1.fastq Dol1_reads_unmapped.2.fastq
```

!!! question
    Inspect the 2 output files from Kraken and comment.


## Assembly

Megahit will be used for the *de novo* assembly of the metagenome.

```
module load bioinfo/MEGAHIT/1.2.9
megahit -1 Dol1_reads_unmapped.1.fastq -2 Dol1_reads_unmapped.2.fastq -o Dol1_assembly
```

The resulting assembly can be found under `assembly/final.contigs.fa`.

!!! question
    How many contigs does this assembly contain? Is there any long contig?

!!! tip
    Use the command `grep` with the symbol `>` to visualise the fasta sequences headers and count them




## Taxonomic classification of contigs

We will use Kraken again with the same nt database for the classification of the produced contigs.

```bash
# in the results directory, link the contigs file
ln -s Dol1_assembly/final.contigs.fa Dol1_assembly_contigs.fa

# then run you previously made Kraken script using sbatch
sbatch ../../scripts/kraken2_nt.sh Dol1_assembly_contigs.fa
```

!!! question
    Does the classification of contigs produce different results than the classification of reads?

## Extraction of the contig of interest

Let's go for a little practice of your Unix skills!

!!! question
    Find a way to to find the longest contig. They look like this:
```
>k141_1 flag=1 multi=1.0000 len=301
>k141_2 flag=1 multi=1.0000 len=303
```

**hint 1**: all lines containing '>'

**hint 2**: use grep and sed

Once we have this file, we want to sort all the sequences headers by the sequence length (len=X):

```bash
cat Dol1_assembly_contigs.fa | grep ">" | sed s/len=// | sort -k4n | tail -1
```

!!! question
    What is the size of the longest contig?

Now that you have identified the sequence header or id of the longest contig, you want to save it to a fasta file.

```bash
grep -i '>k141_XXX' -A 1 Dol1_assembly_contigs.fa > longest_contig.fasta
```

!!! note
    You need to replace the XXX by the correct header.
    The option -A 1 of grep allows to print 1 line additionally to the matching line (which enables to print the full sequence that corresponds to one line).
    Test with -A 2 (without the redirection to longest_contig.fasta) to see what happens.


Now that you have identified the longest contig, you will check in Kraken results what was the taxon assigned to this contig.

Have a look at the file **Dol1_assembly_contigs_kn2_nt-res.txt**. It is structured in 5 columns:
classification status (C/U), sequence id, assigned TaxID, sequence length, k-mers classification.
For more info about Kraken2, check the [manual](https://github.com/DerrickWood/kraken2/wiki/Manual)

!!! question
    Identify the TaxID of the longest contig and search on NCBI Taxonomy database to which species it corresponds to.

!!! tip
    Note that there is an easy way to extract all the sequences classified at a certain Taxon (using the NCBI TaxID), including or not the sequences classified at "children taxa" of this. A useful Python script is available through [KrakenTools](https://github.com/jenniferlu717/KrakenTools) with a few other tools around Kraken.
    If you have the time, do not hesitate to ask the teacher to help you installing and using it.

## Pavian

We will use Pavian for visualising Kraken2 results for reads and contigs.
The teacher will guide you for this step.
===> Pavian link Here==> add the Kraken reports in the files....


## Genome annotation of the contig of interest

Once the contig to annotate is extracted and saved in the file longest_contig.fasta, we will use Prokka to detect ORFs (Open Reading Frames) in order to predict genes and their resulting proteins.

First, go to Uniprot database and retrieve a set of protein sequences
belonging to adenoviruses. Save the file as `adenovirus.faa` and copy it in
your results directory.

```bash
prokka --outdir annotation --kingdom Viruses \
--proteins adenovirus.faa longest_contig.fasta
```

!!! question
    How many genes and proteins were predicted?

## Visualization and manual curation.

If there is some time left, you can visualise the produced annotation (gff file)
in Ugene or Artemis for example.


## Go further with proteins functions

For the predicted proteins that are left "hypotetical", you can try running
[Interproscan](https://www.ebi.ac.uk/interpro/search/sequence-search) on them to get more information on domains and motifs.

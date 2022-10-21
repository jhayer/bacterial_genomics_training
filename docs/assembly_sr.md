# De-novo Genome Assembly

## Lecture

<iframe src="https://docs.google.com/presentation/d/e/2PACX-1vRKVI_pHGubDWeRPaAO7c9g55DzHMO5Lgd7g7AZXvjB77wAAb-wED82lXgV5P7GPF02k-21YMx8ObaX/embed?start=false&loop=false&delayms=3000" frameborder="0" width="480" height="389" allowfullscreen="true" mozallowfullscreen="true" webkitallowfullscreen="true"></iframe>

## Practical

In this practical we will perform the assembly of _Klebsiella pneumoniae_, using the reads that we have trimmed in the previous Quality Control tutorial.

## Getting the data

We have trimmed the raw reads in the previous step of the training, so we will now assemble them into longer sequences called contigs.

Find your 2 fastq files containing the trimmed reads.

```bash
cd work
ls -l
```


!!! question
How many reads are in the files?

## De-novo assembly

We will be using the SPAdes assembler to assemble our bacterium

```bash
module load bioinfo/SPAdes/3.15.3

spades.py -1 K2_Illu_trimmed_R1.fastq.gz -2 K2_Illu_trimmed_R2.fastq.gz -o K2_spades_assembly -t 4 --isolate
```

This will take some time...

The result of the assembly is in the directory `K2_spades_assembly` under the name `scaffolds.fasta`
First, have a look of the SPAdes output directory.

!!! question
what are the different files there?

Let's make a link of the file containing the assembled scaffolds

```bash
ln -s K2_spades_assembly/scaffolds.fasta K2_spades_scaffolds.fasta
```

and look at it

```bash
head K2_spades_scaffolds.fasta
```

## Quality of the Assembly

QUAST is a software evaluating the quality of genome assemblies by computing various metrics, including

Run Quast on your assembly

```bash
module module load bioinfo/quast/5.0.2
module load bioinfo/bedtools/2.30.0
module load bioinfo/minimap2/2.24

quast.py -o K2_spades_quast -t 2 --conserved-genes-finding --gene-finding \
  --pe1 K2_Illu_trimmed_R1.fastq.gz --pe2 K2_Illu_trimmed_R2.fastq.gz K2_spades_scaffolds.fasta
```

and take a look at the text report

```bash
cat K2_spades_quast/report.txt
```

You should see something like

```
Here add the quast results
....
....
...
```

which is a summary stats about our assembly.
Additionally, the file `K2_spades_quast/report.html`

You can either download it and open it in your own web browser, or we make it available for your convenience:

- [K2_spades_quast/report.html](data/...)

!!! note
N50: length for which the collection of all contigs of that length or longer covers at least 50% of assembly length

!!! question
How well does the assembly total consensus size and coverage correspond to your earlier estimation?

!!! question
How many contigs in total did the assembly produce?

!!! question
What is the N50 of the assembly? What does this mean?

## Assembly Completeness

Although quast output a range of metric to assess how contiguous our assembly is, having a long N50 does not guarantee a good assembly: it could be riddled by misassemblies!

We will run `busco` to try to find marker genes in our assembly. Marker genes are conserved across a range of species and finding intact conserved genes in our assembly would be a good indication of its quality

First we need to check if the bacterial datasets are available for `busco` and which ones are available

```bash
module load bioinfo/BUSCO/5.2.2

busco --list-datasets
```
!!! question
Which dataset should we select?

then we can run `busco` with:

```bash
busco -i K2_spades_scaffolds.fasta -o K2_spades_busco --mode genome --lineage_dataset enterobacterales_odb10
```

!!! question
How many marker genes has `busco` found?

## Course literature

Course literature for today is:

- Next-Generation Sequence Assembly: Four Stages of Data Processing and Computational Challenges: <https://doi.org/10.1371/journal.pcbi.1003345>

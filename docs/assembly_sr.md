# De-novo Genome Assembly

## Lecture

<iframe src="https://docs.google.com/presentation/d/e/2PACX-1vRKVI_pHGubDWeRPaAO7c9g55DzHMO5Lgd7g7AZXvjB77wAAb-wED82lXgV5P7GPF02k-21YMx8ObaX/embed?start=false&loop=false&delayms=3000" frameborder="0" width="480" height="389" allowfullscreen="true" mozallowfullscreen="true" webkitallowfullscreen="true"></iframe>

## Practical

In this practical we will perform the assembly of _Klebsiella pneumoniae_, using the reads that we have trimmed in the previous Quality Control tutorial.

## Prepare our computing environment

We will first run the appropriate `srun` command to book the computing cores (cpus) on the cluster.

!!! tip
You need to ask the teacher which partition to use !

```bash
srun -p SELECTED_PARTITION --cpus-per-task 2 --pty bash -i
```

You are now on a computing node, with computing 2 cpus reserved for you. That way, you can run commands interactively.

If you want to exit the `srun` interactive mode, press CTRL+D or type `exit`

## Getting the data

We have trimmed the raw short-reads in the previous step of the training, so we will now assemble them into longer sequences called contigs.

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

spades.py -1 K2_Illu_R1_trimmed.fastq -2 K2_Illu_R2_trimmed.fastq -o K2_spades_assembly -t 4 --isolate
```

This will take some time...

Because we know that this will take quite some time, we better put the command in a SLURM script that we can `sbatch`.
In that way, we can log out and come back later, the job will keep running.

You can prepare the script `spades_assembly_K2.sh`, this is an example to help you. Be careful and put the correct name in "your_login" for the paths to be correct.

!!! tip
You need to ask the teacher which partition to use !

```
#!/bin/bash
## SLURM CONFIG ##

#SBATCH --job-name=spades
#SBATCH --output=%x.%j.out
#SBATCH --cpus-per-task 4
#SBATCH --time=24:00:00
#SBATCH -p SELECTED_PARTITION
#SBATCH --mail-type=FAIL,END
#SBATCH --mem-per-cpu=4G

##

module load bioinfo/SPAdes/3.15.3

spades.py -1 K2_Illu_R1_trimmed.fastq -2 K2_Illu_R2_trimmed.fastq -o K2_spades_assembly -t 4 --isolate

```

When you think that your script is ready, you can run the job with SLURM using this command (exit the `srun` if it is still active):

```bash
sbatch spades_assembly_K2.sh
```

Then check that your script is "Running" by typing:
```bash
squeue

## to see only your jobs, select the user
squeue -u your_login
```

The result of the assembly is in the directory `K2_spades_assembly` under the name `scaffolds.fasta`
First, have a look of the SPAdes output directory.

!!! question
what are the different files there?
Check the assembly graph (gfa file) with Bandage => you will need to use the `scp` command from your computer.


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

Run Quast on your assembly (type the srun command first if this is not done already)

```bash
module load bioinfo/quast/5.0.2
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
Assembly                    K2_spades_scaffolds
# contigs (>= 0 bp)         3392               
# contigs (>= 1000 bp)      447                
# contigs (>= 5000 bp)      46                 
# contigs (>= 10000 bp)     40                 
# contigs (>= 25000 bp)     24                 
# contigs (>= 50000 bp)     19                 
Total length (>= 0 bp)      7774823            
Total length (>= 1000 bp)   6044278            
Total length (>= 5000 bp)   5470941            
Total length (>= 10000 bp)  5428782            
Total length (>= 25000 bp)  5151491            
Total length (>= 50000 bp)  4973703            
# contigs                   2408               
Largest contig              815843             
Total length                7335510            
GC (%)                      57.04              
N50                         252444             
N75                         3008               
L50                         9                  
L75                         55                 
# total reads               1129033            
# left                      559060             
# right                     559060             
Mapped (%)                  99.46              
Properly paired (%)         98.12              
Avg. coverage depth         35                 
Coverage >= 1x (%)          100.0              
# N's per 100 kbp           0.00      
```

which is a summary stats about our assembly.
Additionally, the file `K2_spades_quast/report.html`

You can either download it and open it in your own web browser, or we make it available for your convenience:

- [K2_spades_quast_report.html](data/assembly/quast_report.html)


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

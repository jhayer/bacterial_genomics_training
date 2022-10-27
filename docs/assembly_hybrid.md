# Hybrid Genome Assembly using long and short reads data



## Practical

In this practical we will perform the assembly of _Klebsiella pneumoniae_, using the short and the long reads that we have trimmed in previous tutorials.

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

We have trimmed the raw short-reads and the long reads in the previous steps of the training, so we will now assemble them into longer sequences called contigs, using a method of hybrid assembly combining both long and short reads, with the [Unicycler](https://github.com/rrwick/Unicycler) pipeline.

Find your 3 fastq(.gz) files containing the trimmed reads for strain K2.

```bash
cd work
ls -l
```

## Unicycler hybrid assembly

As you might ghave noticed, assemblies can take some time. For that reason, we will prepare a `unicycler_assembly_K2.sh` script that we will run on the Slurm job scheduler, using the `sbatch` command.

!!! tip
You need to ask the teacher which partition to use !

```
#!/bin/bash
## SLURM CONFIG ##

#SBATCH --job-name=unicycler
#SBATCH --output=%x.%j.out
#SBATCH --cpus-per-task 4
#SBATCH --time=24:00:00
#SBATCH -p SELECTED_PARTITION
#SBATCH --mail-type=FAIL,END
#SBATCH --mem-per-cpu=4G

##
module load bioinfo/racon/1.4.3
module load bioinfo/unicycler/0.4.4

unicycler -1 K2_Illu_trimmed_R1.fastq.gz -2 K2_Illu_trimmed_R2.fastq.gz --long K2_MinION.nanofilt_trimmed.fastq \
  -o K2_unicycler_assembly -t 4

```

When you think that your script is ready, you can run the job with SLURM using this command (exit the `srun` if it is still active):

```bash
sbatch unicycler_assembly_K2.sh
```

Then check that your script is "Running" by typing:
```bash
squeue

## to see only your jobs, select the user
squeue -u your_login
```

The result of the assembly is in the directory `K2_unicycler_assembly` under the name `assembly.fasta`
First, have a look of the Unicycler output directory.

!!! question
what are the different files there?
Check the assembly graph (gfa file) with Bandage => you will need to use the `scp` command from your computer.


Let's now make a link of the file containing the assembled scaffolds, to simplify the run of Quast and BUSCO

```bash
ln -s K2_unicycler_assembly/assembly.fasta K2_unicycler_scaffolds.fasta
```

and look at it

```bash
head K2_unicycler_scaffolds.fasta
```

## Quality of the Assembly

QUAST is a software evaluating the quality of genome assemblies by computing various metrics, including

Run Quast on your assembly (type the srun command first if this is not done already)

```bash
module load bioinfo/quast/5.0.2
module load bioinfo/bedtools/2.30.0
module load bioinfo/minimap2/2.24

quast.py -o K2_unicycler_quast -t 2 --conserved-genes-finding --gene-finding \
  --pe1 K2_Illu_trimmed_R1.fastq.gz --pe2 K2_Illu_trimmed_R2.fastq.gz K2_unicycler_scaffolds.fasta
```

and take a look at the text report

```bash
cat K2_unicycler_quast/report.txt
```

!!! question
How well does the assembly total size and coverage correspond to your earlier estimation?

!!! question
How many contigs in total did the assembly produce?

!!! question
Has the assembly improved compared with short-reads only assembly? in term of N50 and L50 for example.


Let's now check the completeness in term of essential genes expected with BUSCO

## Assembly Completeness

You can find more info regarding BUSCO in the short-reads assembly tutorial.
Let's run it !

```bash
module load bioinfo/BUSCO/5.2.2

busco -i K2_unicycler_scaffolds.fasta -o K2_unicycler_busco --mode genome --lineage_dataset enterobacterales_odb10
```

!!! question
How many marker genes has `busco` found?
Was this number improved compared to previous assemblies of short-reads only and long-reads only ?

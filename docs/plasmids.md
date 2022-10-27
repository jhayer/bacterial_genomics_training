# Antimicrobial Resistance Genes and Mutations detection

## Before we start

We will now search for plasmids features in the assembled scaffolds/contigs to detect which ones could have originated from a plasmid.

For this, we will use 2 different tools:

- [PlasmidFinder](https://cge.food.dtu.dk/services/PlasmidFinder/), developed at the [Center for Genomic Epidemiology](http://www.genomicepidemiology.org/services/), like many other tools for bacterial genomics analysis. It searched for known plasmids and provide the replicon type.

- [Platon](https://github.com/oschwengers/platon), a tool that detects plasmid-borne contigs within bacterial draft (meta) genomes assemblies. Therefore, Platon analyzes the distribution bias of protein-coding gene families among chromosomes and plasmids. This analysis is complemented by comprehensive contig characterizations followed by heuristic filters.
Platon depends on a custom database based on MPS, RDS, RefSeq Plasmid database, PlasmidFinder db as well as manually curated MOB HMM models from MOBscan, custom conjugation and replication HMM models and oriT sequences from MOB-suite.

## Prepare our computing environment

We will first run the appropriate `srun` command to book the computing cores (cpus) on the cluster.

!!! tip
You need to ask the teacher which partition to use !

```bash
srun -p SELECTED_PARTITION --cpus-per-task 2 --pty bash -i
```

You are now on a computing node, with computing 2 cpus reserved for you. That way, you can run commands interactively.

If you want to exit the `srun` interactive mode, press CTRL+D or type `exit`

## Practical

First, you need to identify in your work directory the fasta file corresponding to the unicycler hybrid assembly.

```bash
cd work
ls -l
```

## PlasmidFinder

We will use the PlasmidFinder container that has all tools needed for running PlasmidFinder

```bash
module load system/singularity/3.6.0

singularity shell path/to/plasmidfiner_container

mkdir K2_plasmidfinder

plasmidfinder.py -i K2_unicycler_contigs.fasta -o K2_plasmidfinder -p /path/to/PF_DB -mp blastn -x
```
Once finished, you can exit the container by typing `exit` or press CTRL+D

!!! question

Inspect the output file and comment.
How many potential contigs of plasmidic origin were identified ?

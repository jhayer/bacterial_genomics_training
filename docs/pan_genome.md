# Pan-Genome Analysis

In this tutorial we will learn how to determine a pan-genome from a collection of isolate genomes.
The pangenome analysis will be based on the genome annotations made in the previous step, that we obtained thanks to Prokka.
The presence/absence of genes will be used by Roary for determining genes belonging to the core genome, and the genes being accessory among the pangenome (not present in all strains).

We will use [Roary](https://sanger-pathogens.github.io/Roary/) for performing this pangenome analysis.

## Prepare our computing environment

We will first run the appropriate `srun` command to book the computing cores (cpus) on the cluster.

!!! tip
    You need to ask the teacher which partition to use !

```bash
srun -p SELECTED_PARTITION --cpus-per-task 2 --pty bash -i
```

You are now on a computing node, with computing 2 cpus reserved for you. That way, you can run commands interactively.

If you want to exit the `srun` interactive mode, press CTRL+D or type `exit`

## Input data

Prokka requires assembled contigs. You can prepare you working directory for this annotation tutorial.

```bash
cd results/annotation
ls -l
```
Put all the .gff files in the same folder (e.g., `./gff`) for running Roary

```bash
mkdir gff
cp K*prokka/*.gff gff/
```

## Pan-genome analysis

We will now run Roary from the gff directory


```bash
module load bioinfo/roary/3.12.0

cd gff
roary -p 4 -o 5_Kp_roary_clust -f 5_Kp_roary -e -n -r -v  *.gff
```

This can take 15-20 min.
Roary will get all the coding sequences, convert them into protein, and create pre-clusters. Then, using BLASTP and MCL, Roary will create clusters, and check for paralogs. Finally, Roary will take every isolate and order them by presence/absence of orthologs. The summary output is present in the `summary_statistics.txt` file.

Additionally, Roary produces a `gene_presence_absence.csv` file that can be opened in any spreadsheet software to manually explore the results. In this file, you will find information such as gene name and gene annotation, and, of course, whether a gene is present in a genome or not.

Inspect the results files produced
```bash
cd 5_Kp_roary
ls -l
```

!!! question
    How many genes are found to be in the core genome?
    How many are shared by several strains but not all (shell)?
    How many are found in only one strain (cloud genes)?


## Plotting the results

Roary comes with a python script that allows you to generate a few plots to graphically assess your analysis output.

First, we need to generate a tree file from the alignment generated by Roary:

```bash
FastTree -nt -gtr core_gene_alignment.aln > my_tree.newick
```

Then we can plot the Roary results with `roary_plots.py`, a community contributed python script to visualise roary results:

```
wget https://raw.githubusercontent.com/sanger-pathogens/Roary/master/contrib/roary_plots/roary_plots.py
module load system/python/3.8.12
pip install seaborn
python roary_plots.py
python roary_plots.py my_tree.newick gene_presence_absence.csv
```

then look at the 3 `/png` files that have been generated
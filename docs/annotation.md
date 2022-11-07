# Genome Annotation

## Lecture

<br>

<iframe src="https://docs.google.com/presentation/d/e/2PACX-1vTERGc6gJyJeGylr6xzXvioMFixfI6x9XIT8QHqC8XIq8cP3KHe6PUuumbMrunSCVlbFhFJaVh2wvMh/embed?start=false&loop=false&delayms=3000" frameborder="0" width="480" height="389" allowfullscreen="true" mozallowfullscreen="true" webkitallowfullscreen="true"></iframe>

After you have de novo assembled your genome sequencing reads into contigs, it is useful to know what genomic features are on those contigs. The process of identifying and labelling those features is called genome annotation.

Prokka is a "wrapper"; it collects together several pieces of software (from various authors), and so avoids "re-inventing the wheel".

Prokka finds and annotates features (both protein coding regions and RNA genes, i.e. tRNA, rRNA) present on on a sequence. Prokka uses a two-step process for the annotation of protein coding regions: first, protein coding regions on the genome are identified using [Prodigal](http://compbio.ornl.gov/prodigal/); second, the function of the encoded protein is predicted by similarity to proteins in one of many protein or protein domain databases. Prokka is a software tool that can be used to annotate bacterial, archaeal and viral genomes quickly, generating standard output files in GenBank, EMBL and gff formats. More information about Prokka can be found [here](https://github.com/tseemann/prokka).

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
cd work
ls -l
mkdir annotation
cd annotation
```

You will link the hybrid assemblies of the 5 Klebsiella strains in this annotation directory:

```bash
ln -s ../K*_unicycler_scaffolds.fasta
```

You will also need a proteins set specific of *Klebsiella pneumoniae* for the annotation. You can go to the [Uniprot/Swiss-Prot](https://www.uniprot.org) database website and search for all the proteins sequences for the organism *Klebsiella pneumoniae*, select only the *reviewed* entries, and download the fasta file of those.

!!! question
How many reviewed protein entries are available for *Klebsiella pneumoniae* in Swiss-Prot ?

For your convenience, we have made the resulting file available. You can copy it by typing the following command:

```bash
cp /scratch/genesys_training/files/swissprot_kp_221107.fasta .
```


## Running prokka

```bash
module load bioinfo/prokka/1.14.6

prokka --force --genus Klebsiella --species pneumoniae \
  --kingdom Bacteria --usegenus --proteins swissprot_kp_221107.fasta \
  --notrna --prefix K2 --outdir K2_prokka K2_unicycler_scaffolds.fasta
```

Once Prokka has finished, examine each of its output files.

* The GFF and GBK files contain all of the information about the features annotated (in different formats.)
* The .txt file contains a summary of the number of features annotated.
* The .faa file contains the protein sequences of the genes annotated.
* The .ffn file contains the nucleotide sequences of the genes annotated.

## Preparing Prokka script for loop

We now want to run Prokka on all our strains, so we can compare the annotations later.

We will prepare a `prokka_kp.sh` script for doing so. Here is an example:


```
#!/bin/bash
## SLURM CONFIG ##

#SBATCH --job-name=prokka
#SBATCH --output=%x.%j.out
#SBATCH --cpus-per-task 2
#SBATCH --time=24:00:00
#SBATCH -p SELECTED_PARTITION
#SBATCH --mail-type=FAIL,END
#SBATCH --mem-per-cpu=4G

#variables
suffix="_scaffolds.fasta"
prefix="${1%$suffix}"

module load bioinfo/prokka/1.14.6

prokka --force --genus Klebsiella --species pneumoniae \
          --kingdom Bacteria --usegenus --proteins swissprot_kp_221107.fasta \
          --notrna --prefix ${prefix} --outdir ${prefix}_prokka ${1}
```

Once your are ready with your script, you can run the loop for `sbatch` it:

```bash
for i in K*scaffolds.fasta; do sbatch prokka_kp.sh $i; done
```


## Visualising the annotation

Artemis is a graphical Java program to browse annotated genomes. Download it [here](http://www.sanger.ac.uk/science/tools/artemis) and install it on your local computer.

Copy the .gff file produced by prokka on your computer, and open it with artemis.

You will be overwhelmed and/or confused at first, and possibly permanently. Here are some tips:

* There are 3 panels: feature map (top), sequence (middle), feature list (bottom)
* Click right-mouse-button on bottom panel and select Show products
* Zooming is done via the vertical scroll bars in the two top panels

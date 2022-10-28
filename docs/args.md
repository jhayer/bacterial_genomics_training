# Antimicrobial Resistance Genes and Mutations detection

## Before we start

We will be using 2 different tools and their associated databases for detecting genes and mutations that could confer an antimicrobial resistance to the bacteria carrying it.
These tools are:

- [AMRFinderPlus](https://www.ncbi.nlm.nih.gov/pathogens/antimicrobial-resistance/AMRFinder/) from NCBI

- [CARD](https://card.mcmaster.ca/home) [RGI](https://card.mcmaster.ca/analyze/rgi) (Resistance Gene Identifier)


## Practical

In this practical you will learn to run these tools and their associated tools, to inspect the output files, and to visualise them in a graphical way.

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

## AMRFinderPlus

First we will activate the AMRFinder container, and check the options and the different organisms datasets available for resistance mutations detection

```bash
module load system/singularity/3.6.0

singularity shell path/to/amrfinder_container

amrfinder -h
amrfinder --list_organisms
```

Now that we have identified the organism option to use, we can run the command
```bash
amrfinder --nucleotide K2_unicycler_scaffolds.fasta -o K2_unicycler_AMRfinder.txt --organism Klebsiella --plus --mutation_all K2_unicycler_AMRfinder_all_mut.txt
```

!!! question

Inspect both output files and comment. Which resistance could be identified ?

Now that we are more used to the command line, we can prepare a generic script `amrfinder_script.sh` to run AMRFinderPlus on all our strains draft genomes, using a loop for `sbatch` the jobs.
Here is an example of script that we will explain, and where you need to change some paths...

```
#!/bin/bash
## SLURM CONFIG ##

#SBATCH --job-name=amrfinder
#SBATCH --output=%x.%j.out
#SBATCH --cpus-per-task 4
#SBATCH --time=24:00:00
#SBATCH -p SELECTED_PARTITION
#SBATCH --mail-type=FAIL,END
#SBATCH --mem-per-cpu=4G

###
HELP="USAGE: amrfinderplus.sh contigs.fasta"
# If we didn't get any arguments, print help and exit
if [[ $# < 1 ]]
then
    echo "$HELP"
    exit 0
fi

#variables
suffix=".fasta"
prefix1="${1%$suffix}"

##
module load system/singularity/3.6.0

singularity run path/to/amrfinder_container \
  amrfinder --nucleotide ${1} -o ${prefix1}_AMRfinder.txt --organism Klebsiella --plus --mutation_all ${prefix1}_AMRfinder_all_mut.txt
```

Once your script seems ready, exit the container, and the `srun` if you are still on the node.
Then you are ready to loop on the unicycler fasta files for running the script with `sbatch`

First, we can test it on one strain:

```bash
sbatch amrfinder_script.sh K1_unicycler_scaffolds.fasta
```

If the test has worked, you can run the loop:

```bash
for i in K*_unicycler_scaffolds.fasta; \
  do sbatch amrfinder_script.sh $i;\
  done
```


## CARD RGI

Now that we feel more familiar with preparing SLURM scripts, we will make `card_script.sh` for running CARD RGI on all samples.

```
#!/bin/bash
## SLURM CONFIG ##

#SBATCH --job-name=card
#SBATCH --output=%x.%j.out
#SBATCH --cpus-per-task 4
#SBATCH --time=24:00:00
#SBATCH -p highmemplus
#SBATCH --mail-type=FAIL,END
#SBATCH --mem-per-cpu=4G

###

HELP="
USAGE: card_script.sh contigs.fasta
"
# If we didn't get any arguments, print help and exit
if [[ $# < 1 ]]
then
    echo "$HELP"
    exit 0
fi

#variables
suffix=".fasta"
prefix1="${1%$suffix}"

##
module load system/singularity/3.6.0

# loading the CARD database
singularity run /path/to/containers/rgi_6.0.0--pyha8f3691_0.sif \
  rgi load --card_json /path/to/databases/CARD/card.json

# Running RGI Main => output json for Heatmap plotting
singularity run /path/to/containers/rgi_6.0.0--pyha8f3691_0.sif \
  rgi main -i ${1} -o ${prefix1}_RGI_main --debug -a BLAST -d wgs -n 4

# Preparing tabular file for import to excel or R
singularity run /path/to/containers/rgi_6.0.0--pyha8f3691_0.sif \
  rgi tab -i ${prefix1}_RGI_main.json

rm ${1}.temp.*
rm ${1}.temp
```

Make a test run on one sample before running the loop.

If the test has worked, you can run the loop:

```bash
for i in K*_unicycler_scaffolds.fasta; \
  do sbatch card_script.sh $i;\
  done
```

Once we have all the json files from CARD RGI, we will place them in the same directory.

```bash
mkdir card_json
mv *_RGI_main.json card_json
```

After that, we can run the RGI program for making the heatmap clustering

```bash
srun -p SELECTED_PARTITION --cpus-per-task 2 --pty bash -i

module load system/singularity/3.6.0

singularity run path/to/card_container

#Generate a heat map from pre-compiled RGI main JSON files, samples clustered by similarity of resistome and AMR genes organized by Drug Class
rgi heatmap -i card_json -cat drug_class -o hm_drug_class -clus samples
```
!!! tip
You can find other ways to cluster/classify the resistance genes, as the following examples:

```bash
#Generate a heat map from pre-compiled RGI main JSON files, samples clustered by similarity of resistome and AMR genes organized by resistance resistance_mechanism
rgi heatmap -i card_json -cat resistance_mechanism -o hm_resistance_mechanism -clus samples
#Generate a heat map from pre-compiled RGI main JSON files, samples clustered by similarity of resistome and AMR genes organized by AMR gene family
rgi heatmap -i card_json -cat gene_family -o hm_genefamily_samples -clus samples

#Generate a heat map from pre-compiled RGI main JSON files, samples clustered by similarity of resistome (with histogram used for abundance of identical resistomes) and AMR genes organized by distribution among samples:
rgi heatmap -i card_json -o cluster_both_frequency -f -clus both
```

Then you can `scp` all the pictures on you computer and inspect.

# Multi-Locus Sequence Typing

In this practical we will perform the genome typing by MLST analysis on the assembly of _Klebsiella pneumoniae_, using the contigs/scaffolds produced by the hybrid assembly in a previous tutorial.


## Prepare our computing environment

We will first run the appropriate `srun` command to book the computing cores (cpus) on the cluster.

!!! tip
    You need to ask the teacher which partition to use !

```bash
srun -p SELECTED_PARTITION --cpus-per-task 2 --pty bash -i
```

You are now on a computing node, with computing 2 cpus reserved for you. That way, you can run commands interactively.

If you want to exit the `srun` interactive mode, press CTRL+D or type `exit`

## Run the MLST program
We should now identify our fasta file containing the contigs from the Unicycler hybrid assembly, produced from long and short reads. This will be the input file for mlst.

```bash
cd results
ls -l
```

Now we will run the MLST tool:

```bash
module load system/singularity/3.6.0
singularity run /path/to/mlst_singularity_container ...

mlst K2_unicycler_scaffolds.fasta > K2_unicycler_scaffolds_mlst.tsv
```

!!! tip
    Ask the teacher for the MLST container path.

Once finished, you can exit the container by typing `exit` or press CTRL+D

!!! question
    Check the output file. What is the sequence type of this strain?

!!! question
    If the (hybrid) assemblies of the other _Klebsiella pneumoniae_ strains are provided, try to make a loop to run the MLST analysis on all.

# Multi-Locus Sequence Typing


## Practical

In this practical we will perform the genome typing by MLST analysis on the assembly of _Klebsiella pneumoniae_, using the contigs/scaffolds produced by the hybrid assembly in a previous tutorial.

We should now identify our fasta file containing the contigs from the Unicycler hybrid assembly, produced from long and short reads. This will be the input file for mlst.

```bash
cd work
ls -l
```

Now we will run the MLST tool:

```bash
module load ...

mlst K2_unicycler_contigs.fasta > K2_unicycler_contigs_mlst.tsv
```

!!! question
Check the output file. What is the sequence type of this strain?

!!! question
If the assembly of the other _Klebsiella pneumoniae_ strains are provided, try to make a loop to run the MLST analysis on all.

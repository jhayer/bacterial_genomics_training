# BASE CALLING
&nbsp;

## objective
Base calling is the process of translating the electronic raw signal (fast5 format) of the sequencer into bases (fastq format).
&nbsp;

&nbsp;

## environment
This process will be carried out on the i-Trop computing cluster.
&nbsp;

&nbsp;

## software
guppy
&nbsp;

<span style="color:green">**QUESTION 1: What is the lasted version of guppy installed on the i-Trop computing cluster?**</span>
&nbsp;

&nbsp;

## location of input files and how to enter them in the command line
long_reads_analyses/fast5/

-i [ --input_path ]
	Path to input files.
&nbsp;

&nbsp;


## location of output files and how to set it in the command line
long_reads_analyses/guppy_output/

-s [ --save_path ]
	Path to save output files.
&nbsp;

&nbsp;

## other required files and how to set them in the command line
Entering information on which flowcell and library preparation kit was used for sequencing is mandatory for launching guppy.
This data is found in configuration files. Each library preparation kit/flowcell pair corresponds to a unique configuration file. 

-c [ --config ]
	Configuration file for application.

To list available configuration files enter `guppy_basecaller ––print_workflows` (to be entered manually!).
The provided data was prepared using the SQK-LSK108 library preparation kit that was sequenced on a MIN106 flowcell.
&nbsp;

<span style="color:green">**QUESTION 2: What is the name of the configuration file guppy needs to basecall the tutorial data?**</span>
&nbsp;

&nbsp;

## command line to launch guppy
```
guppy_basecaller \
-i */fast5 \
-s */guppy_out \
-c dna_*.cfg \
--num_callers 1 --cpu_threads_per_caller 1
```
Let the run continue and go to the directory containing precompiled output files.
It contains pass and fail directories, each containing fastq files.

Check the sequencing_summary file.
&nbsp;

<span style="color:green">**QUESTION 3: What is the minimal threshold for the mean quality score (mean qscore) to pass quality evaluation?**</span>
&nbsp;

&nbsp;

## concatenate fastq files passing quality evaluation
guppy output is often composed of several fastq files for a single sample that must be concatenated. 
`cat *.fastq > sample1.fastq`

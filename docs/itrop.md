# i-Trop cluster

## Architecture of the i-Trop cluster

The IRD Bioinformatic Cluster is composed of a pool of machines reachable through a single entry point. The connections to the internal machines are managed by a master node that tries to ensure that proper balancing is made across the available nodes at a given moment: bioinfo-master.ird.fr

The cluster is composed of:

- 1 master
- 3 nas servers for a 127To data storage
- 32 nodes servers : 8 nodes with 12 cores, 2 nodes with 16 cores, 4 nodes with 20 cores, 11 with 24 cores, 1 with 40 cores with RAM from 48Go to 1To and a GPU serveur with 8 RTX 2080 graphical cards.

[Here](https://github.com/lokman89/bioinformatics_tutorials/blob/temp/schema_cluster_150221.png) is the architecture


## Connect to the cluster via *ssh*

Open the terminal application and type the following command:

`ssh login@bioinfo-master.ird.fr`

with login: your cluster account


**First connection**:

Your password has to be changed at your first connection.

“Mot de passe UNIX (actuel)”: you are asked to type the password provided in the account creation email.

#### Connect to a node in interactive mode and launch commands:

To connect to a node in interactive mode for X minutes, use the following command:

`srun -p short --time=X:00 --pty bash -i`

Then you can launch on this node without using the srun prefix srun
Then type your new password twice.

The session will be automatically closed.

You will need to open a new session with your new password.

## Explore the cluster

The list of nodes in the cluster:

`sinfo -N nodes`

## Choose a partition

*short* partition: short Jobs < 1 day  

*normal* partition: job of maximum 7 days

*long* partition: 7 days< jobs < 45 days

*highmem* partition: jobs with more memory needs

*highmemplus* partition: jobs with more memory needs

*supermem* partition: jobs with much more memory needs

*gpu* partition: need of analyses on GPU cores


#### Choose **node5** (*highmemplus* partition):
`srun -p highmemplus --nodelist=node5 --pty bash -i`

## Data location

All the data used in this training can be found in the *scratch* space of node5.

`ls /scratch/genesys_training/`

## Launching a program on the cluster

### List the program already installed on the cluster:

`module avail`

### Display the description of a particular software

`module whatis module_type/module_name/version`

with module_type: bioinfo or system with module_name: the name of the module.

For example : for the version 1.7 of the bioinformatic software samtools:

`module whatis bioinfo/samtools/1.7`

### Load a particular software version

`module load module_type/module_name/version`

with module_type: bioinfo or system with module_name: the name of the module.

For example : for the version 1.7 of the bioinformatic software samtools:

`module load bioinfo/samtools/1.7`

### Unload a particular software version

`module unload module_type/module_name/version`

with module_type: bioinfo or system with module_name: the name of the module.

For example : for the version 1.7 of the bioinformatic software samtools:

`module unload bioinfo/samtools/1.7`

### Display all the modules loaded

`module list`

### unload all the modules loaded

`module unload`

## Launch a job

A job can be launched interactively using *srun* or via a script using *sbatch*

### Lauch an interactive job

#### Connect to a node in interactive mode and launch commands:

To connect to a node in interactive mode for X minutes , use the following command :

`srun -p short --time=X:00 --pty bash -i`

Then you can launch on this node without using the srun prefix srun

#### Launching commands from the master

The following command allocate computing resources ( nodes, memory, cores) and immediately launch the command on each allocate resource.

`srun + command`

Example:
```
module load bioinfo/FastQC/0.11.9
srun -p highmemplus --nodelist=node5 fastqc -t 2 agropolis_2022/data/untrimmed_lr_fastq/all_guppy.fastq
```

### Launching jobs via a script

The batch mode allows to launch an analysis by following the steps described into a script

Slurm allows to use different types of scripts such as bash, perl or python.

Slurm allocates the desired computing resources and launch analyses on these resources in background.

To be interpreted by Slurm, the script should contain a specific header with all the keyword **#SBATCH** to precise the Slurm options. .

Slurm script example:
```
#!/bin/bash
## Define job's name
#SBATCH --job-name=flye
## Define the number of tasks
#SBATCH --ntasks=1
## Choose the node
#SBATCH --nodelist=node11
## Choose partition
#SBATCH --partition=long
#Define the timelimit
#SBATCH --time=400:00:00
## Define the number of cpus
#SBATCH --cpus-per-task=8
## Define the amount of ram per cpu
#SBATCH --mem-per-cpu=6000
```

To launch an analysis use the following command:

`sbatch script.sh`

with `script.sh` the name of the script to use.

#### Check job status
`sacct -S 2020-11-2 -u galal --format=jobid,jobname,user,submit,start,end,state,NNodes,CPUTimeRAW,comment,Timelimit,TotalCPU,CPUTime,MaxDiskWrite,NodeList`

For checking all the jobs of a certain user:

`squeue -u username`

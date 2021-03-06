pbwa
====

Parallel implementation of the BWA DNA alignment software

### Introduction

pBWA is a parallel implementation of the popular software BWA. It was developed by modifying the BWA source code with the OpenMPI C library on the SHARCNET. pBWA has been successfully tested on other systems with the most basic OpenMPI installs. pBWA currently implements three commands from BWA: aln, samse, and sampe. pBWA retains and improves upon the multithreading provided by BWA while adding efficient parallelization for the above listed functions. pBWA has shown that its wall-time speedup is bounded only by the size of the parallel system available as pBWA can run on any number of nodes and/or cores simultaneously.


### Requirements

pBWA requires a multi-node (or multi-core) *nix system with a parallel scheduler alongside the OpenMPI C library in order to compile and run. Each processor executing pBWA requires as much RAM as BWA requires given the same dataset and parameters. pBWA requires an index generated by BWA for each genome it wishes to align to.


### How to Use
#### Overview
pBWA can be executed as long as there is a pre-existing index created by BWA's index command. Below are generic commands, and the other tabs will show examples using a parallel scheduler. The scheduling commands will match those used on the SHARCNET.

    ./pBWA aln -f SaiPrefix /path/to/Index.fa /path/to/Read_1.fq [/path/to/Read_2.fq]
    [./pBWA aln -f SaiPrefix2 /path/to/Index.fa /path/to/Read_2.fq]
    ./pBWA sampe -f SamPrefix /path/to/Index.fa SaiPrefix SaiPrefix[2] /path/to/Read_1.fq /path/to/Read_2.fq 

A list of parameters available for the above commands can be found in the BWA manual.


#### Standard MPI
Lack of multithreading is best for systems with sufficient RAM (>=4GB/core). This ensures that all stages (sampe, aln) will receive the maximum amount of parallelism. Note that each stage of pBWA MUST be executed with the same number of parallel processors, although the number of threads can differ.

    sqsub -q mpi -n 240 -r 1h --mpp 4G ./pBWA aln -n 2 -f Aln Index/hg18.fa Reads/Read_1.fq Reads/Read_2.fq
    sqsub -q mpi -n 240 -r 1h --mpp 4G ./pBWA sampe -f Aln Index/hg18.fa Aln Aln Reads/Read_1.fq Reads/Read_2.fq 

The -q flag tells us it is an MPI program. The -n flag tells us we want 240 parallel processors executing pBWA. The -r flag is a system requested time limit and the --mpp flag tells the system we need each parallel process to be given 4GB of RAM.


#### MPI + Multithreading
Using a combination of multithreading and parallelism is best for systems lacking sufficient RAM. The number of processors and threads used depends on the system. Play around and see what works best for yours! Below is an example for the Orca cluster (24 cores/node and 1.33GB RAM/core) on the SHARCNET. Note that some systems will not support a combination of parallelism and multithreading and attempts to do so can lead to unexpected results such as segmentation faults.

    sqsub -q mpi -n 40 -N 10 --mpp 8G -r 1h -f noaffinity ./pBWA aln -n 2 -f Aln -t 6 Index/hg18.fa ...
    sqsub -q mpi -n 40 --mpp 4G -r 1h ./pBWA sampe... 

The -N flag tells us we want our 40 processors to be spread out evenly over 10 nodes. Requesting 8G of memory (more than we need) ensures that the 4 processors per node are taking up all of the RAM on that node, ensuring the remaining 20 cores will be available for threading. This allows each process to spawn 6 threads for a total of 240 threads of execution. Note that for sampe, we are only able to run it with 40 threads (processes) of execution due to the lack of multithreaded availability. Future releases of pBWA are expected to introduce multithreading for samse/sampe.


### How to Cite
<a href="http://www.scitechnol.com/JABCB/JABCB-1-101.pdf" target="_blank">Peters D, Luo X, Qiu K, Liang P (2012) Speeding Up Large-Scale Next Generation Sequencing Data Analysis with pBWA. J Biocomput 1:1.</a>

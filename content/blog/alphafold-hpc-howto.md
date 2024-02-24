+++
title = 'Running AlphaFold2 on a HPC cluster'
author = 'by James Lingford'
date = 2024-02-20T14:08:51+11:00
draft = true
toc = true
math = true
tags = ["AlphaFold2", "HPC", "how-to", "colabfold", "ssh", "singularity"]
categories = ["How to", "AlphaFold2"]
+++

For most researchers, running AlphaFold2 (AF2) on their personal computer isn't feasible due to the demanding GPU requirements.
One option to meet the GPU requirements might be to pay for and/or build a custom high-end gaming PC to run AF2 locally.
Another option is to pay for Google or AWS cloud services to run AF2 remotely.
But if you as a researcher have access to a university high performance computing (HPC) cluster, this is likely the best overall long-term solution to running AF2, especially when it comes to running large AF2_multimer predictions.
However, using a HPC cluster comes with a bit of a learning curve (especially for biologists like myself with limited computational experience...).
What follows in this post is a short overview of different options available to getting AF2 setup and running on your HPC cluster.

## Option 1: Using a Signularity container

Singularity containers are basically Docker containers that are HPC "friendly".
The issue with Docker containers is they require administration privileges, so using them on a shared computing resource like a cluster is obviously a no-go.
Singularity was developed to give cluster users all the advantages of containerized environments for their code like Docker does, but without the security concerns of everyone using Docker on a shared cluster.
Furthermore, using Singularity is highly similar to using Docker, so users don't need to learn a whole new piece of software.
Singularity is also compatible with containers built in Docker, so there's no need to build a container specifically using Singularity.

### Setup

**1. Check Singularity is installed on your cluster**:
The administrators of your university HPC cluster should have already installed Singularity given how popular it is at this point.
Regardless, check the clusters official online documentation.
Log in to your cluster and check with:

```bash
module load singularity
module list
# the output should list the singularity version
```

**2. Change the Singularity Cache Directory location**:
When you pull a container into Singularity it gets stored under `.cache`.
By default, `.cache` is located in your home directory.
This can be an issue given how large Singularity/Docker containers can be, and disk space on user login nodes is usually in short supply.
I recommend setting the `.cache` location to a shared project directory with more abundant disk space.
Simply:

```bash
module load singularity
SINGULARITY_CACHEDIR=/home/user/path/to/project/dir
```

You can also set this option in your `.bashrc` or `sbatch` script by including the line:

```bash
export SINGULARITY_CACHEDIR="/home/user/path/to/project/dir"
```

The `.cache` directory should now appear in the designated location.


### AlphaFold2 Singularity container



#### Note: Updating AF2 databases



### ColabFold Singularity container



## Option 3: LocalColabFold local install



## Option 4: Running a Jupyter notebook



## References and further reading



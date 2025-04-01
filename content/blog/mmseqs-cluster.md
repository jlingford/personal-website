---
title: 'Running MMseqs2 cluster and splitting clusters into separate fasta files'
author: 'by James Lingford'
date: 2025-03-30T18:04:28+11:00
draft: false
toc: true
math: true
tags: []
categories: []
---

[MMseqs2](https://github.com/soedinglab/MMseqs2) can cluster protein sequences.
Clustering is useful for finding reducing the complexity of a sequence dataset (such as by finding representative sequences for a cluster), and for filtering out sequences that would likely fit poorly in an MSA.
It's an amazingly well-crafted command-line tool that includes great [documentation](https://github.com/soedinglab/MMseqs2/wiki) and even a helpful [tutorial](https://github.com/soedinglab/MMseqs2/wiki/Tutorials).
That said, once *very minor* criticism I have of MMseqs2 is that it was written by people who are very smart, and not every user (i.e. me), is as smart as they are.
A problem that I encountered was that **I wanted to simply acquire separate fasta files for each cluster**, but I did not know how to do that, nor were there any guides online as far as I could gather.

So, here is how I run MMseqs2 clustering, plus a couple of different ways to retrieve fasta files for each sequence cluster.

## Running MMseqs2 cluster in one go

First things first, set-up and organise your directory structure.
MMseqs2 requires a lot of different intermediate databases, so best to keep them all separate.
And I like to keep different fasta files and projects separate in their own child directories too.

```bash
name=your_protein_of_interest # for example

mkdir -p {seqDB,clustDB,alignDB,fastaclustDB,repclustDB,outfiles}/${name}
mkdir tmp
```

Second, create the sequence database out of your fasta file for clustering.
MMseqs2 requires this "seqDB" for all downstream processes.
Obviously make sure you have MMseqs2 installed (I use the bioconda package).

```bash
mmseqs createdb \
    ./YOUR_FASTA_FILE.faa \
    ./seqDB/${name} \
    --dbtype 1
```

Now the main part: clustering.
I like to run all the following steps in one go from a bash script (including the "optional" steps), since it can be quite tedious to run each step individually from the command line.
In my experience, these steps all run extremely fast on a regular laptop that uses SSD disk space, even with a large input fasta file (and hence large seqDB), and even on the highest sensitivity setting (`-s 7.5`).
Sensitivity can be increased further by increasing the `--max-seqs` in the cluster prefilter step.
Change the parameters to what you want too.

```bash
# variables
name=your_protein_of_interest # for example
COV=0.85 # for example
SEQID=0.20 # for example
suffix=${COV##*.}
suffix2=${SEQID##*.}

# step 1 (done): create sequenceDB out of fastas

# step 2: create cluster database
mmseqs cluster \
    ./seqDB/${name}-DB \
    ./clustDB/${name}/${name}-cov${suffix}-id${suffix2} \
    ./tmp \
    --remove-tmp-files 1 \
    -c ${COV} \
    --min-seq-id ${SEQID} \
    --max-seqs 20 \
    -s 7.5 \
    -e 0.0001 \
    -a 1 \
    --cov-mode 0 \
    --cluster-mode 0

# step 3: output tsv of clusters
mmseqs createtsv \
    ./seqDB/${name}-DB \
    ./clustDB/${name}/${name}-cov${suffix}-id${suffix2} \
    ./outfiles/${name}/${name}-cov${suffix}-id${suffix2}-clust.tsv

# step 4: convert clusterDB to alignmentDB
mmseqs align \
    ./seqDB/${name}-DB \
    ./seqDB/${name}-DB \
    ./clustDB/${name}/${name}-cov${suffix}-id${suffix2} \
    ./alignDB/${name}/${name}-cov${suffix}-id${suffix2} \
    -a 1

# step 5: convert alignmentDB to alignment output file
mmseqs convertalis \
    ./seqDB/${name}-DB \
    ./seqDB/${name}-DB \
    ./alignDB/${name}/${name}-cov${suffix}-id${suffix2} \
    ./outfiles/${name}/${name}-cov${suffix}-id${suffix2}-all_clusters.tsv \
    --format-mode 4 \
    --format-output query,target,evalue,pident,tseq

# creating fasta files...

# optional step 6: convert cluster to fasta files (2 steps)
# optional step 6.1: convert clusterDB to fasta-clusterDB
mmseqs createseqfiledb \
    ./seqDB/${name}-DB \
    ./clustDB/${name}/${name}-cov${suffix}-id${suffix2} \
    ./fastaclustDB/${name}/${name}-cov${suffix}-id${suffix2}

# optional step 6.2: covert fasta-clusterDB to fasta file
mmseqs result2flat \
    ./seqDB/${name}-DB \
    ./seqDB/${name}-DB \
    ./fastaclustDB/${name}/${name}-cov${suffix}-id${suffix2} \
    ./outfiles/${name}/${name}-cov${suffix}-id${suffix2}.faa

# optional step 7: retrieve representative sequences from clusters (2 steps)
# optional step 7.1: convert clusterDB to cluster_representativesDB
mmseqs createsubdb \
    ./clustDB/${name}/${name}-cov${suffix}-id${suffix2} \
    ./seqDB/${name}-DB \
    ./repclustDB/${name}/${name}-cov${suffix}-id${suffix2}-clust_representatives

# optional step 7.2: convert cluster_representativesDB to fasta file
mmseqs convert2fasta \
    ./repclustDB/${name}/${name}-cov${suffix}-id${suffix2}-clust_representatives \
    ./outfiles/${name}/${name}-cov${suffix}-id${suffix2}-clust_representatives.faa

echo "done!"
```

The .tsv output from "step 5" is gives your useful data about each cluster and is needed for generating the split fasta files for each cluster (see below).
This .tsv file can be modified with the `--format-output` parameter (see the [documentation here](https://github.com/soedinglab/MMseqs2/wiki#custom-alignment-format-with-convertalis)), though I find the `query, target, evalue, pident, tseq` information is all I care about.
The optional step 6 noted above will be used as input for an alternative way to split fasta files by cluster (discussed below).
Optional step 7 is also very useful for downstream analysis, but not the focus of this blog post.

## Retrieving separate fasta files with Awk and Bash

Now we can generate a separate fasta file for each cluster of sequences.
Each fasta file will contain all the sequences belonging to one cluster, and the fasta file will be named after the representative sequence ID, and contain a count of how many sequences are in the fasta file.

These fasta files are generated with Awk and Bash using the `-all_clusters.tsv` file from step 5 above.

```bash
# create a new directory home for your split fasta files
if [[ ! -d ./outfiles/${name}/${name}-cov${suffix}-id${suffix2} ]]; then
    mkdir -p ./outfiles/${name}/${name}-cov${suffix}-id${suffix2}-splitfastas
fi

# split tsv into separate fasta files based on shared cluster name ($1)
awk -F'\t' '
    NR>1{
        gsub(/[^a-zA-Z0-9]/, "-", $1)
        printf ">%s\n%s\n", $2,$NF > $1 ".faa"
}' ./outfiles/${name}/${name}-cov${suffix}-id${suffix2}-all_clusters.tsv

# move all renamed fastas to its new home.
# CAUTION: be sure no other fasta files are present in the current directory
mv *.faa ./outfiles/${name}/${name}-cov${suffix}-id${suffix2}-splitfastas

# cd to the new directory
cd !:2

# rename file with number of fastas in headers
# NOTE: be sure to have "perl-rename" installed on your system, some Linux distros don't come with it by default
for file in *.faa; do
    COUNT=$(grep -c "^>" ${file})
    perl-rename "s/^/${COUNT}-seqsincluster-/" ${file}
done

# Optional:
# keep only the top N number of split fasta file clusters, and remove the rest
N=10 # for example
ls -la | grep .faa | awk '{print $NF}' | sort -gr | tail -n+$((${N} + 1)) | xargs rm
```

Going step by step through this script:

1. Create a new directory for the split fasta files, so as to not clutter up the "outfiles" directory.
2. Use Awk to read the .tsv file:
    * `NR>1`: skip the header row (only necessary if `--format-mode 4` was used in step 5).
    * `gsub`: rename each fasta ID to something that won't mess with generating new output files (i.e. removing problematic "." and "/" symbols).
    * `printf`: print the second and last columns (which are the fasta ID's and the sequences) in a fasta format, and output it to a new fasta file named after the cluster representative ID.
3. Move the files to the new directory, and cd to said directory.
4. Rename the split fasta files to include the total number of sequences in each file with `grep` and `perl-rename`. I find this step useful for the next optional step.

This will likely generate hundreds of fasta files, depending on how many sequences you started with and how they all cluster with your chosen parameters.
Many of these fasta files may contain only a handful of sequences, so it's probably useful to focus on only the fasta files with the biggest number sequences, as these are represent the main clusters of interest.
To do this, I like to keep only the top 10 biggest clusters, which can be done easily by sorting the fasta files by how many sequences they contain and removing the rest (see the final optional commands above).

## Retrieving separate fasta files with Python

One thing that annoyed me with the above method was that the fasta file generation step using Awk is quite slow.
While Awk is a fantastic command-line tool for simple things, it is not optimised for writing to new files.
This is why I turned to Python in the following steps, which is somehow faster than Awk in this regard.

Run the "optional step 6" MMseqs2 steps from before.

```bash
# optional step 6: convert cluster to fasta files (2 steps)
# optional step 6.1: convert clusterDB to fasta-clusterDB
mmseqs createseqfiledb \
    ./seqDB/${name}-DB \
    ./clustDB/${name}/${name}-cov${suffix}-id${suffix2} \
    ./fastaclustDB/${name}/${name}-cov${suffix}-id${suffix2}

# optional step 6.2: covert fasta-clusterDB to fasta file
mmseqs result2flat \
    ./seqDB/${name}-DB \
    ./seqDB/${name}-DB \
    ./fastaclustDB/${name}/${name}-cov${suffix}-id${suffix2} \
    ./outfiles/${name}/${name}-cov${suffix}-id${suffix2}.faa

echo "done!"
```

This will output one big fasta file containing all the sequences in each cluster.
The start of each cluster is denoted by two consecutive and identical ID lines, where the identical ID is the sequence representative of each cluster, like so:

```
>Q0KJ32 
>Q0KJ32
MAGA....R
>C0W539
MVGA....R
>D6KVP9
MVGA....R
>D1Y890
MVGV....R
>E3HQM9 
>E3HQM9
MCAT...Q
>Q223C0
MCAR...Q
```

It would be nice to split this one big fasta file into many smaller files, where each new smaller fasta file is an individual cluster.
The following Python script will do exactly that.

```python

#!/usr/bin/env python3
"""
Split mmseqs clustered fasta files into separate fasta files for each cluster.
The mmseqs fasta outputs denote different clusters with identical fasta headers ">" on two consecutive lines.
This script scans the fasta file for those identical and consecutive lines, and splits them into separate files.

Usage:
python3 ./split_clusters.py [YOUR_FASTA_FILE.faa]
"""

import re
import argparse
import pathlib

# accept one positional argument (the input file)
parser = argparse.ArgumentParser(
    description="take input fasta file from mmseqs clustering and split them into separate fasta files"
)
parser.add_argument(
    "input",
    metavar="FILE",
    type=argparse.FileType("r"),
    help="provide a path/to/file.faa",
)
args = parser.parse_args()

# make new output dir with pathlib
pathtofile = args.input.name
new_dir_name = pathtofile.rpartition(".")[0] + "-split_fastas"
dir = pathlib.Path(new_dir_name)
dir.mkdir(parents=True, exist_ok=True)

# initialize variables
seqs = 0
cluster = 0
passed_first_cluster = False
filename = None
cluster_name = None
prev_line = None
arr = []

# read and process input file
with args.input as infile:
    for line in infile:
        # if two consecutive lines are identical, write output file and reset variables for next cluster of fastas
        if line == prev_line:
            filename = (
                f"{seqs - 1 if seqs > 1 else 1}-seqs_in_cluster-{cluster_name}.faa"
            )
            # write all lines except the previous line to new file
            if passed_first_cluster is True:
                outpath = dir / filename
                with open(outpath, "w") as outfile:
                    for a in arr[:-1]:
                        outfile.write(a)
            # reset array, make new name for next filename
            arr = []
            cluster_name = re.sub(r"[-]", "_", line.rstrip()) # replace problematic symbols in ID name with file friendly ones
            cluster_name = re.sub(r"[./]", "_", line.rstrip())
            cluster_name = re.sub(r"[>]", "", cluster_name)
            seqs = 0
            passed_first_cluster = True

        # collect line into an array
        arr.append(line)

        # count number of fasta headers in cluster chunk
        if line.startswith(">"):
            seqs += 1

        # iterate to next line
        prev_line = line

    # write last fasta sequences to final file
    if filename is not None:
        filename = f"{seqs - 1 if seqs > 1 else 1}-seqs_in_cluster-{cluster_name}.faa"
        outpath = dir / filename
        with open(outpath, "w") as outfile:
            for a in arr[:-1]:
                outfile.write(a)

```

It will take the big fasta file as a positional argument, create a new child directory to output all the split fasta files to, and split the input fasta file into smaller fasta files when it reads two consecutive and identical fasta header ID lines.
This is probably not the most memory efficient way to do this task, since the script stores each sequence from a new cluster in an array before writing it to a file, but it gets the job done better than the Awk/Bash method in my opinion.
Like the previous Awk/Bash script, this Python script will name the new fasta files by the number of sequences in each cluster plus the representative sequence ID of each cluster.

Hope this saves others some time.

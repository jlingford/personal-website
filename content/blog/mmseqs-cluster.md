---
title: 'Tips for running MMseqs2 cluster and retrieving fasta files of clustered sequences'
author: 'by James Lingford'
date: 2025-03-30T18:04:28+11:00
draft: true
toc: true
math: true
tags: []
categories: []
---

[MMseqs2](https://github.com/soedinglab/MMseqs2) can cluster protein sequences.
Clustering is useful for finding reducing the complexity of a sequence dataset (such as by finding representative sequences for a cluster), and for filtering out sequences that would likely fit poorly in an MSA.
It's an amazingly well-crafted command-line tool that includes great [documentation](https://github.com/soedinglab/MMseqs2/wiki) and even a helpful [tutorial](https://github.com/soedinglab/MMseqs2/wiki/Tutorials).
That said, once *very minor* criticism I have of MMseqs2 is that it was written by people who are very smart, and not every user (i.e. me), is as smart as they are.
A problem that I encountered was that after I had generated sequence clusters, **I wanted to simply acquire separate fasta files for each cluster**, but I did not know how to do that, nor were there any guides online as far as I could gather.

So, here is how I run MMseqs2 clustering, plus a couple of different ways to retrieve fasta files for each sequence cluster.

## Running MMseqs2 cluster in one go

```bash
name=your_protein_of_interest # for example

mkdir -p {seqDB,clustDB,alignDB,fastaclustDB,outfiles}/${name}
mkdir tmp
```

```bash
mmseqs createdb \
    ./your_fasta_file.faa \ # for example
    ./seqDB/${name} \
    --dbtype 1
```

```bash
#variables
name=your_protein_of_interest # for example
COV=0.85 # for example
SEQID=0.20 # for example
suffix=${COV##*.}
suffix2=${SEQID##*.}


#step 1 (done): create sequenceDB out of fastas

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

## Retrieving separate fasta files with Awk and Bash

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

# cd to that directory
cd !:2

# rename file with number of fastas in headers
# NOTE: be sure to have "perl-rename" installed on your system, some linux distros don't come with it by default
for file in *.faa; do
    COUNT=$(grep -c "^>" ${file})
    perl-rename "s/^/${COUNT}-seqsincluster-/" ${file}
done

# Optional:
# keep only the top N number of split fasta file clusters, and remove the rest
N=10 # for example
ls -la | grep .faa | awk '{print $NF}' | sort -gr | tail -n+$((${N} + 1)) | xargs rm


```

## Retrieving separate fasta files with Python

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

```python

#!/usr/bin/env python3
"""
Split mmseqs clustered fasta files into separate fasta files for each cluster.
The mmseqs fasta outputs denote different clusters with identical fasta headers ">" on two consecutive lines.
This script scans the fasta file for those identical and consecutive lines, and splits them into separate files.
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

# set output dir with pathlib
pathtofile = args.input.name
new_dir_name = pathtofile.rpartition(".")[0] + "-split_fastas/"
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
            # write all lines minus the previous line to new file
            if passed_first_cluster is True:
                outpath = dir / filename
                with open(outpath, "w") as outfile:
                    for a in arr[:-1]:
                        outfile.write(a)
            # reset array, make new name for next filename
            arr = []
            cluster_name = re.sub(r"[-]", "_", line.rstrip())
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

    # write last fastas to final file
    if filename is not None:
        filename = f"{seqs - 1 if seqs > 1 else 1}-seqs_in_cluster-{cluster_name}.faa"
        outpath = dir / filename
        with open(outpath, "w") as outfile:
            for a in arr[:-1]:
                outfile.write(a)

```

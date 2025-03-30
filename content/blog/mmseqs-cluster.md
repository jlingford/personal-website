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
That said, once *very minor* criticsm I have of MMseqs2 is that it was written by people who are very smart, and not every user (i.e. me), is as smart as they are.
A problem that I encountered was that after I had generated sequence clusters, **I wanted to simply acquire separate fasta files for each cluster**, but I did not know how to do that, nor were there any guides online as far as I could gather.

So, here is how I run MMseqs2 clustering, plus a couple of different ways to retrieve fasta files for each sequence cluster.

## Running MMseqs2 cluster in one go

## Retrieving separate fasta files with Awk and Bash

## Retrieving separate fasta files with Python

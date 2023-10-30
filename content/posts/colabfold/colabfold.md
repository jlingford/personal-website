+++
title = 'How to run the ColabFold notebook on a HPC over ssh'
date = 2023-10-29T12:24:45+11:00
draft = false
toc = true
tags = ['alphafold', 'HPC', 'colabfold', 'ssh', 'protein structure']
categories = ['computational biology', 'how to']
+++

![image](/images/a.png)

## Introduction

ColabFold is amazing. If you haven't heard of it yet, it's basically a user friendly
implementation of AlphaFold2 with a significantly faster multiple sequence alignment (MSA)
step. You can run it straight from a [Google Colab notebook](https://colab.research.google.com/github/sokrypton/ColabFold/blob/main/AlphaFold2.ipynb) for free,
and it gives you numerous options to change the Alphafold parameters without messing around
with scripting. Furthermore, ColabFold outputs beautiful publication ready plots in .png form of the MSA, pLDDT, and PAE data,
as well as giving you a well organised .zip folder with all the results and .pdb models.
The ColabFold authors (namely Sergey Ovchinnikov) have also made [other Colab notebooks](https://github.com/sokrypton/ColabFold) that
allow users to accomplish tasks such as running RoseTTAFold2, OmegaFold, and AlphaFold2 in
batch.

### The problem of relying of Google for GPUs 

The issue, however, is that GPUs are expensive, and you can quickly chew through the free GPU quota Google sets for
users after a few ColabFold jobs. One way around this is to keep registering free Google
accounts to get more free quotas. However, this is tedious, and still comes with the risk
of relying on free GPU instances, which Google reserves the right to terminate at any
point, even mid-job. Another option is to pony up and spend some cash on GPU time with a
Colab Pro account. I've tried this out of curiosity and spent $15 AUD for a "pay as you
go" plan with a quota of 100 compute units. I spent all these 100 units during a single
job, and it still wasn't enough units to get the job finished before it terminated on me.
So that was $15 down the drain... A third solution is to setup a local runtime on your own
computer and run the ColabFold job through your personal GPU resources. This option is
getting better because it's free (ignoring your power bill) and you're not relying on
Google's fickle GPU allocation system. However, this still isn't ideal for myself and most
people, who only have a standard laptop GPU available (I have a NVIDIA GeForce GTX 1650 Ti running on my Dell XPS 17 9700).
These sorts of GPUs aren't powerful enough to run Alphafold/ColabFold in a reasonable amount of time
and will likely most of your day depending on the size of the protein sequence submitted.
It's best to stick with higher end GPUs common in machine learning work, like NVIDIA's
A40, T4, and better still, the A100, which will run Alhafold/ColabFold in a couple of
hours to minutes (depending on the GPU, sequence length, sequence complexity, and other
parameters of course).

### A solution for some

The solution I've found that works best for myself is running ColabFold on my university
high performance computing (HPC) cluster which I have access to (Monash Uni's Massive M3 cluster).
Of course, not everyone has access to a HPC, but I assume that if you're running AlphaFold a lot,
there's a good chance you're a researcher at a university to begin with, so that makes
you the target audience of this blog post! Most universities have a HPC that will grant
staff and students access to after filling out some forms, and that gives you an extremely
powerful computing resource at your fingertips which is well worth taking advantage of for
your computational needs.

Working with a HPC can be tricky though. And trying to connect a Colab notebook like
ColabFold to a HPC is not straightforward for a novice. So, in an attempt to provide some
clarity to this issue, **here's my guide on how to connect the ColabFold notebook to a local
runtime on your university HPC over ssh**. Keep in mind, this guide is broadly applicable to running any
Colab notebook on your HPC too.


## Prerequisites

Some things I'm going to assume you have setup already are:

* A Linux command line environment, like having a linux operating system, a MacOS, or the
  Windows Subsystem for Linux (WSL) if you're a Windows user
* access to a HPC, with permission to use some of the GPU partitions (preferably an A40, T4, or A100
  GPU)
* a `ssh` keypair to login to your HPC without the need for passwords
* a `conda` environment with jupyter notebook installed

If you don't have the second point setup yet, you should contact your HPC's system administrators and request all the access you need.

If you haven't setup an `ssh` keypair yet, I highly recommend you do so because it makes
logging in to your HPC so much easier (and more secure) than relying on typing in
passwords over and over. It will also be crucial for the `ssh` port forwarding step required to
make this whole thing work (specifically in Step #5).

Briefly, to setup a `ssh` keypair run the following on the terminal of your local machine (not logged into
your HPC).

```bash
ssh keygen -b 4096 -t rsa -C "Add your own personal comment between quotes"
```

The command will prompt you to give this new keypair a name. I name mine something simple
like `hpc_key`. It will also prompt you for a passphrase next. You can leave it blank,
although it's strongly recommended that you enter a strong passphrase that you can
remember for an extra layer of security.

This will generate the keypair in your `~/.ssh` folder, which consists of a *public* key and a *private* key.
The private key stays exactly where it is, while the public key (denoted by the `.pub` file extension) is what we need to send to the remote `~/.ssh` folder on your HPC.
You can simply copy and paste the contents of the public key to remote `~/.ssh/authorized_keys` on your HPC.
Alternatively, I prefer using other ssh command line tools to achieve the same outcome,
which saves me from the potential for human error involved in manual copy and pasting.
To do this, first add your new ssh key to the ssh client like so:

```bash
ssh-add ~/.ssh/<KEY_NAME>
```

No need to specify whether it's the public or private key, it'll figure that out
automatically. It's good to double check this command worked as expected with,

```bash
ssh-add -l
```

This should list the various keys you have setup on your ssh client, including your new
keypair you just made for the HPC.

Next, add the public key to your remote HPC server with,

```bash
ssh-copy-id -i <KEY_NAME> username@remoteserver
```

No need to specify the path to the key here. This command will copy the public key to the
`~/.ssh/authorized_keys` file on your HPC. If the file `authorized_keys` didn't already
exist, it creates one automatically.

Now you should be able to login to your HPC over ssh by simply typing

```bash
ssh username@remoteserver
```

And that's it!

>**Note!**: if your `authorized_keys` file is already populated with other public keys,
there is a small chance that the `ssh-copy-id` command can accidentally paste your new
public key in on the same line as another key without adding a newline break.
If this happens, it would render those keys ineffective. It's an easy fix though,
just make sure every key appears on its own separate line by checking `authorized_keys` in
your favourite terminal editor like `vim` or `nano`


## 1. `conda install` ColabFold dependencies




## 2. `sbatch` script setup

## 3. Find open ssh port

## 4. Submit Jupyter notebook job to GPU node

## 5. Setup `.ssh/config` on local side and login

## 6. Copy Jupyter token to ColabFold local runtime

## 7. Saving output and potential issues

## References and further reading

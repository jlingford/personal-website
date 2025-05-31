---
title: 'Converting GTDB faa files to a DIAMOND database with taxonomy'
author: 'by James Lingford'
date: 2025-05-30T15:20:33+10:00
draft: false
toc: true
math: true
tags: []
categories: []
---

The [Genome Taxonomy Database (GTDB)](https://gtdb.ecogenomic.org/), is a database of high quality and complete bacterial and archaeal genomes.
It covers the phylogenetic tree in a consistent way, so that no one type of microbe is overrepresented in the database.
The latest release of the GTDB (R226) contains 732,475 genomes in total.
Of this, 143,614 genomes are selected as species representatives that capture most known microbial diversity.

I've wanted to search for proteins in the GTDB, which requires converting the GTDB fasta files of the species representative genomes into a database that's searchable by a tool like BLASTp, DIAMOND, or MMseqs2.
[MMseqs2](https://github.com/soedinglab/MMseqs2/) has a convenient built-in command that handles the downloading and building of a protein sequence database with `mmseqs databases`, which is handy.
However, building an equivalent GTDB database with taxonomy information as well as genome identifiers for [DIAMOND](https://github.com/bbuchfink/diamond) requires a lot more manual effort.
The following is a solution I've come up with to build such a DIAMOND database.

## Downloading fasta files and taxdump information for the GTDB

First, download and extract all the fasta files for the species representative genomes.
Tip: use the <https://data.ace.up.edua.au/public/gtdb> mirror, since the one other mirror is extremely slow.

```bash
wget https://data.ace.uq.edu.au/public/gtdb/data/releases/release226/226.0/genomic_files_reps/gtdb_proteins_aa_reps_r226.tar.gz
tar xvf gtdb_proteins_aa_reps_r226.tar.gz
```

The extracted archive contains a directory called `protein_faa_reps`, which contains two child directories `bacteria` and `archaea`, which themselves contain all the fasta files for each individual genome/species.

Next, download the taxdump information for the GTDB.
The GTDB provides this information in a taxdump archive under the [auxillary files](https://data.ace.uq.edu.au/public/gtdb/data/releases/release226/226.0/auxillary_files/) directory.
Sadly, this archive doesn't contain any sort of taxid map file (with a list of genome ID's in one column with the corresponding taxonomy ID's in a second column).
Thankfully, the GitHub repo [shenwei356/gtdb-taxdump](https://github.com/shenwei356/gtdb-taxdump) provides such a taxid map file for each GTDB release.
The files I want for DIAMOND are `taxid.map`, `names.dmp`, and `nodes.dmp` which can be found [here](https://github.com/shenwei356/gtdb-taxdump/releases/tag/v0.6.0).
Download these files and extract them too.

```bash
wget https://github.com/shenwei356/gtdb-taxdump/releases/download/v0.6.0/gtdb-taxdump-R226.tar.gz
tar xvf gtdb-taxdump-R226.tar.gz
```

If you're interested in generating the `taxid.map` file from scratch, have a look at the documentation from [shenwei356/gtdb-taxdump](https://github.com/shenwei356/gtdb-taxdump) for how to do this.
Another way I've found comes from the authors of the [MMseqs2 wiki](https://github.com/soedinglab/MMseqs2/wiki#create-a-seqtaxdb-for-gtdb) and uses an amazing Bash/Awk script.
I've copy and pasted their script below, with some minor edits.

```bash
# Script from the MMseqs2 wiki with minor edits
# build name.dmp, node.dmp from GTDB taxonomy
mkdir taxonomy/ && cd "$_"
wget https://data.ace.uq.edu.au/public/gtdb/data/releases/release226/226.0/genomic_files_all/ssu_all_r226.fna.gz
gzip -d ssu_all_r226.fna.gz
buildNCBITax=$(cat << 'EOF'
BEGIN{
  ids["root"]=1;
  rank["c"]="class";
  rank["d"]="superkingdom";
  rank["f"]="family";
  rank["g"]="genus";
  rank["o"]="order";
  rank["p"]="phylum";
  rank["s"]="species";
  taxCnt=1;
  print "1\t|\t1\t|\tno rank\t|\t-\t|" > "nodes.dmp"
  print "1\t|\troot\t|\t-\t|\tscientific name\t|" > "names.dmp";
}
/^>/{
  str=$2
  for(i=3; i<=NF; i++){ str=str" "$i}
  n=split(str, a, ";");
  prevTaxon=1;
  for(i = 1; i<=n; i++){
    if(a[i] in ids){
      prevTaxon=ids[a[i]];
    }else{
      taxCnt++;
      split(a[i],b,"_");
      printf("%s\t|\t%s\t|\t%s\t|\t-\t|\n", taxCnt, prevTaxon, rank[b[1]]) > "nodes.dmp";
      printf("%s\t|\t%s\t|\t-\t|\tscientific name\t|\n", taxCnt, b[3]) >"names.dmp"; 
      ids[a[i]]=taxCnt;
      prevTaxon=ids[a[i]];
    }
  }
  gsub(">", "", $1);
  printf("%s\t%s\n", $1, ids[a[n]]) > "mapping";
}
EOF
)
awk -F'\\[loc' '{print $1}' ssu_all_r226.fna | awk "$buildNCBITax"
touch merged.dmp
touch delnodes.dmp
cd ..

mkdir ssu tmp
mmseqs createdb ssu_all_r226.fna ssu

# add taxonomy to GTDB
mmseqs createtaxdb ssu tmp --ncbi-tax-dump taxonomy/ --tax-mapping-file taxonomy/mapping
```

However, I'll be sticking with the pre-made taxid.map file from [shenwei356/gtdb-taxdump](https://github.com/shenwei356/gtdb-taxdump) for the rest of this.

## Adding in GenomeID's to the fasta accessions

Each fasta file in the `protein_faa_reps` dir is named with its genome ID, starting with the `GB_GCA_` or `RS_GCF_` prefix, and ending with the suffix `_protein.faa.gz`.
Each fasta feader within the files contains a unique protein accession identifier and other information from Prodigal.
These protein accession ID's are not very useful on their own.
It would be more helpful if each fasta header contained the genome ID too, since the genome ID can be used to search against the GenBank, RefSeq, or GTDB webservers.
We can add genome ID's to each header with Bash and Awk:

```bash
# navigate to the extracted faa file directory
cd protein_faa_reps

# make output dir
mkdir -p reformatted_files

# REFORMAT FILES
# read all faa files and write new ones with better accession IDs.
for file in ./{bacteria,archaea}/*.gz; do

    # set variable names
    # remove file path from file name
    filename=${file##*/}
    # remove file suffix
    stemname=${filename%_*}
    # remove RS_ and GB_ prefix to get the genome ID
    MAGID=$(echo $stemname | sed -E 's/^(RS_|GB_)//')

    # write new fasta file with accession IDs formatted to read as >GENOMEID~PROTEINID
    zcat ${file} | awk -vMAGID="${MAGID}" '/^>/{gsub(/^>/,"",$1); printf ">%s~%s\n", MAGID, $1; next}{print}' >./reformatted_files/${stemname}_withgenomeids.faa

    echo "written reformatted faa file for genome: ${MAGID}"

done
```

Now each fasta file will be in the format of:

```
>GENOMEID~PROTEINID
SEQUENCE
>GENOMEID~PROTEINID
SEQUENCE
```

Something I want to try in the future is adding other useful info into the fasta headers so that the DIAMOND flag `--outfmt 6 salltitles` will return it.

## Creating the TaxID map file for each faa accession

Making the TaxID map file for each fasta protein accession ID is quite simple with the [shenwei356/gtdb-taxdump](https://github.com/shenwei356/gtdb-taxdump/releases/tag/v0.6.0) `taxid.map` file.
I initially overcomplicated this step with using a nested for loop on an array of `taxid.map` entries, and popping out entries after each interaction with `mapfile`.
This was extremely slow (though it was cool learning about uncommonly used Bash features).
The better solution is to just `grep` over the `taxid.map` file, and extract the TaxID that way:

```bash
# for each new formatted faa file, find it's corresponding taxID and write all accessionIDs and taxIDs to a new file
for file in ./reformatted_files/*.faa; do

    # set variable names
    # remove file path from file name
    filename=${file##*/}
    # remove file suffix
    stemname=${filename%_*}
    # get the genome ID
    MAGID=$(echo $stemname | sed -E 's/^(RS_|GB_)//')

    # get the TaxID from the taxid.map file
    taxid=$(grep "${MAGID}" ../gtdb-taxdump-R226/taxid.map | cut -f2)

    # write new taxid map file in the format of GENOMEID~PROTEINID\tTAXID
    awk -vTAXID="${taxid}" -vOFS="\t" '/^>/{gsub(/^>/,"",$1); print $1, TAXID}' ${file} >./taxmapping_dir/${stemname}_taxmapping.txt

    echo "written taxmap file for genome: ${MAGID}"

done

```

By the way, I'm a big fan of using the `awk -v` flag for adding in external variables into an Awk program.
I also find it way easier to set the output field separator `OFS` variable with a `-v` flag rather than using a `BEGIN{OFS="\t"}` block.

## Combining the reformatted fasta files and taxid map files for DIAMOND `makedb`

All the reformatted fasta files and taxid map files need to be combined into one big file each.

```bash
# make the header row for the taxidmap file so DIAMOND recognises it
echo -e "accession.version\ttaxid" >taxidmap

# loop over all taxidmap files and combine them
for file in ./taxmapping_dir/*.txt; do
    cat ${file} >>taxidmap
    echo "combined ${file} into full taxidmap"
done

# move to a more convenient location
mv taxidmap ../gtdb-taxdump-R226

# loop over all reformatted fasta files and combine them
for file in ./reformatted_files/*.faa; do
    cat ${file} >>gtdb226_combined_faa_withgenomeids.faa
    gzip ${file}
    echo "combined faa ${file} and gzipped"
done

# compress this big fasta file (DIAMOND makedb can use .gz fasta files as input)
gzip gtdb226_combined_faa_withgenomeids.faa
echo "gzipped full combined faa file"
mv gtdb226_combined_faa_withgenomeids.faa.gz ..
```

## Making the DIAMOND database with taxonomy information

Now we can construct the DIAMOND database finally.
We will use the `nodes.dmp` and `names.dmp` files provided by [shenwei356/gtdb-taxdump](https://github.com/shenwei356/gtdb-taxdump/releases/tag/v0.6.0) repo.

```bash
# NOTE: need diamond binary accessible, assumes you have installed diamond
# NOTE: needs quite a lot a RAM

# set path to GTDB reformatted and combined fasta file
DBDIR=path/to/your/database/dir

# run DIAMOND
./diamond makedb \
    --in ${DBDIR}/gtdb226_combined_faa_withgenomeids.faa.gz \
    --db ${DBDIR}/gtdb226_combined_faa_withgenomeids \
    --taxonmap ${DBDIR}/gtdb-taxdump-R226/taxidmap \
    --taxonnodes ${DBDIR}/gtdb-taxdump-R226/nodes.dmp \
    --taxonnames ${DBDIR}/gtdb-taxdump-R226/names.dmp
```

If everything runs successfully, we should see a log message like this confirming that all accessions have been mapped to a TaxID:

```

Closing the input file...  [0.189s]
Closing the database file...  [0.055s]

Database sequences                   437238178
Database letters                     138730167844
Accessions in database               437238178
Entries in accession to taxid file   23560576
Database accessions mapped to taxid  437238178
Database sequences mapped to taxid   437238178
Database hash                        4660cada12b8409dd679240a0193968d
Total time                           2748s
```

## Querying the DIAMOND database and reformatting the output tsv file

To run a simple BLASTP job with DIAMOND, provide a fasta query file and run the following:

```bash
# change these
DBDIR=path/to/your/database/dir
INPUT=path/to/your/input/fasta_file
OUTPUT=path/to/your/output/dir

./diamond blastp \
    -q $INPUT/your_fasta.faa \
    -d $DBDIR/gtdb226_combined_faa_withgenomeids.dmnd \
    -o $OUTPUT/quick_test.tsv \
    --fast \
    --max-target-seqs 1 \
    --header simple \
    --outfmt 6 qseqid sseqid pident evalue bitscore qcovhsp scovhsp staxids sskingdoms skingdoms sphylums sscinames salltitles stitle full_qseq full_sseq
```

An example output will look like this:

```
qseqid  sseqid  pident  evalue  bitscore        qcovhsp scovhsp staxids sskingdoms      skingdoms       sphylums        sscinames       salltitles      stitle  full_qseq       full_sseq
WP_004030875.1  GCA_002509665.1~DADN01000311.1_4        100     2.88e-253       697     100     99.7    1052962652      Archaea 0       Methanobacteriota       002509665       GCA_002509665.1~DADN01000311.1_4      GCA_002509665.1~DADN01000311.1_4        MKLAILGAGCYRTHAASGITNFSRACEVAEQVGKPEIAMTHSTIAMGAELKELAGIDEIVVSDPVFDNDFTVIDDFEYEAVIEAHKKDPESIMPQIREKVNAVAKDLPKPPKGAIHFTHPEDLGFEVTTDDNEAVQDADWVMTWFPKGDMQMGIIKEFADNLKEGAILTHACTVPTTMFQKIFEDLSSDEMNIAPKVNVSSYHPGAVPEMKGQVYIAEGYASEDAICKLVDWGVAARGDAFKLPAELLGPVCDMCSALTAITYAGILSYRDSVMNILGAPAGFAQMMAKESLTQVTDLMNSVGIDHMEEKLDPGALLGTADSMNFGAAADVLPSVLEVLENRKGKGPTCNI     MKLAILGAGCYRTHAASGITNFSRACEVAEQVGKPEIAMTHSTIAMGAELKELAGIDEIVVSDPVFDNDFTVIDDFEYEAVIEAHKKDPESIMPQIREKVNAVAKDLPKPPKGAIHFTHPEDLGFEVTTDDNEAVQDADWVMTWFPKGDMQMGIIKEFADNLKEGAILTHACTVPTTMFQKIFEDLSSDEMNIAPKVNVSSYHPGAVPEMKGQVYIAEGYASEDAICKLVDWGVAARGDAFKLPAELLGPVCDMCSALTAITYAGILSYRDSVMNILGAPAGFAQMMAKESLTQVTDLMNSVGIDHMEEKLDPGALLGTADSMNFGAAADVLPSVLEVLENRKGKGPTCNI*
```

But it would be helpful if the `sseqid` column was split into two new columns to show the genomeID and the proteinID of each hit, and concurrently split each `GENOMEID~PROTEINID` entry.
We can do this with a simple line of Awk.
The following will change the header line and split each `GENOMEID~PROTEINID` entry at the `~` character.

```bash
awk 'FNR==1{$2="genomeid" OFS "protaccession"; print; next}{gsub(/~/,OFS,$2); print}' quick_test.tsv
```

And the resulting output which now contains a new column:

```
qseqid genomeid protaccession pident evalue bitscore qcovhsp scovhsp staxids sskingdoms skingdoms sphylums sscinames salltitles stitle full_qseq full_sseq
WP_004030875.1 GCA_002509665.1 DADN01000311.1_4 100 2.88e-253 697 100 99.7 1052962652 Archaea 0 Methanobacteriota 002509665 GCA_002509665.1~DADN01000311.1_4 GCA_002509665.1~DADN01000311.1_4 MKLAILGAGCYRTHAASGITNFSRACEVAEQVGKPEIAMTHSTIAMGAELKELAGIDEIVVSDPVFDNDFTVIDDFEYEAVIEAHKKDPESIMPQIREKVNAVAKDLPKPPKGAIHFTHPEDLGFEVTTDDNEAVQDADWVMTWFPKGDMQMGIIKEFADNLKEGAILTHACTVPTTMFQKIFEDLSSDEMNIAPKVNVSSYHPGAVPEMKGQVYIAEGYASEDAICKLVDWGVAARGDAFKLPAELLGPVCDMCSALTAITYAGILSYRDSVMNILGAPAGFAQMMAKESLTQVTDLMNSVGIDHMEEKLDPGALLGTADSMNFGAAADVLPSVLEVLENRKGKGPTCNI MKLAILGAGCYRTHAASGITNFSRACEVAEQVGKPEIAMTHSTIAMGAELKELAGIDEIVVSDPVFDNDFTVIDDFEYEAVIEAHKKDPESIMPQIREKVNAVAKDLPKPPKGAIHFTHPEDLGFEVTTDDNEAVQDADWVMTWFPKGDMQMGIIKEFADNLKEGAILTHACTVPTTMFQKIFEDLSSDEMNIAPKVNVSSYHPGAVPEMKGQVYIAEGYASEDAICKLVDWGVAARGDAFKLPAELLGPVCDMCSALTAITYAGILSYRDSVMNILGAPAGFAQMMAKESLTQVTDLMNSVGIDHMEEKLDPGALLGTADSMNFGAAADVLPSVLEVLENRKGKGPTCNI*
```

## SUMMARY: Running all these steps with a SLURM script

Example of a full SLURM script below. Note: be sure to change parameters and target directories for your own case.

```bash
#!/bin/bash
#SBATCH -D ./
#SBATCH -J GTDB
#SBATCH --time=48:00:00
#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=12
#SBATCH --mem=40000
#SBATCH --output=log-%j.out
#SBATCH --error=log-%j.err

# set database directory
DBDIR=/home/user/path/to/your/database/dir
cd $DBDIR

# DOWNLOAD & EXTRACT THE GTDB PROTEIN FAA FILES
wget -v -P . https://data.ace.uq.edu.au/public/gtdb/data/releases/release226/226.0/genomic_files_reps/gtdb_proteins_aa_reps_r226.tar.gz
tar xvf gtdb_proteins_aa_reps_r226.tar.gz

# DOWNLOAD & EXTRACT TAXID INFO
# acquire prepared taxdump files from: https://github.com/shenwei356/gtdb-taxdump/releases/tag/v0.6.0
wget -v -P . https://github.com/shenwei356/gtdb-taxdump/releases/download/v0.6.0/gtdb-taxdump-R226.tar.gz
tar xvf gtdb-taxdump-R226.tar.gz

# navigate to the extracted faa file directory
cd protein_faa_reps

# prep output dirs
if [[ ! -d reformatted_files ]]; then
    mkdir -p reformatted_files
fi
if [[ ! -d ./taxmapping_dir ]]; then
    mkdir -p taxmapping_dir
fi

# REFORMAT FILES
# read all faa files and write new ones with better accession IDs.
for file in ./{bacteria,archaea}/*.gz; do
    # set variable names
    filename=${file##*/}
    stemname=${filename%_*}
    MAGID=$(echo $stemname | sed -E 's/^(RS_|GB_)//')

    # write new fasta file with accession IDs formatted to read as >GENOMEID~PROTEINID
    zcat ${file} | awk -vMAGID="${MAGID}" '/^>/{gsub(/^>/,"",$1); printf ">%s~%s\n", MAGID, $1; next}{print}' >./reformatted_files/${stemname}_withgenomeids.faa
    echo "written reformatted faa file for genome: ${MAGID}"
done

# WRITE TAXID MAP FILES
# for each new formatted faa file, find it's corresponding taxID and write all accessionIDs and taxIDs to a new file
for file in ./reformatted_files/*.faa; do

    # set variable names
    filename=${file##*/}
    stemname=${filename%_*}
    MAGID=$(echo $stemname | sed -E 's/^(RS_|GB_)//')
    taxid=$(grep "${MAGID}" ../gtdb-taxdump-R226/taxid.map | cut -f2)

    # write new taxid map file in the format of GENOMEID~PROTEINID\tTAXID
    awk -vTAXID="${taxid}" -vOFS="\t" '/^>/{gsub(/^>/,"",$1); print $1, TAXID}' ${file} >./taxmapping_dir/${stemname}_taxmapping.txt
    echo "written taxmap file for genome: ${MAGID}"

done

# MAKE THE FULL TAXID MAP FILE FOR DIAMOND
echo -e "accession.version\ttaxid" >taxidmap
for file in ./taxmapping_dir/*.txt; do
    cat ${file} >>taxidmap
    echo "combined ${file} into full taxidmap"
done
mv taxidmap ../gtdb-taxdump-R226

# COMBINE AND COMPRESS THE NEW REFORMATTED FAA FILES FOR DIAMOND
for file in ./reformatted_files/*.faa; do
    cat ${file} >>gtdb226_combined_faa_withgenomeids.faa
    gzip ${file}
    echo "combined faa ${file} and gzipped"
done
gzip gtdb226_combined_faa_withgenomeids.faa
echo "gzipped full combined faa file"
mv gtdb226_combined_faa_withgenomeids.faa.gz ..
echo "done!"

# RUN DIAMOND MAKE DATABSE COMMAND
# NOTE: need diamond binary accessible, assumes you have installed diamond
# NOTE: needs quite a lot a RAM
./diamond makedb \
    --in ${DBDIR}/gtdb226_combined_faa_withgenomeids.faa.gz \
    --db ${DBDIR}/gtdb226_combined_faa_withgenomeids \
    --taxonmap ${DBDIR}/gtdb-taxdump-R226/taxidmap \
    --taxonnodes ${DBDIR}/gtdb-taxdump-R226/nodes.dmp \
    --taxonnames ${DBDIR}/gtdb-taxdump-R226/names.dmp

```

## Other methods

Shoutout to [nick-youngblut/gtdb_to_taxdump](https://github.com/nick-youngblut/gtdb_to_taxdump)'s GitHub repo, which does the same thing but with nicer Python scripts rather than my ugly slapdash Bash scripts.

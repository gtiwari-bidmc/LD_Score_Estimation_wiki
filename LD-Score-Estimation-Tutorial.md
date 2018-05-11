## Overview

This tutorial describes how to use `ldsc` to estimate LD Scores from [`plink`](https://www.cog-genomics.org/plink2/) format `.bed/.bim/.fam` filesets. 
If you are interested in estimating genetic correlation or the LD Score regression intercept from European-ancestry GWAS data, you can download suitable pre-computed LD Scores from [this URL](https://data.broadinstitute.org/alkesgroup/LDSCORE/eur_w_ld_chr.tar.bz2); there is no need to compute your own LD Scores.

The tutorial is divided into two components. The first component describes how to estimate non-partitioned LD Scores. The second component describes how to estimate partitioned LD Scores and how to use the full baseline model from [Finucane, Bulik-Sullivan et al., 2015](http://biorxiv.org/content/early/2015/01/23/014241.full-text.pdf). The second component is somewhat more complicated.

## TL;DR

You can estimate HapMap3 LD Scores for chromosome 22 using genotype data from the 1000 Genomes Europeans by entering the following commands:

	wget https://data.broadinstitute.org/alkesgroup/LDSCORE/1kg_eur.tar.bz2
	tar -jxvf 1kg_eur.tar.bz2
	python ldsc.py\
		--bfile 22
		--l2\
		--ld-wind-cm 1\
		--out 22

## Univariate LD Scores 

This section of the tutorial requires downloading about 2MB of data.

In order to estimate LD Scores, you need genotype data in the binary `plink` `.bed/.bim/.fam` format. We recommend using 1000 Genomes data from the appropriate continent, which can be downloaded in `vcf` format from the [1000 Genomes FTP site](ftp://ftp.1000genomes.ebi.ac.uk/vol1/ftp/release/20130502/) and converted to `plink` format using the [`plink --vcf`](https://www.cog-genomics.org/plink2/input#vcf) command. 

We recommend estimating LD Scores using a 1 centiMorgan (cM) window. Most `.bim` files have the cM column zeroed out, because `plink` doesn't actually use cM coordinates for anything. You can use the [`plink --cm-map`](https://www.cog-genomics.org/plink2/input#cm_map) flag along with your favorite genetic map (e.g., [here](https://mathgen.stats.ox.ac.uk/impute/1000GP%20Phase%203%20haplotypes%206%20October%202014.html)) to fill in the cM column of your `.bim` file. 

For the purpose of this tutorial, you can download a `.bed/.bim/.fam` fileset with the cM column filled in [here](https://data.broadinstitute.org/alkesgroup/LDSCORE/1kg_eur.tar.bz2). This fileset contains genotypes for all HapMap3 SNPs on chromosome 22 for 378 1000 Genomes Europeans. You can uncompress this fileset with the command

	tar -jxvf 1kg_eur.tar.bz2

This will produce a directory called `1kg_eur` with four files 
	
	22.bim
	22.fam
	22.bed
	22.hm3.daf.gz

The first three files are the `.bed/.bim/.fam` fileset; the third file contains derived allele frequencies (DAFs) for all SNPs in the `.bim` file, for use with the `ldsc --cts-bin` flag later in the tutorial.

You can estimate LD Scores with the command

	python ldsc.py\
		--bfile\
		--l2\ 
		--ld-wind-cm 1\ 
		--out 22

This should take around 10 seconds to run. The `--bfile` flag points to the `plink` format fileset; the syntax is exactly the same as `plink`. The `--l2` flag tells `ldsc` to compute LD Scores. The `--ld-wind-cm` flag tells `lsdc` to use a 1 cM window to estimate LD Scores. The other options are `--ld-wind-kb`, which defines the window size in kilobases, and `--ld-wind-snp`, which defines the window size in terms of a number of SNPs. We recommend using `--ld-wind-cm`, because this allows the window size to vary with the range of LD. It is sensible to use a larger window (as measured in kb) in regions like the MHC where LD spans over tens of megabases than in regions with high recombination rate, where LD doesn't extend beyond ~100kb.

This command will produce four files:
	
	22.log
	22.l2.M
	22.l2.M_5_50
	22.l2.ldscore.gz
	
I will describe these files one at a time. First, let's have a look at the log file.

### `22.log`
The first section is simply the masthead and a list of command-line options:

	*********************************************************************
	* LD Score Regression (LDSC)
	* Version 1.0.0
	* (C) 2014-2015 Brendan Bulik-Sullivan and Hilary Finucane
	* Broad Institute of MIT and Harvard / MIT Department of Mathematics
	* GNU General Public License v3
	*********************************************************************

	Options:
	--ld-wind-cm 1.0
	--out 22
	--bfile 22
	--l2

The next section contains log messages describing the process of reading the `.bed/.bim/.fam` fileset. `ldsc` will remove monomorphic SNPs by default. You can change the MAF lower bound with the '--maf` flag.

	Beginning analysis at Fri Jan 30 10:58:44 2015
	Read list of 19156 SNPs from 22.bim
	Read list of 379 individuals from 22.fam
	Reading genotypes from 22.bed
	After filtering, 19156 SNPs remain
	Estimating LD Score.
	Writing LD Scores for 19156 SNPs to 22.l2.ldscore.gz

The last section shows some basic metadata about the LD Scores. This section is useful for basic quality checks. For example, LD Score should generally be positively correlated with MAF. In this example, the correlation between MAF and L2 is 0.275, which seems sensible. Some LD Scores may be < 1, because `ldsc` estimates LD Score using an unbiased estimator of r<sup>2</sup>, but generally only a few LD Scores should be < 1. In this example, the mean LD Score is lower than usual, because we only took sum r<sup>2</sup> over SNPs in HapMap3 to save time. 

	Summary of LD Scores in 22.l2.ldscore.gz
	      MAF     L2
	mean  0.232   18.535
	std   0.145   16.104
	min   0.001    0.066
	25%   0.104    7.839
	50%   0.224   13.484
	75%   0.355   22.972
	max   0.500  109.716

	MAF/LD Score Correlation Matrix
         MAF    L2
	MAF  1.000  0.275
	L2   0.275  1.000
	Analysis finished at Fri Jan 30 10:58:46 2015
	Total time elapsed: 2.72s

## `22.l2.M`, `22.l2.M_5_50`

Let's have a look at these files:
	
	cat 22.l2.M 22.l2.M_5_50
	19156
	16885
These files tally the number of SNPs in the `.bed/.bim/.fam` fileset. The `.M` file contains the total number of SNPs; the `.l2.M_5_50` file contains the number of SNPs with minor allele frequency above 5%. By default, `ldsc` uses the `.l2.M_5_50` file for estimating heritability, because this avoids extrapolating that h<sup>2</sup> per SNP among common variants is the same as for rare variants (see the [Flavors of Heritability and Genetic Correlation](http://www.biorxiv.org/content/early/2015/01/27/014498.full-text.pdf) subsection of the supplementary note of Bulik-Sullivan, Finucane, et al., 2015 for a more detailed explanation of this point). You can tell `ldsc` to use the `.l2.M` file with the `--not-M-5-50` flag, but this will generally yield upwardly-biased h<sup>2</sup> estimates.


## `22.l2.ldscore.gz`

`ldsc` compresses `.l2.ldscore` files by default using `gzip`. You can view the contents of this file by typing

	gunzip -c 22.l2.ldscore.gz | head 
	
	CHR SNP	        BP	        L2
	22	rs9617528	16061016	1.271
	22	rs4911642	16504399	1.805
	22	rs140378	16877135	3.849
	22	rs131560	16877230	3.769
	22	rs7287144	16886873	7.226
	22	rs5748616	16888900	7.379
	22	rs5748662	16892858	7.195
	22	rs5994034	16894090	2.898
	22	rs4010554	16894264	6.975

The first three columns are CHR = chromosome, SNP = rs number, BP = base pair. `ldsc` uses rs numbers for merging LD Score files with summary statistics, so don't worry if the BP column refers to an old genome build. The BP column is only used for making sure that SNPs are sorted. If you use `ldsc` to estimate LD Scores, the SNPs will always be sorted. The last column (L2) is LD Scores.

## `--w-ld` 

The LD Scores to use as the argument for the `--w-ld` and `--w-ld-chr` flags should be computed as sum r<sup>2</sup> over SNPs in the regression, i.e., the SNPs for which you have Z-scores. If you have a list of regression SNPs in a file called `regression.snplist`, formatted as one rs number per line, no header, then you can compute the appropriate `--w-ld` LD Scores with the `--extract regression.snplist` flag (this is the same syntax for restricting to a subset of SNPs as in `plink`). 

The `--w-ld` LD Scores are just used for weighting the regression and generally do not have a huge impact on the results. If you are using LD Score regression with a large number of traits have Z-scores for slightly different sets of SNPs for each trait, then we recommend using the same `--w-ld` LD Scores for each trait in order to save time.

## Partitioned LD Scores

### Step 1: Creating an annot file

To compute annotation-specific LD scores, you will need an annot file, with extension `.annot` or `.annot.gz`. An annot file typically consists of CHR, BP, SNP, and CM columns, followed by one column per annotation, with the value of the annotation for each SNP (0/1 for binary categories, arbitrary numbers for continuous annotations). The file can have many categories or just a single category. It must have the same SNPs in the same order as the `.bim` file used for the computation of LD scores. We also now allow for "thin annot" files, which omit the CHR, BP, SNP and CM columns and only have data on the annotation itself. These require you to use the `--thin-annot` flag, as described below.

You can always create an annot file on your own. Make sure you have the same SNPs in the same order as the `.bim` file used for the computation of LD scores! We have simplified the annot file creation in two common cases: first, when an annotation consists of a set of genomic regions described in a UCSC bed file; and second, when an annot file is based on a gene set, as in [Finucane et al. 2018 Nat Genet](https://www.nature.com/articles/s41588-018-0081-4). In these two cases, you can use the script `make_annot.py`. 

Two notes about this script:
1. It requires you to install the `pybedtools` package. 
2. It outputs "thin annot" files. These files have only one column, with the annotation, and the software assumes but does not check that this has the same SNPs in the same order as the plink files you will use to compute LD scores. To compute LD scores from these files, you will need to use the flag `--thin-annot`, as below.

To compute annot files from a gene set, you will need the following inputs:

1. `--gene-set-file`, a gene set file with the names of the genes in your gene set, one line per gene name.
2. `--gene-coord-file`, a gene coordinate file, with columns GENE, CHR, START, and END, where START and END are base pair coordinates of TSS and TES. This file can contain more genes than are in the gene set. We provide ENSG_coord.txt as a default. 
3. `--windowsize`, the window size you would like to use. The annotation will include all SNPs within this many base pairs of the transcribed region. 

To compute annot files from a bed file, you will need the following inputs
1. `--bed-file`, the UCSC bed file with the regions that make up your annotation.
2. `--nomerge` [rare]. Usually, you will not want to use this flag. Only use it if you do not want to merge the bed file; i.e., if you want to make a continuous annot file with values proportional to the number of intervals in the bedfile overlapping the SNP, rather than a binary annotation of SNPs that appear in any region. 

In both cases, you will need the following inputs:
1. `--bimfile`, the plink bim file of the dataset you will use to compute LD scores. 
2. `--annot-file`, the name of the annot file to output. If this ends with `.gz` then the resulting file will be gzip-ed. 

To try out this script, download the sample files [in this directory](https://data.broadinstitute.org/alkesgroup/LDSCORE/make_annot_sample_files/). You will also need `1000G.EUR.QC.22.bim` in `1000G_Phase3_plinkfiles.tgz` [in this directory](https://data.broadinstitute.org/alkesgroup/LDSCORE/). Then run

		python make_annot.py \
			--gene-set-file GTEx_Cortex.GeneSet \
			--gene-coord-file ENSG_coord.txt \
			--windowsize 100000 \
			--bimfile 1000G.EUR.QC.22.bim \
			--annot-file GTEx_Cortex.annot.gz
or

		python make_annot.py \
			--bed-file Brain_DPC_H3K27ac.bed \
			--bimfile 1000G.EUR.QC.22.bim \
			--annot-file Brain_DPC_H3K27ac.annot.gz

### Step 2: Computing LD scores with an annot file.

Once you have annot file corresponding to `1000G.EUR.QC.22.bim`---we will use the file `Brain_DPC_H3K27ac.annot.gz` created above---make sure you have `1000G.EUR.QC.22.bed` and `1000G.EUR.QC.22.fam` from [`1000G_Phase3_plinkfiles.tgz`](https://data.broadinstitute.org/alkesgroup/LDSCORE/1000G_Phase3_plinkfiles.tgz), and also download HapMap3 SNPs (`hm.22.snp` in `hapmap3_snps.tgz` [in this directory](https://data.broadinstitute.org/alkesgroup/LDSCORE)).

Then run

		python ldsc.py\
			--l2\ 
			--bfile 1000G.EUR.QC.22\ 
			--ld-wind-cm 1\ 
			--annot Brain_DPC_H3K27ac.annot.gz\ 
			--thin-annot
			--out Brain_DPC_H3K27ac\
			--print-snps hm.22.snp

(Note: the --thin-annot flag is only included if the annot file does *not* have the CHR, BP, SNP, and CM columns.) Repeat for the other chromosomes. Make sure you save your output files to the same directory, with the same file prefix, as your annot files, so that you have `${prefix}.${chr}.annot.gz`, `${prefix}.${chr}.l2.ldscore`, and `${prefix}.${chr}.l2.M_5_50`. 

Note that if you plan to use a comma-separated list of prefixes with the `--ref-ld-chr` flag, as in the "Cell-type group analysis" section of the  [Partitioning Heritability tutorial](https://github.com/bulik/ldsc/wiki/Partitioned-Heritability), then the two sets of LD scores must match in the sense of having the same set of SNPs that is in the baseline model. So if you are planning to build on top of the baseline model, be sure to use the files in 1000G_Phase3_plinkfiles.tgz, together with the HapMap3 SNPs, as above.
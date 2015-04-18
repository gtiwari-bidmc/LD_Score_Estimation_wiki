##Overview 
In this tutorial, you will partition the heritability of schizophrenia by functional category, using the baseline model. We will assume you have already followed the instructions in the [README](https://github.com/bulik/ldsc) for dowloading and installing `python` and `ldsc`. 

##Step 1: Download the data 

The LD scores used in [Finucane, Bulik-Sullivan et al., bioRxiv](http://biorxiv.org/content/early/2015/01/23/014241) can be downloaded [here](ftp://pricelab:pricelab@ftp.broadinstitute.org/LDSCORE/). To do the basic baseline analysis, you will need to download `baseline.*`, `weights.*`, and `1000G.mac5eur.*`. To do a cell-type group comparison (described below) you will have to download all of the files.

To download the BMI summary statistics, go to http://www.broadinstitute.org/collaboration/giant/index.php/GIANT_consortium_data_files and download `GIANT_BMI_Speliotes2010_publicrelease_HapMapCeuFreq.txt`. (You can use the `download GZIP` link and then unzip the resulting file.)

Check that the file head is:

	 > head GIANT_BMI_Speliotes2010_publicrelease_HapMapCeuFreq.txt 
	MarkerName Allele1 Allele2 Freq.Allele1.HapMapCEU p N
	rs10 a c 0.0333 0.708 80566
	rs1000000 g a 0.6333 0.506 123865
	rs10000010 c t 0.425 0.736 123827
	rs10000012 c g 0.8083 0.042 123809
	rs10000013 c a 0.1667 0.0689 123863
	rs10000017 t c 0.2333 0.457 123262
	rs1000002 c t 0.475 0.0322 123783
	rs10000023 t g 0.5917 0.939 123756
	rs10000029 t c 0.975 0.24 103623

Next, convert this file to the `.sumstats` format (see the [docs](../docs/file_formats_sumstats.txt)) using `munge_sumstats.py`. We recommend only keeping HapMap3 SNPs; to do this, you can download a list of HapMap3 SNPs [here](http://www.broadinstitute.org/~bulik/w_hm3.snplist.bz2). Unzip this file to get `w_hm3.snplist`, and then run

	python munge_sumstats.py --sumstats GIANT_BMI_Speliotes2010_publicrelease_HapMapCeuFreq.txt\
	--merge-alleles w_hm3.snplist 
	--out BMI\
	 --a1-inc

This will give you a file called `BMI.sumstats.gz`.


##Step 2: Partition heritability 

The following command will allow you to partition heritability: 

	python ldsc.py 
		--h2 BMI.sumstats.gz\
		--ref-ld-chr baseline.\ 
		--w-ld-chr weights.\
		--overlap-annot\
		--frqfile-chr 1000G.mac5eur.\
		--out BMI_baseline
		
Partitioning heritability with 53 overlapping categories takes about 10 minutes, mostly spent reading in all of the annotation matrices. Here is what each of the flags means: 

### `--h2` 
This flag tells `ldsc` to compute partitioned heritability. The argument should be a file in the `.sumstats` format, which can be gzipped or not.  This file should not include custom array data like immunochip, and it should be data from a population that is similar to the population in the LD scores. The LD scores downloaded for this tutorial are European LD scores.

###`--ref-ld-chr` 
The `--ref-ld` flag tells `ldsc` which LD Score files to use as the independent variable in the LD Score regression. The `--ref-ld-chr` flag is used for LD Score files split across chromosomes. By default, `ldsc` appends the chromosome number to the end. For example, typing `--ref-ld-chr baseline.` tells `ldsc` to use the files `baseline.1.l2.ldscore.gz`, ... , `baseline.22.l2.ldscore.gz` and `baseline.1.l2.M_5_50`, ... , `baseline.22.l2.M_5_50`. If the chromosome number is in the middle of the filename, you can tell `ldsc` where to insert the chromosome number by using an `@` symbol. For example, `--ref-ld-chr ld/chr@`. The argument to `--ref-ld` should omit the file suffixes (`.l2.ldscore.gz`, `.M_5_50`). 

If you are using the `--overlap-annot` flag, then you must have `.annot.gz` files with the same prefix, and these must be the `.annot.gz` files that were used as input to `--l2` when computing the LD scores. In other words, your `.annot.gz` files must have a row for every SNP in your reference panel. On the other hand, your `.l2.ldscore.gz` files may have fewer rows if you used the `--print-snps` flag when they were computed.

You can input here either a single set of file or a comma-separated list of files. Below, there is an example with comma-separated files. When inputting multiple files, be extra careful not to introduce any co-linearity! For example, don't have the same annotation in two files, or a set of annotations in one file that forms a disjoint union of an annotation from the second file. This will cause `ldsc` to throw an error. This won't be a problem if you only use the files downloaded from the LDSCORE folder above.

###`--w-ld-chr`
This flag gives the location of the LD scores used for regression weights. It should be a non-partitioned `.l2.ldscore.gz` file that was computed using the regression SNPs. We recommend using HapMap3 SNPs, excluding the HLA region, as default regression SNPs; that is what is in `weights.*`.

###`--out`
This flag tells `ldsc` where to print the the output. It will append .log and .results to this prefix. So in this case, you will see `BMI_baseline.log` and `BMI_baseline.results`.

### `--overlap-annot`
This flag tells `ldsc` that the categories you used to generate `baseline.*.l2.ldscore.gz` overlap with each other. To interpret the results when there are overlapping categories, `ldsc` needs to read in the `.annot.gz` files. It does this using the same prefix as in `--ref-ld` or `--ref-ld-chr`. 

###`--frqfile-chr`
In order to use only SNPs with MAF > 5% (see [discussion of 22.M_5_50](https://github.com/bulik/ldsc/wiki/LD-Score-Estimation-Tutorial#22l2m-22l2m_5_50)), `ldsc` needs to know how many SNPs with MAF > 5% there are in every pairwise intersection of categories. Because the same annotations may be used in different combinations--for example, several different reference panels may be input at the same time--a single `.M_5_50` file doesn't suffix. Thus `--overlap-annot` requires a frequency file, rather than just a `.M_5_50` file. If you use the `--not-M-5-50` flag, then this file is not necessary.

## Output files

### `.log` file

Format for overlapping categories changing soon.

### `.results` file

This file has the results of the analysis in tab-delimited form. If any category contains all SNPs, then that category will not appear in this file. There is one row for each category and columns summarizing the results: Proportion of SNPs, Proportion of heritability, Enrichment, and standard errors. Enrichment is (Prop. heritability) / (Prop. SNPs). If you use the `--print-coefficients` flag, then there will also be columns for the regression coefficients. (See [Finucane, Bulik-Sullivan et al., bioRxiv](http://biorxiv.org/content/early/2015/01/23/014241) for a discussion of the relationship between the coefficients and proportions of heritability.)

## Cell-type group analysis

To compare cell-type groups, we use the coefficient z-score in an analysis containing the full baseline model. This means that we are controlling for the 53 categories of the full baseline model when comparing cell-type groups. To run the analysis for a single cell-type group, say CNS, run:

	python ldsc.py 
		--h2 BMI.sumstats.gz\
		--w-ld-chr weights.\
		--ref-ld-chr CNS.,baseline.\
		--overlap-annot\
		--frqfile-chr 1000G.mac5eur.\
		--out BMI_CNS\
		--print-coefficients

Note that here, we are using a comma-separated list of file prefixes with `--ref-ld-chr`, and we also include the `--print-coefficients` flag so that we can compare coefficients as well as proportions of heritability. In [Finucane, Bulik-Sullivan et al., bioRxiv](http://biorxiv.org/content/early/2015/01/23/014241), we rank cell types using the z-score of the coefficient of the cell type. 
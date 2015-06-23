##Overview

This tutorial will walk you through computing the genetic correlation between schizophrenia and bipolar disorder using `ldsc` and the summary statistics from the 2013 [PGC Cross-Disorder paper in the Lancet](http://www.ncbi.nlm.nih.gov/pubmed/23453885). This tutorial assumes that you have already downloaded and installed `python` and `ldsc`. If you have not already done this, see the instructions in the [README](https://github.com/bulik/ldsc).

This tutorial requires downloading about 75 MB of data.

## TL;DR

If you want to compute the genetic correlation between schizophrenia and bipolar disorder, type these commands

	# Download Data
	wget www.med.unc.edu/pgc/files/resultfiles/pgc.cross.bip.zip
	wget www.med.unc.edu/pgc/files/resultfiles/pgc.cross.scz.zip
	wget www.broadinstitute.org/~bulik/eur_ldscores/eur_w_ld_chr.tar.bz2
	wget www.broadinstitute.org/~bulik/w_hm3.snplist.bz2


	# Munge Data
	tar -jxvf eur_w_ld_chr.tar.bz2
	unzip -o pgc.cross.bip.zip
	unzip -o pgc.cross.scz.zip
	bunzip2 w_hm3.snplist.bz2
	munge_sumstats.py\
		--sumstats pgc.cross.SCZ17.2013-05.txt\
		--N 17115\
		--out scz\
		--merge-alleles w_hm3.snplist
	munge_sumstats.py\
		--sumstats pgc.cross.BIP11.2013-05.txt\
		--N 11810\ 
		--out bip\
		--merge-alleles w_hm3.snplist

	# LD Score Regression
	ldsc.py\ 
		--rg scz.sumstats.gz,bip.sumstats.gz\
		--ref-ld-chr eur_w_ld_chr/\ 
		--w-ld-chr eur_w_ld_chr/\ 
		--out scz_bip
	less scz_bip.log


##LD Scores

If you just want to estimate genetic correlation in European GWAS, there is probably no need for you to compute your own LD Scores, so you can skip the tutorial on LD Score estimation and download pre-computed LD Scores with the following commands:
	
	wget www.broadinstitute.org/~bulik/eur_ldscores/eur_w_ld_chr.tar.bz2
	tar -jxvf eur_w_ld_chr.tar.bz2

This will create two a new directories in your current working directory named `eur_ref_ld_chr`. If your machine does not have the `wget` utility, you can go to [this URL](http://www.broadinstitute.org/~bulik/eur_ldscores/) and click the LD Score download links.

These LD Scores were computed using 1000 Genomes European data and are appropriate for use with European GWAS data, including the two psychiatric datasets used in this tutorial. For GWAS from other populations, you will need to compute population-appropriate LD Scores. For descriptions of the LD Score files you have just downloaded, see either the [LD Score file formats](https://github.com/bulik/ldsc/wiki/LD-File-Formats) or [LD Score estimation](https://github.com/bulik/ldsc/wiki/LD-Score-Estimation-Tutorial) pages. In general, you will want to have separate sets of LD Scores for the `--w-ld-chr` and `--ref-ld-chr` flags. For this tutorial, we are using the same set of LD Scores for both in order to reduce the amount of data that it is necessary to download, and because this more closely mimics the analyses in the [PGC Cross-Disorder Nature Genetics paper](http://www.ncbi.nlm.nih.gov/pubmed/23933821) (paywalled, unfortunately), which use `GCTA` with a GRM constructed using only genotpyed SNPs.

##Downloading Summary Statistics

First, download summary statistics for schizophrenia (scz) and bipolar disorder (bip) from the [psychiatric genomics consortium website](http://www.med.unc.edu/pgc/downloads). If your machine has the `wget` utility, you can use the commands

	wget www.med.unc.edu/pgc/files/resultfiles/pgc.cross.bip.zip
	wget www.med.unc.edu/pgc/files/resultfiles/pgc.cross.scz.zip

If your machine does not have `wget`, you can just copy and paste the download urls into your browser. This will yield two files, named `pgc.cross.bip.zip` and `pgc.cross.scz.zip`. Unzip these files either by double-clicking in a GUI, or with the `unzip -o` command from the command line. 
This will yield files called `pgc.cross.BIP11.2013-05.txt` and `pgc.cross.SCZ17.2013-05.txt`. 

The first few lines of the BIP file should look like this:

	head pgc.cross.BIP11.2013-05.txt

	snpid hg18chr bp a1 a2 or se pval info ngt CEUaf
	rs3131972	1	742584	A	G	1.092	0.0817	0.2819	0.694	0	0.16055
	rs3131969	1	744045	A	G	1.087	0.0781	0.2855	0.939	0	0.133028
	rs3131967	1	744197	T	C	1.093	0.0835	0.2859	0.869	0	.
	rs1048488	1	750775	T	C	0.9158	0.0817	0.2817	0.694	0	0.836449
	rs12562034	1	758311	A	G	0.9391	0.0807	0.4362	0.977	0	0.0925926
	rs4040617	1	769185	A	G	0.9205	0.0777	0.2864	0.98	0	0.87156
	rs28576697	1	860508	T	C	1.079	0.2305	0.7423	0.123	0	0.74537
	rs1110052	1	863421	T	G	1.088	0.2209	0.702	0.137	0	0.752294
	rs7523549	1	869180	T	C	1.823	0.8756	0.4929	0.13	0	0.0137615
	
The first few lines of the scz file should look like this 

	head pgc.cross.SCZ17.2013-05.txt

	snpid hg18chr bp a1 a2 or se pval info ngt CEUaf
	rs3131972	1	742584	A	G	1	0.0966	0.9991	0.702	0	0.16055
	rs3131969	1	744045	A	G	1	0.0925	0.9974	0.938	0	0.133028
	rs3131967	1	744197	T	C	1.001	0.0991	0.9928	0.866	0	.
	rs1048488	1	750775	T	C	0.9999	0.0966	0.9991	0.702	0	0.836449
	rs12562034	1	758311	A	G	1.025	0.0843	0.7716	0.988	0	0.0925926
	rs4040617	1	769185	A	G	0.9993	0.092	0.994	0.979	0	0.87156
	rs4970383	1	828418	A	C	1.096	0.1664	0.5806	0.439	0	0.201835
	rs4475691	1	836671	T	C	1.059	0.1181	0.6257	1.02	0	0.146789
	rs1806509	1	843817	A	C	0.9462	0.1539	0.7193	0.383	0	0.600917

##Reformatting Summary Statistics

The summary statistics are not in the `.sumstats` format (defined in the [docs](https://github.com/bulik/ldsc/wiki/Summary-Statistics-File-Format) that `ldsc` understands. We strongly recommend that you use the script `munge_sumstats.py` included in this github repository in order to convert summary statistics into the `ldsc` format, because this script checks for a lot of annoying gotchas that have gotten us in trouble before. 

The `ldsc` `.sumstats` format requires six pieces of information for each SNP:

1. A unique identifier (e.g., the rs number)
2. Allele 1 (effect allele)
3. Allele 2 (non-effect allele)
4. Sample size (which often varies from SNP to SNP)
5. A P-value
6. A signed summary statistic (beta, OR, log odds, Z-score, etc)

(Note that some summary statistic files do not have a signed summary statistic, but are coded so that A1 is always the trait- or risk-increasing allele. This is equivalent to providing a signed summary statistic, and `munge_sumstats.py` will process such files if called with the `--a1-inc1 flag').

Imputation quality is correlated with LD Score, and low imputation quality yields lower test statistics, so imputation quality is a confounder for LD Score regression. To prevent bias from variable imputation quality, we usually remove poorly-imputed SNPs by filtering on INFO > 0.9. The scz and bip summary statistics that we're using for this tutorial have INFO columns, so `munge_sumstats.py` will automatically perform the filtering. If you're using summary statistics that don't come with an INFO column, we recommend filtering to HapMap3 SNPs (using the `--merge` or `--merge-alleles` flags), because these seem to be well-imputed in most studies.

The two sets of summary statistics that we're using for this tutorial don't have sample size columns, so we'll have to assume that sample size is the same for all SNPs and specify these sample sizes using the `--N` flag. The sample size for the scz study in question was 17115 and the sample size for the bip study was 11810. 

It is a good idea to check that the alleles listed in your summary statistics files match the alleles listed in the data used to estimate LD Scores. Sometimes a small number of alleles won't match; this usually indicates mis-labeled SNPs. This is accomplished using the `--merge-alleles` flag which takes as its argument a file with a list of SNPs and alleles. You can download the required alleles file with the following command (or by manually following the download link if your machine does not have the `wget` utility):

	wget http://www.broadinstitute.org/~bulik/w_hm3.snplist.bz2
	bunzip2 w_hm3.snplist.bz2

To convert the summary statistics, type the commands

	munge_sumstats.py\
		--sumstats pgc.cross.SCZ17.2013-05.txt\
		--N 17115\
		--out scz\
		--merge-alleles w_hm3.snplist
	munge_sumstats.py\
		--sumstats pgc.cross.BIP11.2013-05.txt\
		--N 11810\
		--out bip\
		--merge-alleles w_hm3.snplist

These commands should take about 20 seconds each, though of course the precise time will vary from machine to machine. This will print a series of log messages to the terminal (described below), along with files, `scz.log`, `scz.sumstats.gz` and `bip.log`, `bip.sumstats.gz`. `munge_sumstats.py` will print warning messages labeled `WARNING` to the log file if it finds anything troubling. You can and should search your log files for warnings with the command `grep 'WARNING' *log`. It turns out there are no warnings for these data. 

Note that `munge_sumstats.py` interprets `A1` as the reference allele and that the `A1` column in the `.sumstats` file format refers to the reference allele.

## Reading the Log Files

I'll describe the contents of the log file section-by-section.

The first section is just the `ldsc` masthead:

	**********************************************************************
	* LD Score Regression (LDSC)
	* Version 1.0.0
	* (C) 2014-2015 Brendan Bulik-Sullivan and Hilary Finucane
	* Broad Institute of MIT and Harvard / MIT Department of Mathematics
	* GNU General Public License v3
	**********************************************************************

The next section tells you what command line options you entered. This section is useful for when you're looking at old log files, wondering precisely how some data were processed.

	Options:
	--out scz
	--N 17115.0
	--sumstats data/pgc.cross.SCZ17.2013-05.txt.bz2

The next section describes how `munge_sumstats.py` interprets the column headers. `munge_sumstats.py` can understand most column headers by default, but if your summary statistics have exotic column names, you may need to tell `munge_sumstats.py` what it should do. For example, if the `foobar` column contains INFO scores, you should type `munge_sumstats.py --INFO foobar`. You should always check this section of the log file to make sure that `munge_sumstats.py` understood your column headers correctly. If you are not sure whether `munge_sumstats.py` will understand your column headers, the easiest thing to do is just run `munge_sumstats.py`; if it does not understand the column headers, it will raise an error immediately. 

	Interpreting column names as follows:
	info:   INFO score (imputation quality; higher --> better imputation)
	snpid:  Variant ID (e.g., rs number)
	a1:     Allele 1
	pval:   p-Value
	a2:     Allele 2
	or:     Odds ratio (1 --> no effect; above 1 --> A1 is risk increasing)

This section describes the filtering process. By default, `munge_sumstats.py` filters on INFO > 0.9, MAF > 0.01 and 0 < P <= 1. It also removes variants that are not SNPs (e.g., indels), strand ambiguous SNPs, and SNPs with duplicated rs numbers. If there is an N column (sample size), it removes SNPs with low values of N. Finally, `munge_sumstats.py` checks that the median value of the signed summary statistic column (beta, Z, OR, log OR) is close to the null median (e.g., median OR should be close to 1) in order to make sure that this column is not mislabeled (it is surprisingly common for columns labeled OR to contain log odds ratios).

	Reading sumstats from data/pgc.cross.SCZ17.2013-05.txt.bz2 into memory 5000000.0 SNPs at a time.
	Read 1237958 SNPs from --sumstats file.
	Removed 0 SNPs with missing values.
	Removed 303707 SNPs with INFO <= 0.9.
	Removed 0 SNPs with MAF <= 0.01.
	Removed 0 SNPs with out-of-bounds p-values.
	Removed 0 variants that were not SNPs or were strand-ambiguous.
	856732 SNPs remain.
	Removed 0 SNPs with duplicated rs numbers (856732 SNPs remain).
	Using N = 17115.0
	Median value of or was 1.0, which seems sensible.
	Writing summary statistics for 856732 SNPs (856732 with nonmissing beta) to scz.sumstats.gz.

The last section shows some basic metadata about the summary statistics. If mean chi-square is below 1.02, `munge_sumstats.py` will warn you that the data probably are not suitable for LD Score regression.

	Metadata:
	Mean chi^2 = 1.243
	Lambda GC = 1.205
	Max chi^2 = 32.847
	23 Genome-wide significant SNPs (some may have been removed by filtering).

	Conversion finished at Wed Jan 28 18:17:08 2015
	Total time elapsed: 17.38s 


## Estimating Genetic Correlation

Now that we have all the files that we need in the correct format, we can run LD Score regression with the following command:

	ldsc.py\
		--rg scz.sumstats.gz,bip.sumstats.gz\
		--ref-ld-chr eur_w_ld_chr/\
		--w-ld-chr eur_w_ld_chr/\
		--out scz_bip

This will take about a minute, though the precise time will of course vary from machine to machine. Let's walk through the components of this command. 
###### `--rg`
The `--rg` flag tells `ldsc` to compute genetic correlation. The argument to `--rg` should be a comma-separated list of files in the `.sumstats` format. In this case, we have only passed two files to `--rg`, but if we were to pass three or more files, `ldsc.py` would compute the genetic correlation between the first file and the list and all subsequent files (i.e., --rg a,b,c will compute rg(a,b) and rg(a,c) ). 
###### `--ref-ld-chr`
The `--ref-ld` flag tells `ldsc` which LD Score files to use as the independent variable in the LD Score regression. The `--ref-ld-chr` flag is used for LD Score files split across chromosomes. By default, `ldsc` appends the chromosome number to the end. For example, typing `--ref-ld-chr eur_w_ld_chr/` tells `ldsc` to use the files `eur_w_ld_chr/1.l2.ldscore, ... , eur_w_ld_chr/22.l2.ldscore`. If the chromosome number is in the middle of the filename, you can tell `ldsc` where to insert the chromosome number by using an `@` symbol. For example, `--ref-ld-chr ld/chr@`.  The argument to `--ref-ld` should omit the `.l2.ldscore` or `.l2.ldscore.gz` file suffix.
###### `--w-ld-chr`
The `--w-ld` flag tells `ldsc` which LD Scores to use for the regression weights. Ideally, the `--w-ld` LD Score for SNP j should be the sum over all SNPs k included in the regression of r^2_jk. However, for this tutorial, we are using the same set of LD Scores for `--w-ld` and `--ref-ld`. In practice, LD Score regression is not very sensitive to the precise choice of LD Scores used for the `--w-ld` flag. For example, if you want to compute genetic correlation between scz and bip with 850,000 regression SNPs and genetic correlation between scz and major depression with (say) 840,000 regression SNPs, almost all of which are overlapping, then you should save time and use the same `--w-ld-chr` LD Scores for both regressions.

There is also a `--w-ld-chr` flag; the syntax is identical to the `--ref-ld-chr` flag.

###### `--out`
This tells `ldsc` where to print the results. If you set `--out foo_bar`, `ldsc` will print results to `foo_bar.log`. If you do not set the `--out` flag, `ldsc` will default to printing results to `ldsc.log`.


## Reading the Results File

I will describe the log file section-by-section. The basic structure 
1. log messages about reading input files
2. heritability of the first trait (in this case, scz)
3. heritability of the second trait (in this case, bip)
4. genetic covariance
5. genetic correlation
6. table of genetic correlations (this is very useful for `--rg` with more than two traits)

The first section is just the masthead and list of command line options:

	*********************************************************************
	* LD Score Regression (LDSC)
	* Version 1.0.0
	* (C) 2014-2015 Brendan Bulik-Sullivan and Hilary Finucane
	* Broad Institute of MIT and Harvard / MIT Department of Mathematics
	* GNU General Public License v3
	*********************************************************************

	Options:
	--ref-ld-chr eur_w_ld_chr/
	--out test
	--rg scz.sumstats.gz,bip.sumstats.gz
	--w-ld-chr eur_w_ld_chr/

The next section shows some basic log messages about reading and merging LD Scores and summary statistics. This section isn't that interesting; the only thing to check is whether the number of SNPs drops unexpectedly at any stage. If this happens, it can indicate a data munging error (e.g., mismatched rs numbers). If the number of SNPs is below 200,000, this is usually bad, and `ldsc` will print a warning.

	Beginning analysis at Thu Jan 29 19:11:21 2015
	Reading summary statistics from scz.sumstats.gz ...
	Read summary statistics for 856732 SNPs.
	Reading reference panel LD Score from data/[1-22] ...
	Read reference panel LD Scores for 1293150 SNPs.
	Reading regression weight LD Score from data/[1-22] ...
	Read regression weight LD Scores for 1293150 SNPs.
	After merging with reference panel LD, 840504 SNPs remain.
	After merging with regression SNP LD, 840504 SNPs remain.
	Computing rg for phenotype 2/2
	Reading summary statistics from bip.sumstats.gz ...
	Read summary statistics for 829068 SNPs.
	After merging with summary staistics, 803425 SNPs remain.
	803425 SNPs with valid alleles.

The next two sections show the heritabilities of each trait from single-trait LD Score regression. These estimates will be biased downwards by GC correction. Note that these heritability estimates are on the observed scale. Lambda GC is median(chi^2)/0.4549. Mean chi^2 is the mean chi-square statistic. Intercept is the LD Score regression intercept. The intercept should be close to 1, unless the data have been GC corrected, in which case it will often be lower. Ratio is (intercept-1)/(mean(chi^2)-1), which measures the proportion of the inflation in the mean chi^2 that the LD Score regression intercept ascribes to causes other than polygenic heritability. The value of ratio should be close to zero, though in practice values of 10-20% are not uncommon, probably due to sample/reference LD Score mismatch or model misspecification (e.g., low LD variants have slightly higher h^2 per SNP)

	Heritability of phenotype 1
	---------------------------
	Total Observed scale h2: 0.5909 (0.0484)
	Lambda GC: 1.2038
	Mean Chi^2: 1.2336
	Intercept: 1.0013 (0.0112)
	Ratio: 0.0057 (0.0481)

	Heritability of phenotype 2/2
	-----------------------------
	Total Observed scale h2: 0.5223 (0.0532)
	Lambda GC: 1.1396
	Mean Chi^2: 1.1437
	Intercept: 1.0013 (0.0093)
	Ratio: 0.0092 (0.065)
	
The next section shows the genetic covariance. Genetic covariance will be biased downwards by GC correction. The intercept is shown on the same scale as the single-trait LD Score regression intercept. Multiply by sqrt(N<sub>1</sub>N<sub>2</sub>) in order to obtain an intercept on the N<sub>s</sub>*gencov scale. The data we are using have no sample overlap, so the intercept is less than one standard error away from zero.

	Genetic Covariance
	------------------
	Total Observed scale gencov: 0.3643 (0.0368)
	Mean z1*z2: 0.1226
	Intercept: 0.0037 (0.0071)

The next section shows the genetic correlation, Z-score and P-value. The genetic correlation estimate is not biased by GC correction.

	
	Genetic Correlation
	-------------------

	Genetic Correlation: 0.6558 (0.0604)
	Z-score: 10.8518
	P: 1.9549e-27


The last section (which may not fit too well on your screen) is a table summarizing all results. This feature is a little silly when computing a single genetic correlation, but is a big time-saver when running `--rg` with more than two traits. The columns are p1 = trait 1, p2 = trait 2, rg = genetic correlation, se = standard error of rg, p = p-value for rg; h2_obs, h2_obs_se = observed scale h2 for trait 2 and standard error, h2_int, h2_int_se = single-trait LD Score regression intercept for trait 2 and standard error,  gcov_int, gcov_int_se = cross-trait LD Score regression intercept and standard error.

	Summary of Genetic Correlation Results
	              p1               p2     rg    se       z          p  h2_obs  h2_obs_se  h2_int  h2_int_se  gcov_int  gcov_int_se
	 scz.sumstats.gz  bip.sumstats.gz  0.656  0.06  10.852  1.955e-27   0.522      0.053   1.001      0.009     0.004        0.007

## Conversion to Liability Scale

There is no notion of observed or liability scale genetic correlation. We can compute genetic correlation between pairs of quantitative traits, one quantitative trait and one binary trait, and pairs of binary traits without having to worry about different scales. In addition, if we compute genetic correlations from two studies of the same binary trait with different sample prevalences, we should expect to get the same result, modulo noise.

For heritability and genetic covariance, it is customary to report heritability on the liability scale, because liability scale heritability is comparable across studies with different prevalences. By default, `ldsc` outputs observed scale heritability. To convert to the liability scale, we need to tell `ldsc` the sample and population prevalence for each trait using the `--samp-prev` and `--pop-prev` flags, respectively. The population prevalence of scz and bip are both around 1%, and the sample prevalence in each of these studies was about 50%, so 

	ldsc.py\
		--rg scz.sumstats.gz,bip.sumstats.gz\ 
		--ref-ld-chr eur_w_ld_chr/\
		--w-ld-chr eur_w_ld_chr/\
		--out scz_bip\
		--samp-prev 0.5,0.5\
		--pop-prev 0.01,0.01

The output is the same as before, except 'Observed' is replaced with 'Liability', and the numbers are reported on the liability scale. For example, here is the estimate of the liabilty scale SNP-heritability of schizophrenia:

	Heritability of phenotype 1
	---------------------------
	Total Liability scale h2: 0.3261 (0.0267)
	Lambda GC: 1.2038
	Mean Chi^2: 1.2336
	Intercept: 1.0013 (0.0112)
	Ratio: 0.0057 (0.0481)

If you're computing genetic covariance between one binary trait and one quantitative trait, then you can tell `ldsc` that (say) the second trait is a quantitative trait via `--samp-prev 0.5,nan --pop-prev 0.01,nan`.

## Constraining the Intercept

When estimating heritability, the LD Score regression intercept protects from bias from population stratification and cryptic relatedness. When estimating genetic correlation, the LD Score regression intercept protects against bias shared population stratification and sample overlap. However, constraining the intercept can reduce the standard error substantially. If you think that 

1. The amount of error from population stratification is less than error from sampling noise (usually the case for not-so stratified GWAS with PC covariates)
2. You know how much sample overlap there is (even if there is complete sample overlap!),

then you will probably win on mean square error by constraining the intercept.
However, you should be careful with constraining the intercept. If you constrain the cross-trait LD Score regression intercept to zero when there is sample overlap, you can get completely misleading results that will frequently be out-of-bounds (e.g., r<sub>g</sub> >> 1).

You can constrain the intercept with the `--intercept-h2`  and `--intercept-gencov` flags. For h<sup>2</sup> estimation, the synxtax is `--intercept-h2 N`, where N is the desired intercept (usually 1, though sometimes lower if there is GC correction). For estimating r<sub>g</sub>, you should use both the `--intercept-h2` and `--intercept-gencov` flags to constrain the single-trait and cross-trait LD Score regression intercepts, respectively.

For example, if you're computing --rg A,B,C and you want to constrain the h<sup>2</sup> intercepts to (silly example) 1,0.99,1.01, and the intercept for gencov(A,B) to -0.5 and the intercept for gencov(A,C) to 0.5, then you would use

	--intercept-h2 1,0.99,1.01
	--intercept-gencov 0,N0.5,0.5

There are two subtleties here. First, `python`'s command-line argument parser gets confused by dashes, so to specify negative numbers, use N instead of a dash. Second, the first argument to` --intercept-gencov` is silently ignored so that the arguments to `--intercept-h2` and `--intercept-gencov` line up (for p traits, there are p heritabilities but only p-1 genetic covariances).

If there is sample overlap, the correct arguments to `--intercept-gencov` require some care to compute; a formula can be found in the supplementary note of the [genetic correlation paper](http://www.biorxiv.org/content/early/2015/01/27/014498). 

In this case, there is no sample overlap, so we can shortcut `--no-intercept`, which sets all single-trait intercepts to 1 and all cross-trait intercepts to 0.

	ldsc.py\ 
		--rg scz.sumstats.gz,bip.sumstats.gz\ 
		--ref-ld-chr eur_w_ld_chr/\
		--w-ld-chr eur_w_ld_chr/\
		--out scz_bip\
		--no-intercept

	Heritability of phenotype 1
	---------------------------
	Total Observed scale h2: 0.5948 (0.0299)
	Lambda GC: 1.2038
	Mean Chi^2: 1.2336
	Intercept: constrained to 1

	Heritability of phenotype 2/2
	-----------------------------
	Total Observed scale h2: 0.5277 (0.036)
	Lambda GC: 1.1396
	Mean Chi^2: 1.1437
	Intercept: constrained to 1
	
	Genetic Covariance
	------------------
	Total Observed scale gencov: 0.3774 (0.0238)
	Mean z1*z2: 0.1226
	Intercept: constrained to 0
	
	Genetic Correlation
	-------------------
	Genetic Correlation: 0.6736 (0.0394)
	Z-score: 17.0988
	P: 1.5143e-65


	Summary of Genetic Correlation Results
	              p1               p2     rg     se       z          p  h2_obs  h2_obs_se  h2_int h2_int_se  gcov_int gcov_int_se
	 scz.sumstats.gz  bip.sumstats.gz  0.674  0.039  17.099  1.514e-65   0.528      0.036       1        NA         0          NA
	
The output format is essentially the same. Note that the standard error of the genetic correlation estimate has been reduced by about 35% from 0.0604 to 0.0394. This amount of improvement appears to be typical. 

##Overview
Although ldsc software has been designed for binary annotations, the software allows taking continuous annotations as inputs for both `--l2` and `--h2` options. However, some result outputs of `--h2` option are not meaningful anymore.

In this tutorial, you will partition the heritability of BMI by functional categories, using the baseline-LD model from [Gazal et al. bioRxiv](http://biorxiv.org/content/early/2016/10/19/082024), which contains the baseline model as well as several continuous annotations. You will see how to interpret results for binary and continuous annotations. We will assume that you have already followed the [previous tutorial](https://github.com/bulik/ldsc/wiki/Partitioned-Heritability) on partitioned heritability, that you already generated the file BMI.sumstats.gz, and that you already installed `python` and the last version of `ldsc`.

##Step 1: Running ldsc and interpreting outputs

The annotations and the LD scores from the baseline-LD model used in Gazal et al. bioRxiv can be downloaded [here](https://data.broadinstitute.org/alkesgroup/LDSCORE/1000G_Phase3_baselineLD_ldscores.tgz). The frequency and weight files (computed on European of Phase 3 of 1000 Genomes) can be downloaded [here](https://data.broadinstitute.org/alkesgroup/LDSCORE/1000G_Phase3_frq.tgz) and [here](https://data.broadinstitute.org/alkesgroup/LDSCORE/1000G_Phase3_weights_hm3_no_MHC.tgz).

Run `ldsc` on BMI summary statistics using the command

	python ldsc/ldsc.py \
	  --h2 BMI.sumstats.gz \
	  --ref-ld-chr baselineLD. \
	  --frqfile-chr 1000G.EUR.QC. \
	  --w-ld-chr weights.hm3_noMHC. \
	  --overlap-annot \
	  --print-coefficients \
	  --print-delete-vals \
	  --out BMI.baselineLD

The `--print-coefficients` flag outputs the estimated regression coefficients (i.e. the effect size of the annotation conditioned to all others) in the last column of the `.result` file. The `--print-delete-vals` flag outputs the 200 Jacknife coefficient estimates used to compute standard errors around the coefficient estimates in a `.part_delete` file (check that you have installed the last version of `ldsc` -after 10/28/2016- to access this output file).

The file `BMI.baselineLD.results` contains one row for each category and columns summarizing the results: Proportion of SNPs, Proportion of heritability, Enrichment, and standard errors. Enrichment is (Prop. heritability) / (Prop. SNPs). These outputs make sense only for binary annotations. Do not try to interpret them for continuous annotations. Using `--print-coefficients` flag outputs the regression coefficients and corresponding standard errors and Z score for each annotation. These coefficients measure the additional contribution of one annotation to the model and are interpretable for both binary and continuous annotations (see Finucane et al. 2015 Nat Genet and Gazal et al. bioRxiv for a discussion of the relationship between these regression coefficients and proportions of heritability).

##Step 2:  Computing heritability per quantile of one continuous annotation

Computing the proportion of heritability explained by each quantile of a continuous annotation provides a more intuitive interpretation of the magnitude of a continuous annotation effects. We released an R script [`quantile_h2.r`](https://github.com/bulik/ldsc) to compute these quantities. To compute the proportion of heritability explained by each quintile of MAF-adjusted predicted allele age in the baseline-LD model run the following command 

	Rscript quantile_h2.r baselineLD.MAF_Adj_Predicted_Allele_Age.q5.M BMI.baselineLD BMI.baselineLD.Allele_Age.q5.txt

where the 3 arguments of the function need to be 

1) the file containing the sum of each annotation by quantile of continuous annotations (here `baselineLD.AlleleAge.q5.M`; this file is available when downloading LD scores from the baseline-LD model, in addition to quintiles `.q5.txt` and deciles `.q10.txt` files of each LD-related annotations of the baseline-LD model). For example, the 3 first lines of `baselineLD.AlleleAge.q5.M` are  

	> head -3 baselineLD.MAF_Adj_Predicted_Allele_Age.q5.M | column -t
	N            1070940  1070943  1070946  1070941  1070947
	base         1070940  1070943  1070946  1070941  1070947
	Coding_UCSC  18830    16652    15737    14964    11810

and indicates the number of SNPs in each quintile (line starting with N), and the sum of the base and Coding_UCSC annotations in each quantile (here for example 18830 coding reference SNPs are in the quintile with the lowest MAF-adjusted predicted allele value, while 11810 coding reference SNPs are in the quintile with the highest MAF-adjusted predicted allele value). 
Note that these files are computed only on reference SNPs with MAF >= 5%, as stratified LD score regression only uses common SNPs to compute heritability estimates (see Finucane et al. 2015 Nat Genet for more details).

2) the header of the ldsc outputs (here `BMI.baselineLD`). 

3) the output file name (here `BMI.baselineLD.AlleleAge.q5.txt`).

This output file has one row for each quantile (starting with lowest values) and column summarizing the heritability explained by each quantile, its enrichment and corresponding standard error and P value.

##Step 3:  Adding one continuous annotation to the baseline-LD model

Let’s now summarize the different steps to perform if you want to add one continuous the baseline-LD model.

1) Create 22 `yourannot.*.annot.gz` files with your continuous annotation of interest. 

2) Compute LD scores for this annotation. List of HapMap 3 SNPs (that we recommend to perform regression) can be downloaded [here](https://data.broadinstitute.org/alkesgroup/LDSCORE/w_hm3.snplist.bz2). PLINK files of European of Phase 3 of 1000 Genomes can be downloaded [here](https://data.broadinstitute.org/alkesgroup/LDSCORE/1000G_Phase3_plinkfiles.tgz). The commands to compute LD scores on continuous annotations are the same as the ones to compute LD scores on binary annotations:

	awk '{if ($1!="SNP") {print $1} }' w_hm3.snplist > listHM3.txt
	for CHR in {1..22}
	do
	python ldsc/ldsc.py \
	  --l2 \
	  --bfile 1000G.EUR.QC.$CHR \
	  --ld-wind-cm 1 \
	  --print-snps listHM3.txt \
	  --annot yourannot.$CHR.annot.gz \
	  --out yourannot.$CHR
	done

3) Run `ldsc` on BMI summary statistics using baseline-LD model annotations and your new annotation using the command

	python ldsc/ldsc.py \
	  --h2 BMI.sumstats.gz \
	  --ref-ld-chr baselineLD.,yourannot. \
	  --frqfile-chr 1000G.EUR.QC. \
	  --w-ld-chr weights.hm3_noMHC. \
	  --overlap-annot \
	  --print-coefficients \
	  --print-delete-vals \
	  --out BMI.baselineLD_yourannot

and interpret the regression coefficient results of file `BMI.baselineLD_yourannot.results`. Remember that to combine two arguments to the `-–ref-ld-chr` flag, the corresponding files need to have the same SNPs in the same order.

4) To create the file containing the sum of each annotation by quantile of your continuous annotation, we released a perl script [`quantile_M.pl`](https://github.com/bulik/ldsc) to compute these quantities. Run the command

	perl quantile_M.pl \
	  --ref-annot-chr baselineLD.,yourannot. \
	  --frqfile-chr 1000G.EUR.QC. \
	  --annot-header your_annot_header\
	  --nb-quantile 5\
	  --maf 0.05 \
	  --out AlleleAge.baselineLD_yourannot.q5.M

where `--ref-annot-chr` indicates the annotation files to read, `--frqfile-chr` indicates the frq files to read, `--annot-header`  indicates the header of your annotation in the `yourannot.*.annot.gz` files, `--nb-quantile` indicates the number of quantiles to generate (5 by default), `--maf` indicates the MAF threshold to use to include reference SNPs (0.05 by default), and `--out` indicates the ouput file. Type `perl quantile_M.pl --help` for more information.
Note that perl module IO::Uncompress::Gunzip should be installed to run this script.

5) Compute heritability per quantile of one continuous annotation

	Rscript quantile_h2.r baselineLD_yourannot.AlleleAge.q5.M BMI.baselineLD_yourannot BMI.baselineLD_yourannot.AlleleAge.q5.txt

##General


**Q.** Is it cool if I fork `ldsc`?

**A.** Of course!

##LD Score Estimation

**Q.** Should I parallelize over chromosomes when estimating LD Score?

**A.** Yes. In addition to saving time, LD across chromosome boundaries is almost always spurious LD due to finite sample noise or sample structure.

**Q.** How much time/memory does it take to estimate LD Scores?

**A.** Using a 1cM window, it takes about 1 hour and 1GB RAM on my 1.7GHz Macbook Air to compute LD Scores for chromosome 1 using the N=378 1000 Genomes Europeans. The time and memory complexity are both linear in the number of samples and the number of SNPs. Computing partitioned LD Scores with a large number of partitions increases the memory usage  (~8GB for 50 categories) but has only minimal impact on the runtime.

**Q.** Why are some LD Scores negative?

**A.** The naive estimator of r<sup>2</sup> is biased upwards by approximately 1/N where N is sample size. The amount of bias per pair of SNPs is small, but when summing over thousands of r<sup>2</sup> estimate, the bias becomes substantial. LDSC therefore uses an unbiased estimator of r<sup>2</sup>; which necessarily has the property of returning negative r<sup>2</sup> estimates occasionally. It is normal for a small number of LD Scores to be negative (e.g., a few percent of all SNPs). This has minimal impact on the regression. If more than a few percent of all SNPs have negative LD Scores, this probably means that something is wrong, either that the sample size used for estimating LD Scores is too small or the window size used is too large. 

**Q.** Why am I getting the error message 
>The .annot file must contain the same SNPs in the same order as the `.bim` file.

**A.** Because your `.annot` file does not contain the same SNPs in the same order as your `.bim` file. Merging files on SNP ID's (rs numbers) or chromosomal coordinates is a frequent source of bugs, so LDSC tries to avoid doing this whenever possible. 

**Q.** Why am I getting the error message
>Some SNPs have no annotation in `--cts-bin`. This is a bug!

**A.** This error message should never appear. Email me. 

**Q.** Why am I getting the error message
>Do you really want to compute whole-chomosome LD Score? If so, set the
`--yes-really flag` (warning: it will use a lot of time / memory)"

**A.** Your `--ld-wind-[snp | kb | cm]` option told `ldsc` to compute whole-chromosome LD Scores, i.e., sum r<sup>2</sup> with the sum taken over all the SNPs in your .bim file. This is more often a time-consuming mistake than the real intent of the user (whole-chromosome LD Score contains more information about population structure in the sample than anything else, and this information can be obtained much more rapidly via e.g., PCA). If you actually want to compute sum r<sup>2</sup> with the sum taken over all SNPs in your `.bim` file, set the `--yes-really flag`, but be warned that if you .bed file is large, this could take a very
long time to finish running. 

**Q.** Why are the entries in my `l2.M` or `.l2.M_5_50` file non-integers? 

**A.** If you set any of the flags in the set {`--per-allele`, `--pq-exp`, `--maf-exp`} or you passed a `.annot` file with non-integer entries, then this is to be expected. For example, `--per-allele` computes LD Scores as sum<sub>k</sub> p<sub>jk</sub>(1-p<sub>jk</sub>)r<sup>2</sup><sub>k</sub>, where p<sub>k</sub> denotes the MAF of SNP k. The entry in the `.M` file will be M = sum<sub>k</sub> p<sub>k</sub>(1-p<sub>k</sub>). `ldsc` does not normalize by M in order to facilitate parallelization over chromosomes.
 
**Q.** Why am I getting the error message
>WARNING: ill-conditioned LD Score Matrix!

**A.** This only applies to partitioned LD Scores. This error message means that some of the columns of the LD Score matrix are linearly dependent or are sufficiently close to linearly dependent (condition number > 100,000) that `ldsc` will run into numerical difficulties when attempting to perform LD Score regression with these LD Scores. There are several reasons why this might occur: if the number of SNPs is smaller than the number of LD Scores, if the annot matrix has linearly dependent columns or if the number of categories is very large. Sometimes the LD Score matrix for one of the smaller chromosomes will be ill-conditioned, but the whole-genome LD Score matrix will be fine. Increasing the sample size can also help. 


##Genetic Correlation and Heritability


**Q.** What sample size do I need for LD Score regression?

**A.** As a rule of thumb, LD Score regression tends to yield very noisy results when applied to datasets with fewer than ~5k samples, even for univariate h<sup>2</sup> estimation. One needs even larger sample sizes for asking more complicated questions, e.g., partitioned h<sup>2</sup>. If your sample size is so small that standard error is the limiting factor, but you happen to have individual genotype data, consider using REML (e.g., as implemented in [GCTA](http://www.complextraitgenomics.com/software/gcta/)).

**Q.** Why am I getting the error message
>WARNING: One of the h<sup>2</sup>'s was out of bounds.

**A.** This means that the slope of the LD Score regression was < 0. Since r<sub>g</sub> has sqrt(h<sup>2</sup>) in the denominator, LDSC cannot estimate r<sub>g</sub> if one of the heritabilities is nonnegative, because the square root of a negative number is not a real number. Negative heritabilites are not physically meaningful, but sometimes LD Score regression will return a negative slope, especially if the true value of h<sup>2</sup> is low or the sample size is small.

**Q.** Why am I getting the error message
>WARNING: r<sub>g</sub> was out of bounds.

**A.** `ldsc` returns this error message whenever the r<sub>g</sub> estimate is above 1.25 or below -1.25. This usually means that one of the h<sup>2</sup> estimates was very close to zero. Since h<sup>2</sup> is in the denominator of r<sub>g</sub>, if h<sup>2</sup> is close to zero, r<sub>g</sub> estimates can blow up. This can also occur if you have (incorrectly) constrained the gencov intercept. You can override this message with the `--return-silly-things` flag

**Q.** Why is the r<sub>g</sub> standard error so high?

**A.** The usual culprits are low sample size and low h<sup>2</sup> (since h<sup>2</sup> is in the denominator of r<sub>g</sub>, low  h<sup>2</sup> makes r<sub>g</sub> hard to estimate). You may also run into problems if the number of regression SNPs is small. `ldsc` will give a warning message if the number of regression SNPs is below 200,000, because this usually indicates some sort of merge error, but as a rule of thumb, the standard error may become large with fewer than ~600,000 SNP.

**Q.** How long does it take to compute  r<sub>g</sub> and h<sup>2</sup> with `ldsc`?

**A.** The regression itself takes a few seconds. Reading files into memory and merging them can take a few minutes, especially if you are using a large number of regression SNPs and many partitioned LD Scores.

**Q.** Why is my total h<sup>2</sup> estimate negative?

**A.** Negative h<sup>2</sup> is of course not meaningful, but negative h<sup>2</sup>*estimates* can occur. This usually means that the true h<sup>2</sup> is close to zero, and sampling error pushed the estimate below zero. 

Check whether mean chi<sup>2</sup> is above ~1.02. If no, this means there is very little polygenic signal for LDSC to work with. 

If your h<sup>2</sup> estimate is significantly negative and the mean chi<sup>2</sup> is reasonable large, then something is wrong, either model specification or data processing. Check for warning messages and data munging errors.

**Q.** Why is my h<sup>2</sup> estimate for a disease trait different from previous reports?

**A.** By default, `ldsc` reports h<sup>2</sup> on the observed scale. Most publications report h<sup>2</sup> on the liability scale, since this allows for comparison across studies of the same disease with different proportions of cases and controls. You can use the `--samp-prev` and `--pop-prev` flags to convert to liability scale h<sup>2</sup> and genetic covariance.

**Q.** Why is my h<sup>2</sup> estimate above one?

**A.** If you are estimating h<sup>2</sup> from an ascertained study of a highly heritable rare polygenic disease, then it is possible for the observed scale h<sup>2</sup> to be above one. Quantitative trat h<sup>2</sup> and liability scale h<sup>2</sup> should never be above one. The most likely culprits are errors in your `.M_5_50` file or the `N` (sample size) column of your `.sumstats` file. For example, if the entries in you `N` column are half what they should be, then you will over-estimate h<sup>2</sup> by a factor of two.

**Q.** Why is one of my partitioned h<sup>2</sup> estimates negative?

**A.** If your partitioned h<sup>2</sup> estimate is non-significantly negative, then this may just be sampling noise. If your partitioned h<sup>2</sup> estimate is significantly negative, this may indicate model misspecification. Fitting well-specified models of partitioned heritability is hard. I hate to link to a paper in the middle of an FAQ, but this was the main technical challenge in the paper we wrote about partitioned heritability, so I don't know of a better resource. See [Finucane, Bulik-Sullivan et al, 2015.](http://biorxiv.org/content/early/2015/01/23/014241.full-text.pdf)

**Q.** I estimated partitioned LD Score regression using an annot file with overlapping categories. What do the h<sup>2</sup> estimates mean?

**A.** You need to use the `--overlap-annot` flag in order to obtain meaningful h<sup>2</sup> estimates in this case. Consult the tutorial on partitioned h<sup>2</sup> estimation in the wiki.

**Q.** Why is my single-trait LD Score regression intercept below one?

**A.** If the LD Score regression intercept is non-significantly less than one, If your summary statistics were generated from GC corrected data (even data that were only single GC corrected), the intercept should be less than one (see the supp note of Bulik-Sullivan et al., Nature Genetics, 2015). If your summary statistcs contain chi<sup>2</sup> statistics for SNPs with low minor allele *count*, then the chi<sup>2</sup> statistics for these SNPs may be deflated relative to the asymptotic distribution. Try filtering out low MAF SNPs. If mean chi<sup>2</sup>.  is below one, `ldsc` will not work properly.

**Q.** When should I constrain the intercept?

**A.** If you can rule out QC problems, such as inflation from population stratification.
For genetic correlation, you should constrain the intercept if you can quantify sample overlap (or you are sure that there is no sample overlap). 

**Q.** Why is my h<sup>2</sup> estimate so low?
 
**A.** Double- or single-GC correction both bias the h<sup>2</sup> estimate downwards. If you constrain the intercept to 1 with GC corrected data, this will increase the bias, because the intercept should be < 1 for GC corrected data.

Check to make sure that you have the right sample size in the `N` column of your `.sumstats` file. Most GWAS provide summary statistics from their discovery sample, but mention their replication sample size in the abstract, since the replication sample size is always larger and more impressive-sounding. If you mistakenly use the replication sample size in your chisq file, this will bias your h<sup>2</sup> estimate downwards.

If you are comparing an h<sup>2</sup> estimate from GWAS data to an h<sup>2</sup> estimate from a twin study, you should be aware that these are different quantities. The h<sup>2</sup> parameter estimated by twin studiesincludes the contributions of all genetic variation, not just common SNPs, and the twin study estimator of h<sup>2</sup> is probably biased upwards (see [Zuk, et al., PNAS 2012](http://www.pnas.org/content/109/4/1193.abstract)). Almost all h<sup>2</sup> estimates from GWAS data have been much lower than h<sup>2</sup> estimates from twin data. 

**Q.** Why is the standard error for my h<sup>2</sup> estimate so high?

**A.** The common culprits are small sample size, small number of regression SNPs, 
too many partitioned LD Scores. Try constraining the regression intercept.

**Q.** Why am I getting the warning message
>WARNING: number of SNPs less than 200k; this is almost always bad.

**A.** Having fewer than 200k regression SNPs is often a sign that something went wrong when merging summary statistics and LD Score. With only 200k SNPs, the standard error tends to be very high.

**Q.** I used [immuno | metabo | psych | exome]-chip data and got funky results. What gives?

**A.** LD Score regression doesn't work well with targeted genotyping arrays. Figuring out how to deal with this is on our to-do list. For now, we recommend removing individuals who were genotyped on a specialty array from your dataset OR restricting to SNPs in the GWAS backbone of your specialty chip.

**Q.** I am estimating r<sub>g</sub> between the same trait using the same set of summary statistics twice, and `ldsc` is not returning 1. What is wrong?

**A.** When study1 = study2, z<sub>1</sub>*z<sub>1</sub> = chi<sup>2</sup> and genetic covariance = <sup>2</sup>, the true value of <sub>g</sub> is exactly one. The estimate from `ldsc` will probably not be exactly one, because `ldsc` uses a slightly different initial guess for fitting the single-trait regressions than for the cross-trait regressions (`ldsc` fits the regression via iteratively reweighted least squares).

**Q.** I am estimating  r<sub>g</sub> between different studies of the same trait, and `ldsc` is not returning 1. What is wrong?

**A.** Differences in ancestry between studies could push the  r<sub>g</sub> estimate down slighly.  r<sub>g</sub> between different definitions of the same phenotype might not be one (which could be an interesting result).  r<sub>g</sub> between a trait and a non-linear transformation of that trait (e.g., height and log(height) ) will generally not be 1.
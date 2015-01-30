Frequently Asked Questions
==========================

General
=======

Q: Is it cool if I fork / modify / etc LDSC?
A: Of course!

LD Score Estimation
===================

Q: Should I parallelize over chromosomes when estimating LD Score?
A: Yes.

Q: How much time/memory does it take to estimate LD Scores?
A: Using a 1cM window, it takes about 1 hour and 1GB RAM on my 1.7GHz Macbook Air to 
compute LD Scores for chr 1 using the N=378 1000 Genomes Europeans. The time and memory 
complexity are both linear in the number of samples and the number of SNPs. Computing
partitioned LD Scores with a large number of partitions increases the memory usage 
(~8GB for 50 categories) but has only minimal impact on the runtime.

Q: Why are some LD Scores negative?
A: The naive estimator of r2 is biased upwards by approximately 1/N where N is sample 
size. The amount of bias per pair of SNPs is small, but when summing over thousands
of r2 estimate, the bias becomes substantial. LDSC therefore uses an unbiased estimator
of r2; which necessarily has the property of returning negative r2 estimates occasionally.
It is normal for a small number of LD Scores to be negative (e.g., a few percent of all
SNPs). This has minimal impact on the regression. If more than a few percent of all SNPs
have negative LD Scores, this probably means that something is wrong, either that the
sample size used for estimating LD Scores is too small or the window size used is too
large. 

Q: Why am I getting the error message 
'The .annot file must contain the same SNPs in the same order as the .bim file.'
A: Because your .annot file does not contain the same SNPs in the same order as your
.bim file.

Merging files on SNP ID's (rs numbers) or chromosomal coordinates is a frequent 
source of bugs, so LDSC tries to avoid doing this whenever possible. 

Q: Why am I getting the error message
"Some SNPs have no annotation in --cts-bin. This is a bug!"
A: This error message should never appear. Email me. 

Q: Why am I getting the error message
"Do you really want to compute whole-chomosome LD Score? If so, set the
--yes-really flag (warning: it will use a lot of time / memory)"
A: Your --ld-wind-[snp | kb | cm] option told LDSC to compute whole-chromosome LD Scores,
i.e., sum r^2 with the sum taken over all the SNPs in your .bim file. This is more often
a time-consuming mistake than the real intent of the user (whole-chromosome LD Score
contains more information about population structure in the sample than anything else,
and this information can be obtained much more rapidly via e.g., PCA). If you actually 
want to compute sum r^2 with the sum taken over all SNPs in your .bim file, set the 
--yes-really flag, but be warned that if you .bed file is large, this could take a very
long time to finish running. 

Q: Why are the entries in my [.M | .M_5_50] file non-integers? 
A: If you set any of the flags in the set {--per-allele, --pq-exp, --maf-exp} or you
passed a .annot file with non-integer entries, then this is to be expected. For example,
--per-allele computes LD Scores as sum_k p_k(1-p_k)r^2_jk, where p_k denotes the MAF
of k. The entry in the .M file will be M = sum_k p_k(1-p_k). LDSC does not normalize 
by M in order to facilitate parallelization over chromosome.
 
Q: Why am I getting the error message
"WARNING: ill-conditioned LD Score Matrix!"
A: This only applies to partitioned LD Scores. This error message means that some of 
the columns of the LD Score matrix are linearly dependent or are sufficiently close
to linearly dependent (condition number > 100000) that LDSC will run into numerical 
difficulties when attempting to perform LD Score regression with these LD Scores. 
There are several reasons why this might occur: if the number of SNPs is smaller than
the number of LD Scores, if the annot matrix has linearly dependent columns or if the number 
of categories is very large. Sometimes the LD Score matrix for one of the smaller
chromosomes will be ill-conditioned, but the whole-genome LD Score matrix will be fine.
Increasing the sample size can also help. 


Variance Components
===================

Q: What sample size do I need for LD Score regression?
A: As a rule of thumb, anything below ~5k samples is too small for even univariate h2
estimation with LD Score regression, and the results will be very noisy. One needs even
larger sample sizes for asking more complicated questions, e.g., partitioned h2.
If your sample size is so small that SE is the limiting factor, consider using REML.

Q: Why am I getting the error message
"WARNING: One of the h2's was out of bounds."
A: This means that the slope of the LD Score regression was < 0. 
Since rg has sqrt(h2) in the denominator, LDSC cannot estimate rg if one of the 
heritabilities is nonnegative, because the square root of a negative number is not 
a real number. Negative heritabilites are not physically meaningful, but sometimes 
LD Score regression will return a negative slope, especially if the true value of h2 is low
or the sample size is small.

Q: Why am I getting the error message
"WARNING: rg was out of bounds."
A: LDSC returns this error message whenever the rg estimate is above 1.25 or below -1.25.
This usually means that one of the h2 estimates was very close to zero.
Since h2 is in the denominator of rg, if h2 is close to zero, rg estimates can blow up.
This can also occur if you have (incorrectly) constrained the gencov intercept. You
can override this message with the --return-silly-things flag

Q: Why is by rg standard error so high?
A: The usual culprits are low sample size and low h2 (since h2 is in the denominator 
of rg, low h2 makes rg hard to estimate). You may also run into problems if the # of 
regression SNPs is small (e.g., fewer than ~800,000)

Q: How long does it take to compute rg / h2 with LDSC?
A: The regression itself takes a few seconds. Reading files into memory and merging them
can take a few minutes, especially if you are using a large number of regression SNPs 
and many partitioned LD Scores.

Q: Why is my total h2 estimate negative?
A: Negative heritability is of course not meaningless, but negative heritability
*estimates* can occur. This usually means that the true h2 is zero, and a little 
sampling error pushed the estimate below zero. 

Check whether mean chi^2 is above ~1.02. If no, this means there is very little polygenic 
signal for LDSC to work with. 

If your h2 estimate is significantly negative and the mean chi^2 is reasonable large,
then something is wrong, either model specification or data processing. Check for 
warning messages and data munging errors.

Q: Why is my h2 estimate for a binary disease trait different from previous reports?
A: LDSC reports heritability on the observed scale. Most publications report heritability
on the liability scale, since this allows for comparison across studies with different
proportions of cases and controls, and across diseases with different population 
prevalences. You can use the --samp-prev and --pop-prev flags to convert to liability
scale heritability and genetic covariance.

Q: Why is my h2 estimate above one?
A: If you are estimating h2 from an ascertained study of a highly heritable rare polygenic
disease, then it is possible for the observed scale h2 to be above one. If this is not
the case, then something is probably wrong. Liability scale h2 should never be above one. 
If the LD Score regression intercept is way
below one, then there might be something wrong with your data or the model. 

If the LD Score regression intercept is equal to one, then you may have just used the 
wrong value of N or M. 

Q: Why is one of my partitioned h2 estimates negative?
A: If your partitioned h2 estimate is only somewhat negative (a couple SE's), then this
is probably just sampling noise. As an example, suppose we partitioned the genome into
1000 randomly categories. If the true h2 is 0.5, then the partitioned h2 for each category
is ~5e-4. When we estimate partitioned h2 from these data, we are generating 1000 noisy
estimates of the number 5e-4. Some of those estimates will be negative, just by chance. 
See Finucane, Bulik-Sullivan etal, 2015 here: h
ttp://biorxiv.org/content/early/2015/01/23/014241.full-text.pdf
s

Q: I estimated partitioned LD Score regression using an annot file with overlapping 
categories. What do the h2 estimates mean?
A: Nothing. If you are using overlapping categories, you can suppress meaningless 
reports with the --overlap-annot flag

Q: Why is my LD Score regression intercept below one?
A: If the LD Score regression intercept is only a few SE's below one, this may just
be sampling noise. If your summary statistics were generated from GC corrected data
(even data that were only single GC corrected), the intercept should be less than one
(see the Supplementary Note of Bulik-Sullivan et al 2014). If your summary statistcs
contain chi^2 statistics for SNPs with low minor allele count (note: MAC not MAF), then
the chi^2 statistics for these SNPs may be deflated relative to the asymptotic 
distribution. Try filtering out low MAF SNPs. Check the mean chi^2. If it is below one,
LDSC will not work properly.

Q: When should I constrain the intercept?
A: If you can rule out QC problems, such as inflation from population stratification.
For genetic correlation, you should constrain the intercept if you can quantify sample 
overlap (or you are sure that there is no sample overlap). 

Q: Why is my h2 estimate so low? 
A: Double- or single-GC correction both bias the h2 estimate downwards. Check to make
sure that you have the right sample size in your chisq file. Most GWAS provide summary
statistics from their discovery sample, but mention their replication sample size in
the abstract, since the replication sample size is always larger and more impressive-
sounding. If you mistakenly use the replication sample size in your chisq file, this
will bias your h2 estimate downwards.

This may also result from model misspecification bias. If the LD Score regression
intercept is much higher than one despite trustworthy QC, this can indicate that the
model of genetic architecture used when computing LD Scores does not fit the data well.

If you are comparing an h2 estimate from GWAS data to an h2 estimate from a twin study,
you should be aware that these are different quantities (twin study h2 includes the
contributions of all genetic variation, not just common SNPs), and twin study h2 is 
probably biased upwards (see Zuk, et al PNAS 2012). Almost all h2 estimates from
GWAS data have been much lower than h2 estimates from twin data. 

Q: Why is the SE on my h2 estimate so high?
A: The common culprits are small sample size, small number of regression SNPs, 
too many partitioned LD Scores and not excluding huge-effect loci. 

Q: Why am I getting the warning message
"WARNING: number of SNPs less than 200k; this is almost always bad."
A: Having fewer than 200k regression SNPs is often a sign that something went wrong
when merging summary statistics and LD Score

LD Score regression is designed to be used with ~1 million SNPs included in the
regression. There is nothing wrong with using a smaller number of SNPs (e.g., 
600k SNPs off a genotyping array), though the standard error will tend to be lower
with a larger set of regression SNPs. 

Q: I used [immuno | metabo | psych | exome]-chip data and got funky results. What gives?
A: LD Score regression doesn't work well with targeted genotyping arrays. Email us if 
you would like to think about how to improve this. 

Q: I am estimating rg between the same trait using the same set of summary statistics
twice, and LDSC is not returning 1. What is wrong?
A: When study1 = study1, z1*z2 = chi^2 and genetic covariance = h2, the estimate should 
be close to 1. It will probably not be exactly one, because LDSC uses a slightly different
initial guess for iteratively reweighted least squares fitting.

Q: I am estimating rg between the same trait using summary statistics from different
studies, and LDSC is not returning 1. What is wrong?
A: This may not be an error. The genetic correlation between different definition of the 
same phenotype, or the same phenotype in different countries need not be one, so this 
could potentially be an interesting result. There are also some less exciting explanations.
For example, the genetic correlation for x and log(x) will be less than 1; the genetic
correlation between WHR and WHR adjusted for BMI is not 1, etc. After excluding obvious
data-munging errors, it would be a good idea to look carefully at the phenotype definition
in each study.
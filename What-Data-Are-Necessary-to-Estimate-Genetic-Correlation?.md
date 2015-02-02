Note: all GWAS datasets should be ancestry-matched. LD Score regression allows you to compute the genetic correlation between (say) two European GWAS or two Asian GWAS, but cannot deal with e.g., one European GWAS and one Asian GWAS, or two multi-continental meta-analyses. If you have multi-continental GWAS data, the right approach is to estimate genetic correlation for each continent separately then average the results.

## Required

In order to generate a `.sumstats` file for estimating genetic correlation, the following data are required for each SNP

1. rs number
2. alleles
3. sample size
4. a signed summary statistic (i.e., anything that can be converted into a Z-score)

If sample size does not vary from SNP to SNP, you can set sample size using the `--N,--N-cas,--N-con` flags, for total sample size, number of cases and number of controls. If sample size does vary from SNP to SNP and there is a sample size column in your summary statistic file, `munge_sumstats.py` will read this column and filter out SNPs with low sample size.

Some examples of signed summary statistics include BETA, OR, log OR, Z. P-values alone are not sufficient, because P-values do not indicate which allele is trait- or risk-increasing.

Some summary statistic files do not have a signed summary statistic column, but code alleles such that A1 is always increasing. You can process such files into `ldsc` format using `munge_sumstats.py` and the `--a1-inc` flag.

## Optional

The following data are nice to have:

1. Sample MAF
2. Sample INFO

These data are used to filter out rare or poorly-imputed variants. Imputation quality is correlated with LD Score, so imputation quality is a confounder for LD Score regression. It is important to apply a strict INFO filter for LD Score regression. We recommend INFO < 0.9, which is the default for `ldsc`. If imputation quality data are not available, our preferred workaround is to restrict to HapMap3 SNPs with 1000 Genomes European MAF > 1%, because this set of variants tends to have high INFO in most studies.

It is not necessary to know the amount of sample overlap, but if you do can quantify sample overlap (e.g., if you can rule out sample overlap), you can obtain a better standard error by constraining the LD Score regression intercepts appropriately (with the `--constrain-intercept` flag).

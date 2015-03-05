This page describes all new file formats introduced for use with the --h2 and --rg flags.

NOTE chromosomes are assumed to be integers. We haven't yet implemented LD Score regression for sex chromosomes

## `.sumstats`
For GWAS data. Whitespace-delimited text, one row per SNP with a header row. Column order does not matter. 

We recommend that you convert your summary statistics to the `.sumstats` format using the `munge_sumstats.py` program included with `ldsc`, because `munge_sumstats.py` checks all the gotchas that we've run into over the course of developing this software and applying it to a lot of data.

Required Columns

1. `SNP` -- SNP identifier (e.g., rs number)
2. `N` -- sample size (which may vary from SNP to SNP).
3. `Z` -- z-score. Sign __with respect to `A1`__ (warning, possible gotcha)
4. `A1` -- first allele (effect allele)
5. `A2`-- second allele (other allele)

Note that `ldsc` filters out all variants that are not SNPs and strand-ambiguous SNPs. 
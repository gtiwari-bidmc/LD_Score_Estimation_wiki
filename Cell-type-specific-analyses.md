## Overview

This tutorial is about the --h2-cts flag, which makes it easy to do cell type specific analyses of the sort done in [Finucane et al. 2018 *Nat Genet*](https://www.nature.com/articles/s41588-018-0081-4). In this example, we will replicate a piece of Figure 4B of that paper, in which we test the gene sets from [Cahoy et al. 2008 *J. Neurosci*](http://www.jneurosci.org/content/28/1/264/tab-article-info) for BMI enrichment, using BMI summary statistics from the UK Biobank, released by [Loh et al. 2017 *bioRxiv*](https://www.biorxiv.org/content/early/2018/01/04/194944).  This tutorial assumes that you are familiar with [partitioning heritability](https://github.com/bulik/ldsc/wiki/Partitioned-Heritability) with LDSC.

Here is the total analysis; in the subsequent sections we go through each step.
```
#Choose which set of gene sets to analyze. Options include Multi_tissue_gene_expr, Multi_tissue_chromatin, GTEx_brain, Cahoy, ImmGen, or Corces_ATAC
cts_name=Cahoy 

#Download the LD scores
wget https://data.broadinstitute.org/alkesgroup/LDSCORE/LDSC_SEG_ldscores/${cts_name}_1000Gv3_ldscores.tgz
wget https://data.broadinstitute.org/alkesgroup/LDSCORE/1000G_Phase3_baseline_ldscores.tgz
wget https://data.broadinstitute.org/alkesgroup/LDSCORE/weights_hm3_no_hla.tgz
tar -xvzf ${cts_name}_1000Gv3_ldscores.tgz
tar -xvzf 1000G_Phase3_baseline_ldscores.tgz
tar -xvzf weights_hm3_no_hla.tgz

#Download and format the summary statistics, or use your own.
wget https://data.broadinstitute.org/alkesgroup/UKBB/body_BMIz.sumstats.gz
wget https://data.broadinstitute.org/alkesgroup/LDSCORE/w_hm3.snplist.bz2
bunzip2 w_hm3.snplist.bz
python munge_sumstats.py \
--sumstats ../body_BMIz.sumstats.gz \
--merge-alleles ../w_hm3.snplist \
--out UKBB_BMI

#Run the regression
ldsc.py \
    --h2-cts UKBB_BMI.sumstats.gz \
    --ref-ld-chr 1000G_EUR_Phase3_baseline/baseline. \
    --out BMI_Cahoy_results \
    --ref-ld-chr-cts $cts_name.ldcts \
    --w-ld-chr weights_hm3_no_hla/weights.
```

## Step 1: Download the data 

Download the Cahoy files as follows:
```
cts_name=Cahoy
wget https://data.broadinstitute.org/alkesgroup/LDSCORE/LDSC_SEG_ldscores/${cts_name}_1000Gv3_ldscores.tgz
tar -xvzf ${cts_name}_1000Gv3_ldscores.tgz
```
To analyze other gene sets from Finucane et al. 2018, replace "Cahoy" above with "Multi_tissue_gene_expr" (includes both GTEx data and Franke lab data), "Multi_tissue_chromatin" (includes both Roadmap and ENTEX data), "GTEx_brain", "ImmGen", or "Corces_ATAC". These datasets are all described in the paper. We unfortunately do not have permission to release the gene sets and LD scores from PsychENCODE.

We will also need the baseline model and standard regression weights (see the [Partitioning Heritability](https://github.com/bulik/ldsc/wiki/Partitioned-Heritability) tutorial):
```
wget https://data.broadinstitute.org/alkesgroup/LDSCORE/1000G_Phase3_baseline_ldscores.tgz
wget https://data.broadinstitute.org/alkesgroup/LDSCORE/weights_hm3_no_hla.tgz
tar -xvzf 1000G_Phase3_baseline_ldscores.tgz
tar -xvzf weights_hm3_no_hla.tgz
```

Finally, download and format the BMI summary statistics, using `munge_sumstats.py`:
```
wget https://data.broadinstitute.org/alkesgroup/UKBB/body_BMIz.sumstats.gz
wget https://data.broadinstitute.org/alkesgroup/LDSCORE/w_hm3.snplist.bz2
bunzip2 w_hm3.snplist.bz
python munge_sumstats.py \
--sumstats ../body_BMIz.sumstats.gz \
--merge-alleles ../w_hm3.snplist \
--out UKBB_BMI
```

## Step 2: Run the regressions 

Next, run `ldsc.py` with the --h2-cts flag to do the regressions. Again, you can replace "Cahoy" with one of the other options above, if you have downloaded the necessary data.

```
cts_name=Cahoy
ldsc.py \
    --h2-cts UKBB_BMI.sumstats.gz \
    --ref-ld-chr 1000G_EUR_Phase3_baseline/baseline. \
    --out BMI_Cahoy_results \
    --ref-ld-chr-cts $cts_name.ldcts \
    --w-ld-chr weights_hm3_no_hla/weights.
```
Two flags are new here:
### `--ref-ld-chr-cts`
This file has two columns. The first column has a label, for example the name of the cell type in question. The second column has a comma delimited list of LD scores to include when doing the regression for that cell type. For example, in `Cahoy.ldcts` there are three lines, corresponding to three brain cell types. Each line has two sets of LD scores to include: one is the set of LD scores corresponding to the specifically expressed genes in the cell type, while the second one is a "control" gene set of all genes. The result that will be reported will be the regression coefficient for the first set of LD scores in the list.

### `--h2-cts`
This is a file in sumstats format, as you would input with the `--h2` flag. The `--h2-cts` flag tells `ldsc.py` to run a cell type specific analysis

The remaining flags are as in [partitioned heritability](https://github.com/bulik/ldsc/wiki/Partitioned-Heritability).

This command will run one regression for each line in the `.ldcts` file. The regression will include both the LD scores from that line, as well as whatever LD scores are given to the `--ref-ld-chr` flag. 

## Output files
### `.log` file
This provides a log of the job, as for jobs with the `--h2` flag.

### `.cell_type_results.txt` file
For each line in the `.ldcts` input file, there is a line in the `.cell_type_results.txt` file. There are four columns:

 - The first column of the `.cell_type_results.txt` file has the labels from the first column of the `.ldcts` file. 
 - The next two columns report, respectively, the estimate and standard error of the first regression coefficient from the regression; i.e., the first regression coefficient of the first set of LD scores in the corresponding line of the `.ldcts` file. In the `.ldcts` files provided, this will be the regression coefficient corresponding to the cell type specific annotation: either the set of specifically expressed genes, or the peaks of chromatin mark.
 - The last column gives a P-value from a one-sided test that the coefficient is greater than zero. We recommend using this for significance testing, with appropriate correction for multiple testing.

For example, here is the `.cell_type_results.txt` file from the analysis of Cahoy gene sets and BMI summary statistics:

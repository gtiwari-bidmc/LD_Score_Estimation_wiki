This document describes the file formats required for estimating and storing LD Scores.

There are three main file formats:
1. `.bed`/`.bim`/`.fam` -- see the [Plink 1.9 documentation](https://www.cog-genomics.org/plink2/)
2.  `.annot.`
3.   `'l2.ldscore`, `l2.M_5_50`

*NOTE* chromosomes are assumed to be integers. We haven't yet implemented LD Score regression for sex chromosomes


## .annot

For annotations. One row per SNP. Whitespace-delimited text, with header. `ldsc` supports compression either with `gzip` or `bzip2`. Column order matters. The columns are

0. CHR -- chromosome
1. BP -- physical position (base pairs)
2. SNP -- SNP identifier (rs number)
3. CM -- genetic position  (centimorgans)
4.  all additional columns -- Annotations 

Annotations can be binary (e.g., DHS yes/no) or continuous (e.g., DAF, McVicker B).

Example:

	CHR BP SNP CM AN1 AN2
	1   1  rs1  0  1  0
	1   2  rs2  0  1  0
	1   3  rs3  0  0  1

## .l2.ldscore 

For LD Scores. This is the default output format of the `--l2` flag. One row per SNP. Whitespace-delimited text, with header. `ldsc` supports compression either with `gzip` or `bzip2`, with `gzip` as the default. Column order matters. The columns are

0. CHR -- chromosome
1. BP -- physical position (base pairs)
2. SNP -- SNP identifier (rs number)
3. CM -- genetic position  (centimorgans)
5. MAF -- minor allele frequency
6. all additional columns -- LD Scores

The CM column can be safely set to zero. Genetic positions are only used by the 
`--ld-wind-cm flag`, which reads cM coordinates from the .bim file.

Example:

	CHR     SNP     BP      CM      MAF     LD
	22      rs146752890     16050612        0       0.0672  71.06
	22      rs139377059     16050678        0       0.0553  100.0
	22      rs6518357       16051107        0       0.0593  108.6
	22      rs62224609      16051249        0       0.0897  20.12
	22      rs62224610      16051347        0       0.3337  19.59
	22      rs143503259     16051453        0       0.0936  21.34


## l2.M_5_50
One line, # of columns = number of annotations in the accompanying `.l2.ldscore` file in the same order. Each column contains the number of SNPs in the corresponding annotation category with MAF > 5%. The `.l2.M` file format is the same, except without the restriction on MAF.

Example: 

	100	1000
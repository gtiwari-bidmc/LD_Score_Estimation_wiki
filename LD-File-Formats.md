This document describes all new file formats introduced for use with the --l1, --l1sq, 
--l2 and --l4 flags in the ldsc.py software package.

Glossary

(0) bed/bim/fam -- see Plink documentation http://pngu.mgh.harvard.edu/~purcell/plink/
	or Plink 1.9 documentation https://www.cog-genomics.org/plink2/
(1) annot
(2) ldscore, l2.M_5_50

*NOTE* chromosomes are assumed to be integers. This software does not yet deal with sex
chromosomes.


(1) annot
---------
With header. Columns are 
(0) CHR -- chromosome
(1) BP -- physical position (base pairs)
(2) SNP -- SNP identifier (RS number)
(3) CM -- genetic position in centimorgans
(4 - end) -- Annotations 

Note that one generally thinks of annotations as binary (e.g., DHS = 1 if a SNP is in a 
DHS peak and 0 otherwise), but ldsc.py stores annotations as floats for forwards 
compatibility with future LD Scores (e.g., weighted LD Scores instead of partitioned LD 
Scores).

Example:
CHR BP SNP CM AN1 AN2
1 1 rs1 0 1 0
1 2 rs2 0 1 0
1 3 rs3 0 0 1


(2.1) ldscore 
-------------
Each .ldscore field should always be paired with a .ldscore.M file. Files ending in 
.ldscore contain LD Scores (and optionally standard errors); files ending in .ldscore.M 
contain information about thenumber of SNPs per annotation. Both file formats are 
whitespace-delimited text.

.ldscore files have a header and one SNP per row. The columns are
(0) CHR -- chromosome
(1) BP -- physical position (base pairs)
(2) SNP -- SNP identifier (RS number)
(3) MAF -- minor allele frequency
(4 - end) -- LD Scores

The CM column can be safely set to zero. Genetic positions are only used by the 
--ld-wind-cm flag, which reads cM coordinates from the .bim or .snp file.

Example:
CHR     SNP     BP      CM      MAF     LD
22      rs146752890     16050612        0       0.06728232189973615     71.0634336207099
22      rs139377059     16050678        0       0.0554089709762533      100.06863866975021
22      rs6518357       16051107        0       0.05936675461741425     108.64701224811621
22      rs62224609      16051249        0       0.08970976253298157     20.11209132360909
22      rs62224610      16051347        0       0.3337730870712401      19.593657102402894
22      rs143503259     16051453        0       0.09366754617414252     21.340927392797948


(2.2) l2.M_5_50
---------------
One line, # of columns = number of annotations in the accompanying .ldscore file in the
same order. Each column contains the number of SNPs in the corresponding annotation
category.

Example: 
100	1000
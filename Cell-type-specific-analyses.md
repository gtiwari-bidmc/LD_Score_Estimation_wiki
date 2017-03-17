## Instructions to do the analyses from Finucane et al. 2017 bioRxiv. 

Full tutorial coming soon. In the meantime, here is a script:


    #!/bin/bash
    
    sumstats=$1 #LDSC formatted sumstats file
    cts_name=$2 #options: Cahoy, Franke, GTEx, GTEx_brain, ImmGen, Roadmap
    out=$3
    
    # Download the baseline model and weights
    wget https://data.broadinstitute.org/alkesgroup/LDSCORE/1000G_Phase3_baseline_ldscores.tgz
    wget https://data.broadinstitute.org/alkesgroup/LDSCORE/weights_hm3_no_hla.tgz
    tar -xvzf 1000G_Phase3_baseline_ldscores.tgz
    tar -xvzf weights_hm3_no_hla.tgz
    
    # Download the cts data
    wget https://data.broadinstitute.org/alkesgroup/LDSCORE/LDSC_SEG_ldscores/$cts_name.ldcts
    wget https://data.broadinstitute.org/alkesgroup/LDSCORE/LDSC_SEG_ldscores/${cts_name}_1000Gv3.tgz
    tar -xvzf ${cts_name}_1000Gv3.tgz
    
    # run LDSC
    ldsc.py \
        --h2-cts $sumstats \
        --ref-ld-chr $cts_name/$cts_name.control.,1000G_EUR_Phase3_baseline/baseline. \
        --out $out \
        --ref-ld-chr-cts $cts_name.ldcts \
        --w-ld-chr weights_hm3_no_hla/weights.
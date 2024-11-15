# facets2n_access
Pipeline to run the FACETS2n algorithm on MSK cfDNA ACCESS samples. 

## Background
FACETS2n, developed by Ryan Ptashkin, is designed to compute copy number estimates using unmatched normal samples to reduce noise in log ratio calculations. It is an allele-specific copy number analysis pipeline designed for NGS data. See the full documentation for the FACETS2n package [here](https://github.com/rptashkin/facets2n). 

The motivation behind this pipeline is to implement the FACETS2n package for cfDNA samples. A major issue with cfDNA samples is the difference in sequencing coverage between tumor and matched normal samples. By including several unmatched normals in the analysis, some of these biases can be mitigated. In this repository, I will provide a comprehensive workflow to run FACETS2n on MSK-ACCESS samples and generate downstream copy number calls.

## Requirements
- R (>= 3.4.0)
- packages: pctGCdata, facets2n, facetsSuite
- snp-pileup and HTSlib: see [Installation and usage](https://github.com/rptashkin/facets2n/blob/master/inst/extcode/README.txt)
- BAM file from tumor sample
- BAM from patient matched normal sample (optional)
- BAM(s) from unmatched normal sample(s)
- VCF file

## Installation and required libraries

pctGCdata:
```
remotes::install_github("veseshan/pctGCdata")
```

FACETS2n:
```
devtools::install_github("rptashkin/facets2n")
```

FACETS-suite:
```
devtools::install_github("mskcc/facets-suite")
```

## Step 1: Generate reference files

#### Snp pileup on unmatched reference normals:
```
inst/extcode/snp-pileup-wrapper.R \
  --vcf-file hg_19_00-common_all.vcf \
  --unmatched-normal-BAMS "<path_to_normals>/*.bam"  \
  --output-prefix ref_normals  
```

#### Loess normalization: 

Use the object created from the snp pileup to pass into the `MakeLoessObject` function
```
facets2n::MakeLoessObject(pileup = PreProcSnpPileup(filename = "ref_normals.snp_pileup.gz", 
  is.Reference = TRUE), write.loess = TRUE, outfilepath = "ref_normals.loess.txt", is.Reference = TRUE)
```




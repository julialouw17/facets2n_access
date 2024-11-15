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

#### Step 1a: Snp pileup on unmatched reference normals:
```
inst/extcode/snp-pileup-wrapper.R \
  --vcf-file hg_19_00-common_all.vcf \
  --unmatched-normal-BAMS "<path_to_normals>/*.bam"  \
  --output-prefix ref_normals  
```
In my analysis, I used 39 unmatched normal BAMs.

#### Step 1b: Loess normalization: 

Use the object created from the snp pileup to pass into the `MakeLoessObject` function
```
facets2n::MakeLoessObject(pileup = PreProcSnpPileup(filename = "ref_normals.snp_pileup.gz", 
  is.Reference = TRUE), write.loess = TRUE, outfilepath = "ref_normals.loess.txt", is.Reference = TRUE)
```

## Step 2: Generate input counts data

```
inst/extcode/snp-pileup-wrapper.R \
  --snp-pileup-path <optional> \
  --vcf-file hg_19_00-common_all.vcf \
  --normal-bam Normal.bam \
  --tumor-bam Tumor.bam \
  --unmatched-normal-BAMS <"<some/path/PoolNormal.bam">
  --output-prefix <Sample_ID>
```
Note: for the unmatched normal bams, there should not be any overlap between the BAMs used in the snp pileup command for the reference normals (Step 1a). I utilized 8 unmatched normal BAMs in this step, containing 4 females and 4 males.

## Step 3: Analysis

#### Step 3a: Load the data into R using the `readSnpMatrix` command as part of the `facets2n` package
```
readu_<Sample_ID> <- facets2n::readSnpMatrix(filename = "<path to snp pileup from Step 2>.snp_pileup.gz",
  MandUnormal = TRUE,
  ReferencePileupFile = "<path to snp pileup from Step 1a>.snp_pileup.gz",
  ReferenceLoessFile = "<path to loess normalization file from Step 1b>.loess.txt",
  useMatchedX = TRUE,
  refX=TRUE, 
  unmatched = FALSE)
```
Note: set `unmatched = TRUE` if not using matched normal.

#### Step 3b:  Pre-process inputs count data using the `procSample` function
```
data_<Sample_ID? <- facets2n::preProcSample(readu_<Sample_ID>$rcmat, unmatched = FALSE,
  ndepth = 100, het.thresh = 0.25, ndepthmax = 5000, spanT = readu_<Sample_ID>$spanT, spanA = readu_<Sample_ID>$spanA, spanX = readu_<Sample_ID>$spanX, MandUnormal = TRUE)
```
Note: the `ndepth` parameter was changed to 100 (default is 35) to account for sequencing artifacts in cfDNA samples.

#### Step 3c: First pass with high cval (low sensitivity) to determine logR associated with the diploid state (dipLogR)
```
pass1_<Sample_ID> <- facets2n::procSample(data_<Sample_ID>,min.nhet = 10, cval = 150)
dlr_<Sample_ID> <- pass1_<Sample_ID>$dipLogR
```
The cval used in this step is 150.

#### Step 3d: Second pass with low cval (high sensitivity) to focus on detecting focal events
```
pass2_<Sample_ID> <- facets2n::procSample(data_<Sample_ID>, min.nhet = 10, cval = 50, dipLogR = dlr_<Sample_ID>)
fit_C_<Sample_ID> <- facets2n::emcncf(pass2_<Sample_ID>, min.nhet = 10)
```
The cval used in this step is 50. If you want more segmentation, then decrease this parameter.

## Step 4: Plot
```
facets2n::plotSample(x=pass2_<Sample_ID>, emfit = fit_<Sample_ID>, plot.type = "both")
```
put png image here






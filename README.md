# facets2n_access
Pipeline to run the FACETS2n algorithm on MSK cfDNA ACCESS samples. 

## Background
FACETS2n, developed by Ryan Ptashkin, is designed to compute copy number estimates using unmatched normal samples to reduce noise in log ratio calculations. It is an allele-specific copy number analysis pipeline designed for NGS data. See the full documentation for the FACETS2n package here: https://github.com/rptashkin/facets2n. 

The motivation behind this pipeline is to implement the FACETS2n package for cfDNA samples. A major issue with cfDNA samples is the difference in sequencing coverage between tumor and matched normal samples. By including several unmatched normals in the analysis, some of these biases can be mitigated. In this repository, I will provide a comprehensive workflow to run FACETS2n on ACCESS samples generate downstream copy number calls.

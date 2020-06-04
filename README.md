# covid19_sequencing
As part of the COVID-19 Host Genetics Global initative, this repo serves to corroborate sample scripts for the sequencing analysis. The document that the scripts are based on is titled **COVID-19 Host Genetics Initiative: Whole Exome/Genome Sequencing Analysis Plan**: [link found here](https://docs.google.com/document/d/1X_qjplH8T4BJXSeMQ_sBfQUTiu_kAisicOqGb6B8hcM/edit#heading=h.o2bjrcsk8fjw)

The entire QC plan is using the [Hail](https://hail.is/index.html) software developed by the Broad Institute of Harvard and MIT. 


**Instructions on running the demonstration**

1) Clone Git repo `git clone https://github.com/mkveerapen/covid19_sequencing.git` or download the files found in this repo  into your local directory.

2) Install Hail on your local directory by using instructions found [here](https://hail.is/docs/0.2/getting_started.html).

3) Running the file

_if you are using the Jupyter notebook: Hail_COVID19weswgs.ipynb_

`jupyter notebook Hail_COVID19weswgs.ipynb`

This will open up the file in your default browser for interacting with and exploring Hail code.

_if you are using the `html` file: Hail_COVID19weswgs.html_

Double click on the file to open in your default browser.

The demo `notebook` prepared is meant as a guide which uses a downsampled 1000 genomes dataset (found in the `resources/` folder)

_**Take Note**_: NOT all lines can be run and are meant to serve as an example or template to building your own code for your own datasets. Further instructions on building your own pipeline and exploring code can be found on the [Hail documentation](https://hail.is/docs/0.2/index.html).

If you have questions about set up, debugging, or troubleshooting, please get in contact with the Hail team via the [Hail discuss forum](https://hail.is/index.html).



_The following text is taken from the Google docs_



**PART 2 : SAMPLE and VARIANT QUALITY CONTROL**


**2.0	Sample Quality Control**

All vcf files can be imported as Hail MatrixTables. This can be achieved using the import_vcf function in Hail. We highly recommend using this input format and the Hail platform for conducting analytics because of ease of use, and portability. 


  _2.0.1	WES Interval QC_

This step is an optional step. For sample QC purposes, we would also suggest to filter for intervals where 85% of samples had a mean coverage of 20X, especially when disparate sequencing platforms are used. Intervals that did not pass this interval QC can then be flagged as "fail_interval_qc". 


  _2.0.2	Sex Imputation_
 
We suggest inferring for sex using the Hail function `impute_sex`. This function should be performed on common biallelic SNPs (AF > 0.05) with a high `callrate` (callrate > 0.97). Suggested thresholds for this function include the following. We would also recommend plotting the data to observe data is within reasonable limits of thresholds set below: 

`aaf_threshold`: 0.05

`female_threshold`: 0.5

`male_threshold`: 0.75.

In order to refine inferred sex, we suggest utilizing each sample's fraction of chromosome Y coverage that are normalized using chromosome 20 coverage, where aneuploidies can be determined in samples that impute female but have normalized chrY fraction > 0.1, as well as samples that impute male but have normalized chrY fraction < 0.1. If sex was imputed missing, sex would then be marked as ambiguous. 


  _2.0.3	Additional Filters_
  
Recommended filters removing samples that are  
  
  Mean coverage < 20.0

  Ambiguous sex

  Aneuploids

  Call rate < 97


  _2.0.4	Relatedness filters_

Samples can be filtered to remove one of each pair of related samples using Hail's maximal_independent_set (uses model free relatedness estimation via PC-Relate). We suggest filtering for samples with second-degree relatedness or higher, where one of each pair of samples with a kinship coefficient of > 0.088 can be removed.


  _2.0.5	Population Ancestry Inference_ 

To increase accuracy of inferring population ancestry, we recommend selecting an approach based on study ascertainment: 

_If population ascertained is relatively homogenous:_
We recommend performing PCA projection of the exome data onto the gnomAD population principal components and then to use a random forest classifier trained on gnomAD ancestry labels to assign ancestry to the exome samples. (Konrad to provide loadings and RF for gnomAD without training of model)

_If population ascertained is relatively heterogeneous (multi-ancestry):_
We recommend using a hybrid approach that would first be PCA projection expressed in point a). Secondly, to account for highly admixed samples, we recommend that a from-scratch PCA be performed on the exome dataset using an unsupervised learning/clustering method, e.g. HDBSCAN. Using this hybrid method, any sample that was assigned to a cluster using the from-scratch PCA is given that cluster as their ancestry assignment in order to preserve the substructure observed compared to a projection PCA method. Any sample that was not assigned to a cluster was given the label from the PCA project and random forest classification.
Methods outside of the above mentioned are welcome, if the user has good enough reason to choose otherwise.


  _2.0.6	Outlier Detection_

Utilizing the Hail sample_qc method, we suggest removing outliers that deviate from the median and median absolute deviation (MAD) (non-parametric equivalent for mean and standard deviation) for the following metrics. It is also important to note that these outlier detection metrics below would need to be stratified by population ancestry (and sequencing platform) determined from subsection 2.0.5: 

`n_snp`: Number of SNP alternate alleles

`r_ti_tv`: Transition/transversion ratio

`r_insertion_deletion`: Insertion/Deletion allele ratio

`n_insertion`: Number of insertion alternate alleles

`n_deletion`: Number of deletion alternate alleles

`r_het_hom_var`: Heterozygous/homozygous call ratio



**2.1	Variant Quality Control**

Upon completion of the Sample QC described in section 2.0, exomes should then be processed for Variant QC that is further elaborated in this section 3.0. We recommend applying a PASS filter using the Variant Quality Score Recalibration (VQSR) metric.  



 **2.2	Genotype Quality Control**

High quality genotypes can be filtered when applying the following thresholds. We would also recommend performing call rate filtering separately for cases and controls: differential missingness is a typical source of false positives:

  `GQ` >= 20

  `DP` >= 10

  `AB` >= 0.25 (for each allele in heterozygous calls)



**2.3	Functional Annotation**

Variants can be annotated using the Variant Effect Predictor (VEP) annotation as implemented in Hail (annotation_db) using the default parameters for GRCh38 (including LOFTEE). In addition, for downstream gene-based tests, we recommend grouping variants into genes by canonical transcripts in Ensembl Gene ID and/or HGNC symbols with the following annotations: 


*pLoF*: High-confidence LoF variants (LOFTEE), including stop-gained, essential splice, and frameshift variants, filtered according to a set of first principles as described on the Github repo and gnomAD


*Missense | Low-confidence(LC)*: Missense variants are grouped with in-frame insertions and deletions, as well as low-confidence LoF variants (filtered out by LOFTEE). The latter have a frequency spectrum consistent with missense variation, and affect a set of amino acids in a similar fashion (e.g. a frameshift in the final exon).


*synonymous*: All synonymous variants in the gene (control set).
Additional VEP or machine learning method annotations available e.g. ‘splice_region_variant’ or  kipoi repository (ref: Julien Gagneur).

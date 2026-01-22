# Filtering in the R package SNPfiltR

## Background

The majority of the SNP filtering will be done using the R package **SNPfiltR** (DeRaad, 2023). This R package contains functions that are helpful for visualizing metrics for determining SNP quality and missing data (per sample and SNP). The only filter applied before the vcf file is brought into R is the *--max-obs-het* set in *populations* in **STACKS**. The input vcf file was downloaded to my desktop and an external rmd file was createdm (*01_VCF_filtering-HO0.5.Rmd*).  

### Inputs
1. vcf file (generated with *popuations* in **STACKS**)

### Filters
1. **Genotype depth** (*hard_filter(depth)*): Determines the minimum depth for genotype calls
2. **Genotype quality** (*hard_filter(gq)*): This sets the minimum genotype quality for the genotype that are called
3. **Allele balance** (*filter_allele_balance*): This determines the ratio of reads showing the reference allele to all reads for heterozygous individuals
4. **Maximum depth** (*max_depth*): This sets the maximum mean depth per sample across all SNPs
5. **Minimum minor allele count** (*min_mac*): This sets the minimm number of gene copies carrying the minor allele at a locus
6. **Missing by SNP** (*missing_by_SNP): Determines the maximum proportion of missing data allowed in a SNP (e.x. cutoff = 0.9 indicates 10% missing data per SNP)
7. **Missing by sample** (*misssing_by_sample*): Determines the maximum proportion of missing data allowed in a sample (e.x. cutoff = 0.3 indicates 30% missing data per sample)

## Filtering settings

See R markdown (*01_VCF_filtering-HO0.5.Rmd*) for the code that was ran and for the graphs that were used to help infrom the settings. See supplementary material for testing alternative values of different filtering settings (i.e. allele balance & max depth).   

**Input vcf file was first filtered to have a max observed heterozygosity of 0.5 in *populations* in STACKS.** 71,000 polymorphic sites were filtered out - resulting in 4,606,447 varaint sites to start the filtering in R with.  
  
| Step | Filter | Rational of filter | Setting | Reported values |
| --- | --- | --- | --- | --- |
| 1. | Genotype depth <br> *hard_filter(depth=5)* | Gives support for the confidence of a genotype call | 5 | 28.98% of genotypes fall below a read depth of 5 (converted to NA) |
| 2. | Geotype qaulity <br> *hard_filter(gq=20)* | Support to minize genotype erorrs | 20 | 0.8% of genotypes fall below a quality score of 20 (converted to NA) |
| 3. | Allele balance <br> *filter_allele_balance(min.ratio = 0.25, max.ratio = 0.75)* | Gets rid of heterozygous SNPs outside of expected heterozygosity range | 0.25 - 0.75| 20.85% of heterozygous genotypes fall outside of this range (converted to NA) |
| 4. | Maximum depth per SNP <br> *max_depth(maxdepth=22)* | Removes excessively high depth which could indicate multilocus contigs | 22 | 4,410,477 SNPs retained |
| 5. | Minor allele count <br> *min_mac(min.mac = 3)* | Removes potentially artifactually called SNPs | 3 | 50.76% of SNPs fell below a mac of 3 - 2,171,753 SNPs retained |

**Dataset with filtering scheme 1: Less missing data**  
**(This is the dataset that is used for all the downstream population genomic analyses)**

| Step | Filter | Setting | Reported values |
| --- | --- | --- | --- |
| 6. | Missing by sample <br> *missing_by_sample* | 0.9 | Two individuals removed <br> DR4 and WETO23-107 |
| 7. | Missing by SNP <br> *missing_by_snp* | 0.85 | 96.93% of SNPs fell below the cutoff - 66,625 SNPs retained |
| 8. | Missing by sample <br> *missing_by_sample* | 0.5 | Four individuals removed <br> AS-3, WETO22-086, WETO23-239, RA-04 |
| 9. | Missing by SNP <br> *missing_by_snp* | 0.95 | 66.8% of SNPs fell below the cutoff - 22,118 SNPs retained |
| 10. | Missing by sample <br> *missing_by_sample* | 0.33 | **NO** individuals removed <br> All individuals have less than 33% missing data <br> 38 have less than 15% missing data |
| 11. | Linkage disequilibrium <br> **DONE IN PLINK** <br> (see 09_Linkage_disequilibrium) | 50 5 0.8 | Retained 11,950 SNPs (removed 10,168 SNPs) |
  
**Number of SNPs retained:** 11,950  
**Number of individuals:** 40  
  


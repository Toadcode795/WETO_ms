# WETO ddRAD pipeline 2024
## Project and data overview
### Project
- **Taxon:** Western toads (*Anaxyrus boreas*)
- **Spatial scale:** Broadscale population analysis
- **Goal:** Assess the genetic structure of western toads (*Anaxyrus boreas*) in the Canadian portion of the range.
- **Background:** The western toads in Canada are currently recognized as two populations (calling population and non-calling population: Pauly, 2008) which are delineated as separate Designatable Units (DUs: COSEWIC, 2012). This designation was determined by morphological and behavioural differences among the populations (i.e. the calling population possesses a vocal sac and a pronounced breeding calling, and the lack thereof in the non-calling population). Genetic differences have yet to be assessed between these two populations.
<br>

### Data - Plate2
- **Sequencing Type:** Double digest restriction-site associated DNA sequencing (ddRADseq)
- **Enzymes:** PstI / MspI
- **Library prep:** Sent to Plateforme d’Analyses Génomiques of the Institut de Biologie Intégrative et des Systèmes (IBIS, Universite ́Laval, Quebec, Canada)
- **Sequencing Info:** Paired-end sequencing on an Illumina NovaSeq 6000 at Centre d’expertise et de service Génome Québec at McGill University in Montreal, QC
<br>  

### Before Filtering
  - **Number of individuals sequenced:** 46 (inclusive of one positive control - i.e. replicate). Plus 2 negative controls.
  - **Number of sites:** 39 breeding sites
  - **Range of individuals per site:** 1 to 5 (one site with 3 individuals, one with 5 individuals)
<br>

### Reference genome
- **Published by:** Trumbo et al. (2023) (DOI: 10.1111/mec.17175)
- **Genome size:** ~5.2GB
<br>

## Pipeline overview
### Raw data
Downloaded raw reads from nanuq and performed md5sum check (Sept. 27, 2024).
<br>

### Prepocessing
1. Demultiplexing in **STACKS**
2. Fastq quality (read quality) filtering in **Fastp**
3. Summarizing fastp results in **Multiqc**

### Reference genome alignment
4. Index reference genome in **BWA**
5. Align reads to reference genome in **BWA**

### Assemble loci & call SNPs
6. Build loci in *gstacks* module in **STACKS**
7. Call SNPs in *populations* module in **STACKS** (filter for max observed heterozygosity at this step)

### Filtering 
8. Filter SNPs in R package **SNPfiltR**
9. Filter SNPs for linkage disequilibrium using **PLINK**

### Population genomic analysis
10. PCA (**PLINK**)
11. DAPC (R package **Adegenet**)
12. STRUCTURE (**Compute Canada STRUCTURE scripts**)
13. Fst values (R package **dartR**)
14. Maximum likelihood phylogenetic tree (**IQTree**)  
15. Isolation-by-distance (and significance test) (R package **dartR** and **baseR**)

### Other analyses (in addition to genetic data)
17. Morphological condordance with genetic boundaries
18. Ecological Niche Models (*Maxent* models in **R**) and amount of Niche overlap
  

## Filtering overview
### Overview of major decisions
| Stage | Filter | Program | Setting | Explaination |
| --- | --- | --- | --- | --- |
| Pre-VCF file | Read quality | Fastp | Q (Phred score) >= 20 | Determines bases with a quality below the given threshold |
| Pre-VCF file | Mapping quality | gstacks | Q >= 20 | Score that indicates the quality of the alignment (mapping) of a read to the reference genome |
| Post-VCF file | Geotype depth | SNPflitR | 5 | Gives support for the confidence of a genotype call  |
| Post-VCF file | Minor allele count (MAC) (locus) | SNPfiltR | 3 | Sets the minimum number of copies for the minor alleles to be found at a locus |
| Post-VCF file | Missing data (per individual and per SNP) | SNPfiltR | a) <33% <br> b) <5% | Maximum amount of missing data allowed per individual (a) and per SNP (b) |
| Post-VCF file | Linkage Disequilibrium (LD) | PLINK | 50 5 0.8 | Physical linkage or non-independent assortment leading to the non-random association of alleles at different loci <br> window, step, R2 |  

### Reporting (Pre-VCF file)
Starting number of reads: 1,506,075,128
| Filtering step | Number of Individuals | Reported value |
| --- | --- | --- |
| Read quality (Q>=20) | 48 (inclusive of negative controls) | 929,487,304 reads |
| Mapped reads (only primary aligned reads) | 46 (no negative controls) | 917,788,446 reads |
| Mapping quality (Q>=20) | 46 | 1,772,249 Genotyped loci |
| Read depth (**gstacks**) | 46 | mean depth of coverage=12.0X |

### Datasets (Post-VCF file)
| Filter | Final Dataset |
| --- | --- |
| Max observed Heterozygosity| 0.5 |
| Minimum genotype depth | 5 |
| Maximum genotype depth | 22 |
| Minimum genotype quality (Q) | 20 |
| MAC | 3 |
| Max missingness by SNP | 5% |
| Max missingness by individual | 33% |
| Mean depth of coverage | 19.0X |
| Number of individuals | 40 |
| Number of SNPs | 11,950 |


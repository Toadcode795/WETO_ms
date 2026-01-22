# Linkage Disequilibrium

## Background
Linkage disequlibrium is when there is nonrandom association of alleles between two or more loci. We want to have a dataset with loci that are likely randomly associated with each other so we filter out SNPs that are highly correlated to one another. Thus, by filtering for linkage disequilibrium we will get a vcf file that has SNPs that are in approximate linkage equilibrium. 

We are using the program **PLINK** and the function *--indep-pairwise* to determine which SNPs are in linkage disequilibrium. This function determines the correlations between the genotype allele counts. There are three values that need to be set with this function: 1) window, 2) step, 3) r2. The Window is the size of the variants that are assessed. The step is the variant count to shift the window. An r2 value is calculated between pairs of variants in the given window and values greater than the specified threshold are noted and pruned out (i.e. higher r2 values will prune out less variants).

## Step 1: Plink

### Inputs
1. Filtered vcf file (from 08_SNPfiltR step)

### Flags
`--vcf` Indicates that the input file is in vcf format.   
`--double-id` Needed to indicate that there is only one ID (for "both family and within-family") (required for this dataset)  
`--allow-extra-chr` Needed if chromosomes or contigs start with additional characters other than a digit (required flag for this dataset)  
`--make-bed` Needed to create a new plink binary fileset (required flag)  
`--indep-pairwise` Is used to produce a subset of SNPs that are in approximate linkage equilibrium. Requires three input values - window, step, r2. We are using 50, 5, 0.8. 

### Script

Run plink to get that file of SNPs that are in approximate linkage equilibrium (prune in file).  

plink_LD.sh
```
#!/bin/bash
#SBATCH -c 4
#SBATCH --mem=28GB
#SBATCH --time=01:00:00
#SBATCH -o plink_%A.out
#SBATCH -e plink_%A.err
#SBATCH --mail-type=BEGIN,END,FAIL
#SBATCH --mail-user=EMAIL

infile=$1
outfile=$2

module load StdEnv/2020
module load plink/1.9b_6.21-x86_64

mkdir $outfile


plink --vcf $infile --double-id --allow-extra-chr --make-bed --out temp_data

plink --bfile temp_data --allow-extra-chr --set-missing-var-ids @:# --make-bed --out sorted_data

plink --bfile sorted_data --allow-extra-chr --indep-pairwise 50 5 0.8 --out $outfile
```

### Outputs
1. filename.prune.in (this is the file we want to use - it is the SNP list that is in approximate linkage equilibrium)
2. filename.prune.out (this is the file that specifies the SNPs that are removed in order to get the SNP list to be in approximate linkage equilibrium)
3. filename.log (tells you the number of SNPs that were removed - in this case ## out of ## variants removed)

## Step 2: Vcftools

### Inputs
1. Filtered vcf file (from 08_SNPfiltR step - same input as what was put into PLINK above)
2. filename.prune.in (list from PLINK of which SNPs to include to have approximate linkage equilibrium)

### Flags
`--gzvcf` Indicates that the input file is a compressed vcf file.  
`--snps` Indicates a file with a list of SNPs to include in the output.    
`--recode` Required to indicate that the output is a vcf file.    
`--recode-INFO-all` Indicates that all the INFO key will be kept in the output vcf file.    

### Script

Run vcftools to get new vcf file with only the prune in SNPs.  

LD_vcftools.sh
```
#!/bin/bash
#SBATCH -c 4
#SBATCH --mem=28GB
#SBATCH --time=01:00:00
#SBATCH -o vcf_%A.out
#SBATCH -e vcf_%A.err
#SBATCH --mail-type=BEGIN,END,FAIL
#SBATCH --mail-user=EMAIL


infile=$1
list=$2
outfile=$3

module load vcftools

vcftools --gzvcf $infile --snps $list --recode --recode-INFO-all --out $outfile

```
### Outputs
A new vcf file that only includes that SNPs that are in approximate linkage equilibrium. This vcf file will end with **.recode.vcf**

## Notes
Several combination of values were tested for the window, step and r2 values (using a vcf file that started with 20,369 SNPs). The settings, SNPs removed, and SNPs kept are shown below. PCAs were generated for each combination of values and all the PCAs showed identical clustering of the individuals (See supplimenty material for the PCAs). I used the following combination of values: **50 5 0.8**. This is because these values have been recommended in the literature, it resulted in the most SNPs retained, and the clustering of individuals were identical regardless of the values.  

| Settings (window, step, r2) | SNPs removed | SNPs kept |
| --- | --- | --- |
| 50 5 0.5 | 12,504 | 7,865 |
| 50 10 0.5 | 12,441 | 7,928 |
| 50 10 0.7 | 10,235 | 10,134 |
| **50 5 0.8** <br> (used these values) | 9,184 | 11,185 |
| 50 10 0.8 | 9,156 | 11,213 |

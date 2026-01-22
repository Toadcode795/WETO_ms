# Principal Component Analysis

## Background
Principal component analysis (PCA) is a multivariate analysis that reduces the dimensionality of a dataset while preserving the covariance. The data is reduced into principal components (PC), where each one explains a portion of the genomic variation among the individuals.  

## 1. PLINK
### Inputs
1. Filtered vcf file (from previous steps)

### Flags
`--pca` Extracts top 20 PCs and writes a file of the Eigenvectors and eigenvalues.   

### Running PLINK
pca_plink.sh
```
#!/bin/bash
#SBATCH -c 4
#SBATCH --mem=28GB
#SBATCH --time=01:00:00
#SBATCH -o plink_%A.out
#SBATCH -e plink_%A.err
#SBATCH --mail-type=BEGIN,END,FAIL
#SBATCH --mail-user=jberg031@uottawa.ca


module load StdEnv/2020
module load plink/1.9b_6.21-x86_64


infile=$1
outfile=$2


plink  --vcf $infile --double-id --allow-extra-chr --make-bed --out Temp_data

plink --bfile Temp_data --allow-extra-chr --set-missing-var-ids @:# --make-bed --out sorted_data

plink --bfile sorted_data --allow-extra-chr --pca --out $outfile
```
### Outputs
1. filename.eigenvec (file with the Eigenvectors)
2. filename.eigenval (file with the Eigenvalues)

## 2. Generating plots in R
1. Download the files with the eigenvectors and eigenvalues to the desktop.
2. Plots to visual the explained variance and to see PC1 vs PC2, PC1 vs PC3, etc. are generated in R. There is an external rmd created to run this code (*02_PCA.rmd*).


# Phylogenetic tree - IQTree

## Background
Phylogenetic trees can help to answer "deep phylogenetic" questions. We are using the program **IQTREE** to reconstruct a maximum likelihood phylogenetic tree. Maximum likelyhood programs will find the tree that maximizes the probability of the data and uses nonparametric bootstrapping to get a confidence interval 

## vcf to fasta 
We need to convert the vcf file (from the step above) to a fasta file. The files will be generated using the python script and tutorial found at the following website: https://github.com/edgardomortiz/vcf2phylip/blob/master/README.md.

### Inputs
1. Filtered vcf file

### Script
submit_python.sh
```
#!/bin/bash
#SBATCH -c 1
#SBATCH --mem=64GB
#SBATCH --account=NAME
#SBATCH --time=0-08:00
#SBATCH -o fasta_%A.out
#SBATCH -e fasta_%A.err
#SBATCH --mail-type=BEGIN,END,FAIL
#SBATCH --mail-user=EMAIL

module load python


inputfile=$1


python vcf2phylip.py --input $inputfile --phylip-disable --fasta
```
### Output
A fasta file with the same file name as the input vcf.

## IQTree

### Input files
Fasta file generated from the filtered vcf file (steps above).

### Flags
`-s` Directory of input alignment files.  
`-m` This tells the program which models to assess for the model select. We use **MFP+ASC** which stands for **Model finder Plus and to apply ascertainment bias correction models** (ASC is needed when there is SNP data with only invariant sites).  
`-mtree` Indates for the program to perfrom full tree search for every model.  
`--seqtype` Indicates what type of data is in the input file. In this case we are specifying that we are using DNA data.  
`-B` Turns on ultrafast bootstap for the generated trees. This will generate confidence intervals for the branch support.  

### Running the program
iqtree.sh
```
#!/bin/bash
#SBATCH -c 4
#SBATCH --mem=28GB
#SBATCH --time=1-00:00:00
#SBATCH -o iqtree_%A.out
#SBATCH -e iqtree_%A.err
#SBATCH --mail-type=BEGIN,END,FAIL
#SBATCH --mail-user=jberg031@uottawa.ca


inputfile=$1

module load StdEnv/2020
module load gcc/9.3.0
module load iq-tree/2.2.2.7



iqtree2 -s $inputfile -m MFP+ASC -mtree --seqtype DNA -B 1000
```

### Output
- filename.iqtree (Full results of the run - this is the main report file)  
- filename.log (Log of the entire run)
- filename.treefile (Maximum likelihood tree in NEWICK format - need to use a treeview program (i.e. Figtree) to visualize the tree)

### NOTE
IQTree cannot handle sites that are considered to be partially constant sites when the ASC model is being applied. Partially constant sites are one that have only alternate alleles in heterozygotes need to be removed. (i.e. GGGGR is considered to be partially constant. GGAGR would be an okay site because the A appeares outside of the abiguous site "R"). These are not actually problem sites outside of IQTree, but rather violate the assumptions of the ASC model specifically. Thus, these sites only need to be removed when running IQTree with the ASC model being applied.  

The first time IQTree runs, there will be an error saying `ERROR: Invalid use of +ASC because of XXXX invariant sites in the alignment.` This error generates a new .phy file with these problem sites removed. In this case (using the **HO_0.5_AB_0.25-0.75_mac3_imiss33_SNP95_LD.recode.min4.fasta** input file starting with 11,950 SNPs) we lose 2,302 SNPs that are considered invaraint sites due to being partially constant. 

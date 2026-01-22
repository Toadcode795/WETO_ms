# Assemble loci

## Background
We will use **STACKS** to assemble the loci and generate output files for additional filtering steps.  
  
*gstacks* is the first module used in the **STACKS** pipeline when reads are aligned to a reference genome. This module is used to assemble the loci after the reads are aligned to the reference genome (done in previous step). *gstacks* will take the paired-end reads and identify SNPs for each locus and then genotype each individual at the identified SNPs.  


### Inputs
1. The aligned and sorted BAM file for each individual
   
2. Populations map (text file that indicates which population the individuals are in. Keep it as a single population unless there is prior evidence to indicate more than one population. In this case we have all the individuals in one population.)

### Flags
*gstacks*  
`I` Input directory that contains the BAM files  
`M` The population map with the list of samples in the BAM files    
`O` The output directory   
`--min-mapq` This is a flag for *gstacks* that determines the minimum mapping quality for a read to be considered.  

## Running scripts
First, to make the population map, in the folder with the bam files type in the command line: 
```
ls *.bam | sed 's/\.bam$//' | awk '{print $0 "\t1"}' > popmap.txt
```

gstacks.sh
```
#!/bin/bash
#SBATCH -c 4
#SBATCH --mem=64GB
#SBATCH --time=1-12:00
#SBATCH --account=NAME
#SBATCH --mail-type=BEGIN,END,FAIL
#SBATCH --mail-user=EMAIL

infolder=$1
popmap=$2
outfolder=$3

mkdir -p $outfolder

~/local/bin/gstacks -I $infolder -M $popmap -O $outfolder --min-mapq 20

```
Command line:
```
sbatch ~/scripts/gstacks.sh BAM_WETO_plate2/ref_aligned/unfiltered_BAMs/ popmap.txt gstacks_plate2
```
### Outputs
gstacks.log:  
```
Read 1042899142 BAM records:
  kept 530684346 primary alignments (57.1%), of which 266061895 reverse reads
  skipped 328906354 primary alignments with insufficient mapping qualities (35.4%)
  skipped 58197746 excessively soft-clipped primary alignments (6.3%)
  skipped 11698858 unmapped reads (1.3%)
  skipped some suboptimal (secondary/supplementary) alignment records

Built 1772249 loci comprising 264622451 forward reads and 212670599 matching paired-end reads; mean insert length was 206.1 (sd: 46.3).

Genotyped 1772249 loci:
  effective per-sample coverage: mean=12.0x, stdev=6.2x, min=3.0x, max=31.3x
  mean number of sites per locus: 182.3
  a consistent phasing was found for 2496692 of out 2771018 (90.1%) diploid loci needing phasing
```

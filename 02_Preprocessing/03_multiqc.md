# Multiqc

## Background

The program **multiqc** is used to aggregate separate files from an external programs (in this case **fastp**) into a single report for easy interpretation. We will aggregate the fastp .json file for each individual into a single html file.  \
  \
Multiqc was downloaded from the following link: https://github.com/MultiQC/MultiQC 

### Inputs
1) **json** file from fastp for each individual

## Running multiqc
Fastp files (.json files) were downloaded to my desktop from Compute Canada.  \
  \
In the same folder as the .json files, type the following in the command line:
```
multiqc .
```
### Outputs
HTML files are in the folder: **multiqc_html_files**  \
  \
Important graphs/tables to consider:
1) Summary stats table (look at number of reads)
2) GC Content plots (want the line to be ~50%)

multiqc_general_stats_Q20.txt: statistics from the fastp outputs. This is where the read counts can be found.
   

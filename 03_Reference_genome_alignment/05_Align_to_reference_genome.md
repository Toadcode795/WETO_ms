# Align reads to reference genome (and sort)

## Background
After the reference genome is indexed, we will use the program **BWA** to align the reads to the reference genome. **BWA mem** is the algorithm to use when you have paired-end Illumina data. 
 
When using **BWA mem** the read alignments come out in the order that they are listed in the Fastq file. Therefore we want to sort the mapped alignments. We will use the program **Samtools** to sort the SAM files into a *coordinate-ordered* format which means that the alignments will be sorted by how they apppear in the reference genome. **Samtools** will also convert the SAM files to BAM format (BAM is the compressed binary form of SAM). 

This script is written so the outputs from **BWA mem** are directly piped into **samtools** using `|`. This means that the SAM file from **BWA mem** dosen't have to be saved which saves memory space.  

Finally, this script is written as an array job. This allows us to submit each individual job at one time. 

### Inputs
1) Path to indexed reference genome
2) Path to read 1 
3) Path to read 2
<br>

**NOTE:** Because this script is written as an array job, we will genetate a text document that has tab separated columns which will indicate the paths to read1, read2, and the reference genome. This text document will also provide the sample ID and the reference information.

### Flags
**BWA:**  
`-t` Number of threads to use  
`-M` Mark shorter split hits as secondary (used for Picard compatibility)   
`-R` Indicates the complete read group header line (read group information). @RG is the header tag, ID is the individual, SM is the sample (These two will be identicical if the individual is only sampled once), PL is the sequencing info.  
<br>
**Samtools:**  
`-@` Number of threads to use  
`-o` output file name

## Running BWA and Samtools
1. Generate the text file for the array script to run properly:  \
generating_map_array_file.sh
```
#!/bin/bash


ls fastp_WETO_plate2/fastp_trimmed_Q30/*.fastq.gz > samples_files_list.txt

exec 3< samples_files_list.txt

exec 4> mapped_files_list.txt


# Read each line from the input file
while IFS= read -r file1_path <&3 && IFS= read -r file2_path <&3; do

    # Extract the base filenames
    file1=$(basename "$file1_path")
    file2=$(basename "$file2_path")

    # Extract the common part from the filenames
    common=$(echo "$file1" | sed 's/\..*//')

    # Output the formatted line with filenames on the same line
    echo "$file1_path $file2_path home/jbergman/projects/def-leeyaw-ab/jbergman/index_WETO_ref_genome/WETO_reference $common WETO_ref" >&4

done

exec 3<&-
exec 4>&-
```
2. Run the array script to map the reads to the reference genome:  
array_map_genome.sh
```
#!/bin/bash
#SBATCH -c 4
#SBATCH --mem=128GB
#SBATCH --time=2-12:00
#SBATCH --account=NAME
#SBATCH --array=1-48
#SBATCH -o map_%A_%a.out
#SBATCH -e map_%A_%a.err
#SBATCH --mail-type=BEGIN,END,FAIL
#SBATCH --mail-user=EMAIL

readinfo=`sed -n -e "$SLURM_ARRAY_TASK_ID p" $1`

IFS=' ' read -r -a readarray <<< "$readinfo"

read_1=${readarray[0]}
read_2=${readarray[1]}
ref=${readarray[2]}
rg_info=${readarray[3]}
ref_info=${readarray[4]}

module load StdEnv/2020

module load bwa
module load samtools

bwa mem -t 4 -M -R $(echo "@RG\tID:$rg_info\tSM:$rg_info\tPL:Illumina") \
    $ref $read_1 $read_2 | samtools sort -@ 4 -o $rg_info.$ref_info.sort.bam 
```
command line
```
sbatch ~/scripts/array_map_genome.sh mapped_files_list.txt
```

### Outputs
BAM file for each individual sample. ex: AS-3-DNA80.WETO_ref.sort.bam

## Notes
Normally at this step we would want to index the aligned reads (using **samtools**). However, we cannot index the alignments because the scaffolds in the western toad genome are too large to be built in the BAM format.  

## Read counts
To determine how many reads aligned to the reference genome we will use the program **samtools** and the command **view**. The flag `-c` indicates "to count".  
  
`-F 260` Only counts mapped primary aligned reads (which are unique alignments that are not aligned to anywhere else in the genome).  
  

bam_file_counts.sh
```
#!/bin/bash
#SBATCH -c 4
#SBATCH --mem=8GB
#SBATCH --time=05:00:00
#SBATCH --account=NAME
#SBATCH --mail-type=BEGIN,END,FAIL
#SBATCH --mail-user=EMAIL


module load samtools

input_files="/home/jbergman/projects/def-leeyaw-ab/jbergman/plate2/BAM_WETO_plate2/ref_aligned/"

output_file="read_count_F260.txt"

for bam_file in "$input_files"/*.bam; do

        filename=$(basename "$bam_file" .bam)

        read_count=$(samtools view -c -F 260 "$bam_file")

        echo "$filename $read_count" >> "$output_file"

done
```
### Outputs
A text file was generated with the sample name and the number of reads that were aligned to the reference genome. This textfile was downloaded to my desktop and the reads from each individual were summed.  \
  \
**Total number of aligned reads = 917,788,446**

### NOTES
Also used `samtools view -c -F 4` to determine the total number of mapped reads = **917,788,446**

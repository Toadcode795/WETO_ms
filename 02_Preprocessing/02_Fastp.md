# FastP

## Background
After the reads are demultiplexed, we have Fastq files for each individual. These Fastq files need to go through preprocessing steps to ensure quality control before they can be used for any downstream analysis. These steps typically include 1) trimming adapters, and 2) prunning data that does not meet set quality thresholds. To process the fastq files in a single step we will use the program **fastp**.  \
  \
The following script is written as an array job which allows many jobs to be submitted at once through a single script.     
  
### Inputs
1. Demultiplexed fastq file for each individual.
2. A list of the individual file names (as a txt file). This is needed to submit the script as an array job.
### Flags
`-i` Read1 input file name  \
`-I` Read2 input file name  \
`-o` Read1 output file name  \
`-O` Read2 output file name  \
`-j` Saves the output as a json format and sets the file name.  \
  \
`f` This value indicates how many bases to trim at the front end of read1. This is needed when there is restriction site contamination on the reads. To know which value to set this to, manually look at the demultiplexed read files and remove the number of bases that are identical and shared at the beginning of all the reads. This value was set to **5**.  \
  \
`F` This value indicates how many bases to trim at the front end of read2 (similar to `f`). This value was set to **3**.  \
  \
`--cut_right` Moves the sliding window from the front of the read to the tail when assessing read quality. If the window reaches quality that is below one of the given thresholds, than the bases in the window and to the right of the window (the front end of the read) will be dropped. 
  
`-q` Is the the quality value that a base is qualified as. Default phred score >=Q15.

## Running Fastp
1) Create a text file with the list of demultiplexed files.

generating_fastp_file_list.sh
```
#!/bin/bash

ls process_radtags_WETO_plate2/*.fq.gz | grep -v "rem\." > samples_files_list.txt

exec 3< samples_files_list.txt
exec 4> fastp_files_list.txt

# Read each line from the input file
while IFS= read -r file1_path <&3 && IFS= read -r file2_path <&3; do

    # Extract the base filenames
    file1=$(basename "$file1_path")
    file2=$(basename "$file2_path")

    # Extract the common part from the filenames
    common=$(echo "$file1" | sed 's/\..*//')

    # Output the formatted line with filenames on the same line
    echo "$file1_path $file2_path $common" >&4

done

exec 3<&-
exec 4>&-
```
2) Run fastp as a diagnostic tool to initially assess reads and decide what to set `f` and `F` to. Once fastp is ran the outputs need to be looked at in multiqc (see *03_multiqc.md*).
  
array_fastp_diagnostic.sh
```
#!/bin/bash
#SBATCH -c 4
#SBATCH --mem=32GB
#SBATCH --array=1-48
#SBATCH --time=0-8:00
#SBATCH --account=NAME
#SBATCH -o arr_fastp_%A_%a.out
#SBATCH -e arr_fastp_%A_%a.err
#SBATCH --mail-type=BEGIN,END,FAIL
#SBATCH --mail-user=EMAIL

module load StdEnv/2020
module load fastp/0.23.4

readinfo=`sed -n -e "$SLURM_ARRAY_TASK_ID p" $1`

IFS=' ' read -a namearr <<< $readinfo

fastp --thread 4 -i ${namearr[0]} \
      -I ${namearr[1]} -o ${namearr[0]%.fastq.gz}_T.fastq.gz \
      -O ${namearr[1]%.fastq.gz}_T.fastq.gz -h ${namearr[2]}.html \
      -j ${namearr[2]}.fastp.json
```
3) Run fastp with the appropriate flags and settings to filter and trim reads
  
array_fastp.sh
```
#!/bin/bash
#SBATCH -c 4
#SBATCH --mem=32GB
#SBATCH --array=1-48
#SBATCH --time=0-8:00
#SBATCH --account=NAME
#SBATCH -o arr_fastp_%A_%a.out
#SBATCH -e arr_fastp_%A_%a.err
#SBATCH --mail-type=BEGIN,END,FAIL
#SBATCH --mail-user=EMAIL

module load StdEnv/2020
module load fastp/0.23.4

readinfo=`sed -n -e "$SLURM_ARRAY_TASK_ID p" $1`

IFS=' ' read -a namearr <<< $readinfo

fastp -f 5 -F 3 -q 20 --cut_right --thread 4 -i ${namearr[0]} \
      -I ${namearr[1]} -o ${namearr[0]%.fastq.gz}_T.fastq.gz \
      -O ${namearr[1]%.fastq.gz}_T.fastq.gz -h ${namearr[2]}.html \
	  -j ${namearr[2]}.fastp.json
```
command line
```
sbatch scripts/array_fastp.sh fastp_WETO_plate2/fastp_files_list.txt 
```
### Outputs
Every individual will have two fastq files and two summary files:
1) sample_name.1.fq.gz_T.fastq.gz (Read1 trimmed fastq file)
2) sample_name.2.fq.gz_T.fastq.gz (Read2 trimmed fastq file)
3) sample_name.json
4) sample_name.html

## Notes
I ran fastp testing out different read quality scores:
| Phred score (Q) | Number of reads retained |
| --- | --- |
| 15 (default) | 929,620,816 |
| 20 | 929,487,304 |

We continued with the pipeline using Q >= 20 
 

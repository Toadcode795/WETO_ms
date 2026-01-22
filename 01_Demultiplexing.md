# Demultiplexing  
  
## Background  
  
The fastq files that were received from Genome Qubec represent a single multiplexed plate of sequences (see *Project and data overview* for more details on the data). The first thing we have to do to process the raw reads is demultiplex the reads in **STACKS** using the **process_radtags** program. This program will sort the raw reads using the unique barcodes to recover the individual samples from the library.   
  
### Inputs   
1) The foward (R1) raw fastq.gz file from Genome Quebec
2) The reverse (R2) raw fastq.gz file from Genome Quebec
3) Barcode file: The program needs to be told which barcodes to expect. The barcodes will be specific for the enzyme pair that was used during library prep (Barcode list is provided by Laval). The barcode file will be a text file (.txt) with one or two columns that are separated by a tab. The first column is the individual barcodes and the second column (optional) is used to rename the output files.
  
### Flags  
`-o` path to output folder  
`-1` R1 input file (fastq.gz)  
`-2` R2 input file (fastq.gz)  
`-b` barcode file  
`--renz-1` first restriction enzyme used in library prep  
`--renz-2` second restiction enzyme used in library prep  
`--inline-null` indicates that the barcodes are only on the foward read and is inline with the sequence  
`-r` rescues barcodes and RAD-Tag cut sites (if there are sequencing errors in the read they get corrected so they can be properly recognized or the read is discareded)  
`-c` cleans data by removing any reads that have an uncalled base  
`-q` discards reads with low quality scores (default threshold is a Phred score of 10)  
`-D` writes a file with the discarded reads so we don't lose this information (output files with *.rem*)

`--adapter-1` provides the adapter sequence for the frist read  
`--adapter-2` provides the adapter sequence for the paired read  
`--adapter-mm` sets the number of mismatches allowed in the adapter sequences
## Running process_radtags
process_radtags.sh

```
#!/bin/bash
#SBATCH -c 1
#SBATCH --mem=64GB
#SBATCH --account=NAME
#SBATCH --time=2-12:00
#SBATCH -o proc_radt_%A.out
#SBATCH -e proc_radt_%A.err
#SBATCH --mail-type=BEGIN,END,FAIL
#SBATCH --mail-user=EMAIL

r1file=$1
r2file=$2
outfolder=$3
barcodes=$4
renz1=$5
renz2=$6

mkdir -p $outfolder

~/local/bin/process_radtags --threads 8 -o $outfolder -1 $r1file \
    -2 $r2file -b $barcodes --renz-1 $renz1 --renz-2 $renz2 \
    --inline-null -r -c -q -D --adapter-1 AGATCGGAAGAG --adapter-2 AGATCGGAAGAG --adapter-mm 1

```
command line
```
sbatch ~/projects/def-leeyaw-ab/jbergman/scripts/
process_radtags.sh WETO_plate2_rawdata/NS.X0110.008.B704---B504.LeeYaw_20240513_WETOplate2-BC49-96_R1.fastq.gz WETO_plate2_rawdata/NS.X0110.008.B704---B504.LeeYaw_20240513_WETOplate2-BC49-96_R2.fastq.gz process_radtags_plate2 WETO_plate2_rawdata/WETO_plate2_barcodes.txt pstI mspI
```
### Outputs
**process_radtags.log**: This has important summary information like *total_raw_read_counts* and *per_barcode_raw_read_counts*. See below the *total_raw_read_counts* which tells you the percent of reads retained and the percent discarded. The values shown below indicate that there are not any significant issues with the inital read quality (i.e. a high percentage of reads are retained and properly paired).
```
Total Sequences 1506075128
Reads containing adapter sequence    525075960     34.9%
Barcode Not Found                      4883500      0.3%
Low Quality                            3013848      0.2%
RAD Cutsite Not Found                  1319987      0.1%
Retained Reads                       971781833     64.5%
Properly Paired                      474622345     63.0%
```
Four files for each individual:
1) sample_name.1.fq (forward reads that will be kept)
2) sample_name.2.fq (reverse reads that will be kept)
3) sample_name.rem.1.fq (forward reads that were removed because they didn't meet the flag requirements)
4) sample_name.rem.2.fq (reverse reads that were removed because they didn't meet the flag requirements)

## Notes
Tested running **process_radtags** with defining adapter sequences (`--adapter-1 AGATCGGAAGAG --adapter-2 AGATCGGAAGAG`) and without defining the adapters (see above code).  

| Type of run | Adapter sequences defined | Adapters not defined |
|:--- | ---:| ---:|
| Starting number of reads | 1,506,075,128 | 1,506,075,128 |
| Reads containing adapter sequence | 525,075,960 (34.9%) |	NA |
| Barcode not found | 4,883,500 (0.3%) | 4,883,500 (0.3%) |
| Low quality | 3,013,848 (0.2%) | 3,013,848 (0.2%) |
| RAD cutsite not found | 1,319,987 (0.1%) | 1,319,987 (0.1%) |
| Retained reads | 971,781,833 (64.5%) | 1,496,857,793 (99.4%) |
| Properly paired | 474,622,345 (63.0%) | 747,122,727 (99.2%) |

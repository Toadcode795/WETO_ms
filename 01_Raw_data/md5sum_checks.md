# md5sum checks

## Background

After you download fastq files from Nanuq it is important to check the integrity of the files and make sure that nothing was changed or got corrupted during the download. This is done using the **.md5** file (Message-Digest Algorithm 5) which is also downloaded from Nanuq (separately from the fastq files). 

### Inputs
1) The forward (R1) raw fastq.gz file
2) The reverse (R2) raw fastq.gz file
3) readSet.md5 (for this dataset renamed to WETO_plate2.md5)

## Running checksum
In the command line:
```
md5sum NS.X0110.008.B704---B504.LeeYaw_20240513_WETOplate2-BC49-96_R1.fastq.gz

md5sum NS.X0110.008.B704---B504.LeeYaw_20240513_WETOplate2-BC49-96_R2.fastq.gz
```

### Output
We have the output from the command lines:
```
469a83e62e6a2ae883a86988cce12bfb  NS.X0110.008.B704---B504.LeeYaw_20240513_WETOplate2-BC49-96_R1.fastq.gz

92952b2ccbb50cc5f099a42a3a9b06ea  NS.X0110.008.B704---B504.LeeYaw_20240513_WETOplate2-BC49-96_R2.fastq.gz
```

Then we can look at the **WETO_plate2.md5** file contents:
```
469a83e62e6a2ae883a86988cce12bfb  NS.X0110.008.B704---B504.LeeYaw_20240513_WETOplate2-BC49-96_R1.fastq.gz
92952b2ccbb50cc5f099a42a3a9b06ea  NS.X0110.008.B704---B504.LeeYaw_20240513_WETOplate2-BC49-96_R2.fastq.gz
```
We see that the 32 character **Hash value** is identical from the command line output and the .md5 file. Therefore we can conclude that the fastq files have not been altered or damanged. 

## Check fastq file size 

We can additionally check the size of the fastq files to ensure that they aren't smaller than we are expecting (indicating something happened during the download) and that the size of the fastq files for R1 and R2 are relatively similar:
```
du -h  NS.X0110.008.B704---B504.LeeYaw_20240513_WETOplate2-BC49-96_R1.fastq.gz
du -h  NS.X0110.008.B704---B504.LeeYaw_20240513_WETOplate2-BC49-96_R2.fastq.gz
```

### Output
The size of the files are:
1) 53G
2) 54G

These are both large files and relatively similar in size. 

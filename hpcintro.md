# prerequites
- myriad account https://www.rc.ucl.ac.uk/docs/Clusters/Myriad/
- gitbash   https://gitforwindows.org/
- cyberduck https://cyberduck.io/
- vscode    https://code.visualstudio.com/download

# log in to myriad using ssh with password
Run this in GitBash or Mac/Linux terminal, replacing `ccaervi` with your UCL username
```
ssh ccaervi@myriad.rc.ucl.ac.uk
```

- https://www.rc.ucl.ac.uk/docs/Clusters/Myriad/
- home/scratch 1TB not backed up - `gquota` command
- ACFS 1TB backed up

# create or edit .ssh/config
Edit using VSCode, change `ccaervi` to your myriad username and `vicker` to your local username
```
Host myriad
  User ccaervi
  HostName login12.myriad.rc.ucl.ac.uk
  IdentityFile C:\Users\vicker\.ssh\id_ed25519_myriad
```

# create ssh public-private key pair
Run this command in GitBash or Mac/Linux terminal
```
ssh-keygen -t ed25519 -a 100 -f ~/.ssh/id_ed25519_myriad
```

# copy key to myriad
```
ssh-copy-id -i ~/.ssh/id_ed25519_myriad.pub myriad
```

# run ssh agent
```
eval $(ssh-agent -s)
ssh-add ~/.ssh/id_ed25519_myriad
ssh-add -L
```

# login without password

Be sure to do this from the same terminal as ssh-agent was started from
```
ssh myriad
ls -l
cd Scratch
mkdir hpcintro
ls -l
cd hpcintro
```

# download test data
Run this locally, we will then copy to myriad
```
curl --output GCF_000010245.2_ASM1024v1_genomic.fna.gz https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/010/245/GCF_000010245.2_ASM1024v1/GCF_000010245.2_ASM1024v1_genomic.fna.gz
curl --output SRR31145988.fastq.gz https://trace.ncbi.nlm.nih.gov/Traces/sra-reads-be/fastq?acc=SRR31145988
```

# Cyberduck file transfer
- Open Connection
- sftp
- myriad
- recommend to not save password
- Upload GCF_000010245.2_ASM1024v1_genomic.fna.gz and SRR31145988.fastq.gz
- save into Scratch/hpcintro

# run a first job on myriad
Log into myriad then use nano to create a script file
```
nano test.sh
```
Enter this into the file
```
#!/bin/bash
#$ -cwd
#$ -pe smp 1
#$ -l h_rt=1:0:0
#$ -l mem=4G

echo Starting...
date
sleep 20
hostname
pwd

echo Finished
date
```

Submit the script and monitor it
```
qsub test.sh
watch qstat
```
Use <CTRL-C> to exit `watch`

While your job is still running you can monitor or kill it using, subtituting your own username and job id number
```
qstat -f -j 37656
qdel -j 37656
qdel -u ccaervi
```
After the job has finished you can retrieve information using, again subtituting your own username and job id number
```
jobhist
qacct -u ccaervi -j 37656
```

During or after the job has run you can view the output and error messages it is producing using
```
less test.sh.e37656
less test.sh.o37656
```

# bwa read mapping example
Create a simplified alias for the genome and read files
```
ln -s GCF_000010245.2_ASM1024v1_genomic.fna.gz genome.fasta.gz
ln -s SRR31145988.fastq.gz reads.fastq.gz
```

Use nano to create a script to index the genome
```
nano bwa_index.sh
```
Enter the following into the file
```
#!/bin/bash
#$ -cwd
#$ -pe smp 1
#$ -l h_rt=1:0:0
#$ -l mem=4G

module load bwa/0.7.12

echo Starting...
date

bwa index genome.fasta.gz

echo Finished
date
```
Submit the script using
```
qsub bwa_index.sh
```
Create a script to map and sort the reads
```
nano bwa_mem.sh
```
Enter the following into the script file
```
#!/bin/bash
#$ -cwd
#$ -pe smp 12
#$ -l h_rt=1:0:0
#$ -l mem=4G

module load bwa/0.7.12
module load samtools/1.11/gnu-4.9.2

echo Starting...
date

bwa mem -t 4 genome.fasta.gz reads.fastq.gz \
        | samtools view -S -b -@ 4 -o - - \
        | samtools sort -@ 4 -O BAM -o output.bam -

echo Finished
date
```

Submit the script
```
qsub bwa_mem.sh
```

Once done you can view the output
```
module load samtools/1.11/gnu-4.9.2
samtools view output.bam | less -S
```

# python script hello world
Create a simple python job script
```
nano python_hello.py
```
Enter the following
```
#!/usr/bin/env python
#$ -cwd
#$ -V
#$ -pe smp 11
#$ -l h_rt=1:0:0
#$ -l mem=4G

print("Hello world!")
```

Load the module for python3.11 before submitting the job
```
module load python3/3.11
qsub python_hello.py
```

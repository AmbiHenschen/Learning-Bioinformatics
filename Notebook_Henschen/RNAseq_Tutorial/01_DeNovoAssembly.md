## De novo transcriptome assembly with RNAseq data
### 12/2/2019
### Condo: /work/GIF/henschen/Learning_Bioinformatics/RNASeqTutorial/01_DeNovoAssembly



The goal of this tutorial is to  create a de novo transcriptome assembly using RNAseq data. This tutorial uses the assembler Trinity.

Download raw data if  you haven't already
```
# download raw data, this data is from the ENA and is RNAseq data from A. thaliana
# Download SRA files from ENA website (Note: this step is slow)
wget ftp://ftp.sra.ebi.ac.uk/vol1/fastq/SRR442/003/SRR4420293/SRR4420293_1.fastq.gz
wget ftp://ftp.sra.ebi.ac.uk/vol1/fastq/SRR442/003/SRR4420293/SRR4420293_2.fastq.gz
wget ftp://ftp.sra.ebi.ac.uk/vol1/fastq/SRR442/004/SRR4420294/SRR4420294_1.fastq.gz
wget ftp://ftp.sra.ebi.ac.uk/vol1/fastq/SRR442/004/SRR4420294/SRR4420294_2.fastq.gz
wget ftp://ftp.sra.ebi.ac.uk/vol1/fastq/SRR442/005/SRR4420295/SRR4420295_1.fastq.gz
wget ftp://ftp.sra.ebi.ac.uk/vol1/fastq/SRR442/005/SRR4420295/SRR4420295_2.fastq.gz
wget ftp://ftp.sra.ebi.ac.uk/vol1/fastq/SRR442/006/SRR4420296/SRR4420296_1.fastq.gz
wget ftp://ftp.sra.ebi.ac.uk/vol1/fastq/SRR442/006/SRR4420296/SRR4420296_2.fastq.gz
wget ftp://ftp.sra.ebi.ac.uk/vol1/fastq/SRR442/007/SRR4420297/SRR4420297_1.fastq.gz
wget ftp://ftp.sra.ebi.ac.uk/vol1/fastq/SRR442/007/SRR4420297/SRR4420297_2.fastq.gz
wget ftp://ftp.sra.ebi.ac.uk/vol1/fastq/SRR442/008/SRR4420298/SRR4420298_1.fastq.gz
wget ftp://ftp.sra.ebi.ac.uk/vol1/fastq/SRR442/008/SRR4420298/SRR4420298_2.fastq.gz
```

```
# grab an interactive node
salloc -N 1 -n 16 -t 8:00:00
```

fastqc
```
# need to run quality control on raw data using fastqc
# load necessary modules
# to see available modules
module avail
module load fastqc/0.11.7-d5mgqc7
module load parallel/20170322-36gxsog
#this reloads perl with a version change: 1) perl/5.24.1-ei7q6gg => perl/5.24.1-yuxpqbr

# make a folder for fastqc output
mkdir fastqcOutput

# run fastqc in parallel on all fastq files
parallel "fastqc {} -o fastqcOutput" ::: *.fastq.gz


# view fastqc output files by transferring them to your local machine
# /Users/amberleighhenschen/GithubRepos/Learning-Bioinformatics/RNAseq_Tutorial/01_DeNovoAssembly
rsync -avz -e ssh henschen@condo2017.its.iastate.edu:/work/GIF/henschen/Learning_Bioinformatics/RNASeqTutorial/01_DeNovoAssembly/fastqcOutput .
```

Multiqc
```
# use multiqc to aggregate fastqc outputs
#first need to load modules
module load miniconda3
conda config --add channels bioconda
conda config --add channels conda-forge
conda install datamash
conda create -n my_root --clone="/opt/rit/spack-app/linux-rhel7-x86_64/gcc-4.8.5/miniconda3-4.3.30-qdauvebbhrjaevrgwkt2wples4psz4w7"
source activate my_root
pip install --upgrade pip --user
load module python/3.8.0-k7w5uj4
conda install multiqc

#run multiqc within fastqcOutput file
# /work/GIF/henschen/Learning_Bioinformatics/RNASeqTutorial/01_DeNovoAssembly/fastqcOutput
multiqc .

#transfer multiqc files to local machine
#/Users/amberleighhenschen/GithubRepos/Learning-Bioinformatics/RNAseq_Tutorial/01_DeNovoAssembly
rsync -avz -e ssh henschen@condo2017.its.iastate.edu:/work/GIF/henschen/Learning_Bioinformatics/RNASeqTutorial/01_DeNovoAssembly/fastqcOutput .
```


If raw data is of sufficient quality we can now run Trinity to create our de novo transcriptome.

Trinity Singularity
```
#we will run Trinity in a Singularity container for consistency
#Grab the latest Trinity Singularity container from broad
wget https://data.broadinstitute.org/Trinity/TRINITY_SINGULARITY/trinityrnaseq.v2.8.6.simg
```

Find available node to run Trinity
```
#check to see which nodes are available
#alias si='sinfo -o "%20P %5D %14F %8z %10m %10d %11l %16f %N"'
si
```


SLURM Trinity script

```
#!/bin/bash
#SBATCH -N 1
#SBATCH -p short_1node
#SBATCH --ntasks-per-node=16 # For Trinity always reserve the whole node
#SBATCH -t 96:00:00
#SBATCH -J trinity
#SBATCH -o trinityARTH.o%j
#SBATCH -e trinityARTH.e%j
#SBATCH --mail-user=henschen@iastate.edu
#SBATCH --mail-type=begin
#SBATCH --mail-type=end

cd $SLURM_SUBMIT_DIR
ulimit -s unlimited


# load modules
module load singularity

# sync data
cd ${TMPDIR}
RC=1
date
while [[ $RC -ne 0 ]]; do
rsync -rvts --exclude=log $SLURM_SUBMIT_DIR/ $TMPDIR/
RC=$?
sleep 10
done
date

# Trinity requires that the each set of reads are concatenated into a file each. make combined left and right reads.
cat /work/GIF/henschen/Learning_Bioinformatics/RNASeqTutorial/01_DeNovoAssembly/*_1.fastq.gz > left_1.gz
cat /work/GIF/henschen/Learning_Bioinformatics/RNASeqTutorial/01_DeNovoAssembly/*_2.fastq.gz > right_2.gz

# running Trinity after mounting our working directory inside the container using $PWD.

singularity exec --bind $PWD trinityrnaseq.v2.8.6.simg Trinity --seqType fq --max_memory 120G --group_pairs_distance 1000 --min_kmer_cov 2 --CPU 32  --output TrinityOut --left left_1.gz --right right_2.gz --trimmomatic


RC=1
date
while [[ $RC -ne 0 ]]; do
rsync -rts $TMPDIR/ $SLURM_SUBMIT_DIR/
RC=$?
sleep 10
done
date
scontrol show job $SLURM_JOB_ID
```

Create a text file for submission and copy and paste script into this file
```
nano trinityARTH.sub
```

Submit job
```
sbatch trinityARTH.sub
```

Can check on the progress of current jobs, will also receive update emails
```
squeue -u henschen
````

 ### Output
Previous Trinity script resulted in error due to read names (example: SRR4420293.1 HWI-ST1136:361:HS250:1:1101:1130:2234/1)
"Error, not recognizing read name formatting: [SRR4420293.1]""


### Updated SLURM Trinity script
```
#!/bin/bash
#SBATCH -N 1
#SBATCH -p short_1node
#SBATCH --ntasks-per-node=16 # For Trinity always reserve the whole node
#SBATCH -t 96:00:00
#SBATCH -J trinity
#SBATCH -o trinityARTH.o%j
#SBATCH -e trinityARTH.e%j
#SBATCH --mail-user=henschen@iastate.edu
#SBATCH --mail-type=begin
#SBATCH --mail-type=end

cd $SLURM_SUBMIT_DIR
ulimit -s unlimited


# load modules
module load singularity

# sync data
cd ${TMPDIR}
RC=1
date
while [[ $RC -ne 0 ]]; do
rsync -rvts --exclude=log $SLURM_SUBMIT_DIR/ $TMPDIR/
RC=$?
sleep 10
done
date

# Trinity requires that the each set of reads are concatenated into a file each. make combined left and right reads.
cat /work/GIF/henschen/Learning_Bioinformatics/RNASeqTutorial/01_DeNovoAssembly/*_1.fastq.gz > left_1.gz
cat /work/GIF/henschen/Learning_Bioinformatics/RNASeqTutorial/01_DeNovoAssembly/*_2.fastq.gz > right_2.gz

# rename each run in the left_1.gz and right_2.gz files so they do not include SRR*
zcat left_1.gz | perl -pe 's/SRR.*\.[0-9]+ //' | gzip > left_1a.gz &
zcat right_2.gz | perl -pe 's/SRR.*\.[0-9]+ //' | gzip > right_2a.gz &

# running Trinity after mounting our working directory inside the container using $PWD.

singularity exec --bind $PWD trinityrnaseq.v2.8.6.simg Trinity --seqType fq --max_memory 120G --group_pairs_distance 1000 --min_kmer_cov 2 --CPU 32  --output TrinityOut --left left_1a.gz --right right_2a.gz --trimmomatic


RC=1
date
while [[ $RC -ne 0 ]]; do
rsync -rts $TMPDIR/ $SLURM_SUBMIT_DIR/
RC=$?
sleep 10
done
date
scontrol show job $SLURM_JOB_ID
```

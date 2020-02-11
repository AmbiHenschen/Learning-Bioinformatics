### Differential expression analyses
#### 12/5/2019

##### Condo: /work/GIF/henschen/Learning_Bioinformatics/RNASeqTutorial/01_DeNovoAssembly/TrinityOut


Goal: mapping RNAseq reads back to the assembly using bowtie2, calculating transcript abundance, using FeatureCounts and then performing differential expression using DESeq2


build a transcriptome index to align the RNAseq back to transcriptome and estimate the abundance of the transcripts

SLURM RSEM Script
```
#!/bin/bash
#SBATCH -N 1
#SBATCH -p short_1node
#SBATCH --ntasks-per-node=16
#SBATCH -t 24:00:00
#SBATCH -J RSEM
#SBATCH -o RSEMARTH.o%j
#SBATCH -e RSEMARTH.e%j
#SBATCH --mail-user=henschen@iastate.edu
#SBATCH --mail-type=begin
#SBATCH --mail-type=end

cd $SLURM_SUBMIT_DIR
ulimit -s unlimited


# load modules
module load singularity


# running RSEM after mounting our working directory inside the container using $PWD.

singularity exec --bind $PWD  trinityrnaseq.v2.8.6.simg /usr/local/bin/trinityrnaseq/util/align_and_estimate_abundance.pl  --transcripts /work/GIF/henschen/Learning_Bioinformatics/RNASeqTutorial/01_DeNovoAssembly/TrinityOut/Trinity.fasta --seqType fq --left /work/GIF/henschen/Learning_Bioinformatics/RNASeqTutorial/01_DeNovoAssembly/left_1a.gz --right /work/GIF/henschen/Learning_Bioinformatics/RNASeqTutorial/01_DeNovoAssembly/right_2a.gz --est_method RSEM --aln_method bowtie2 --trinity_mode  --prep_reference --max_ins_size 1000 --output_dir RSEM_dir1


scontrol show job $SLURM_JOB_ID
```


Build a transcript expression matrix for all samples using the abundance_estimates_to_matrix.pl script
Prefix all the output files with “all” and use the base name of the directory as sample names
```
#!/bin/bash
#SBATCH -N 1
#SBATCH -p short_1node
#SBATCH --ntasks-per-node=16
#SBATCH -t 24:00:00
#SBATCH -J RSEM
#SBATCH -o RSEM_ExMat.o%j
#SBATCH -e RSEM_ExMat.e%j
#SBATCH --mail-user=henschen@iastate.edu
#SBATCH --mail-type=begin
#SBATCH --mail-type=end

cd $SLURM_SUBMIT_DIR
ulimit -s unlimited


# load modules
module load singularity

singularity exec --bind $PWD trinityrnaseq.v2.8.6.simg /usr/local/bin/trinityrnaseq/util/abundance_estimates_to_matrix.pl --est_method RSEM --name_sample_by_basedir --gene_trans_map /work/GIF/henschen/Learning_Bioinformatics/RNASeqTutorial/01_DeNovoAssembly/TrinityOut/Trinity.fasta.gene_trans_map --out_prefix all /work/GIF/henschen/Learning_Bioinformatics/RNASeqTutorial/01_DeNovoAssembly/RSEM_dir1/RSEM.isoforms.results
```


Move output files to local machine
```
rsync rsync -avz -e ssh henschen@condo2017.its.iastate.edu:/work/GIF/henschen/Learning_Bioinformatics/RNASeqTutorial/01_DeNovoAssembly/TrinityOut
all.gene.TMM.EXPR.matrix
all.gene.TPM.not_cross_norm.TMM_info.txt
all.isoform.TMM.EXPR.matrix
all.isoform.TPM.not_cross_norm.TMM_info.txt
all.gene.counts.matrix
all.gene.TPM.not_cross_norm
all.isoform.counts.matrix
all.isoform.TPM.not_cross_norm .
```

Next steps done in Rstudio

```
# load modules
source("http://bioconductor.org/biocLite.R")
biocLite("DESeq2")
library("DESeq2")
```

```
# set working directory and read in data table

setwd(/Users/amberleighhenschen/GithubRepos/Learning-Bioinformatics/RNAseq_Tutorial/02_DiffExAnalysis
)
dat<-read.table("At_count.txt",header = T,quote = "",row.names = 1) (where does this file come from?)

# Convert to matrix
dat <- as.matrix(dat)
head(dat)

# Assign condition (first three are WT, next three are mutants)

condition <- factor(c(rep("WT",3),rep("Mut",3)))
condition=relevel(condition,ref = "WT")


# Create a coldata frame: its rows correspond to columns of dat (i.e., matrix representing the countData)
coldata <- data.frame(row.names=colnames(dat), condition)

head(coldata)

#            condition
# S293        WT
# S294        WT
# S295        WT
# S296       Mut
# S297       Mut
# S298       Mut


##### DESEq pipeline, first the design and the next step, normalizing to model fitting
dds <- DESeqDataSetFromMatrix(countData = dat, colData = coldata,design=~ condition)


dds <- DESeq(dds)

# Plot Dispersions:
png("qc-dispersions.png", 1000, 1000, pointsize=20)
plotDispEsts(dds, main="Dispersion plot")
dev.off()
```

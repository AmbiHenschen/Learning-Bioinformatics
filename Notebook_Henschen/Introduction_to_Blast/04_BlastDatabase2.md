#Creating a Blast database of S. rivoliana genome assemblies
##11/15/2019
##/Users/amberleighhenschen/GithubRepos/Learning-Bioinformatics/IntroductionToBlast/00_S.rivolianaGenomeAssemblies




Create a blast database
```
makeblastdb -in S.rivolianaGenomeAssemblies -dbtype nucl -input_type fasta -out SerRivdb
```
Output: Building a new DB, current time: 11/15/2019 12:20:36
New DB name:   /Users/amberleighhenschen/GithubRepos/Learning-Bioinformatics/IntroductionToBlast/00_S.rivolianaGenomeAssemblies/SerRivdb
New DB title:  S.rivolianaGenomeAssemblies
Sequence type: Nucleotide
Keep MBits: T
Maximum file size: 1000000000B
Adding sequences from FASTA; added 1343 sequences in 10.2362 seconds.

New files
  1. SerRivdb.nhr
  2. SerRivdb.nin
  3. SerRivdb.nsq

Move these files to appropriate directory
```
mv SerRivdb.nhr SerRivdb.nin SerRivdb.nsq /Users/amberleighhenschen/GithubRepos/Learning-Bioinformatics/IntroductionToBlast/04_BlastDatabase2
```

copy benediniaGene.fasta file to appropriate directory
```
cp benediniaGene.fasta /Users/amberleighhenschen/GithubRepos/Learning-Bioinformatics/IntroductionToBlast/04_BlastDatabase2
```

change working directory
```
cd  /Users/amberleighhenschen/GithubRepos/Learning-Bioinformatics/IntroductionToBlast/04_BlastDatabase2
```

Blast the C-lectin protein sequence and put the output into a new file, blastout.txt
```
tblastn -db SerRivdb -query benediniaGene.fasta  -out blastout.txt
```

Use nano to look at new file
```
nano blastout.txt
```


Create a second file, blastout2.txt, with out specific outputs
```
tblastn -db SerRivdb -query benediniaGene.fasta  -outfmt '6 qseqid sseqid pident length mismatch gapopen qstart qend sstart send evalue bitscore qcovs salltitles' -num_threads 16 -out blastout2  .txt
```

Filter only results that have at least 50% identity with our protein sequences
```
more blastout2.txt  | awk '$3>50'
```

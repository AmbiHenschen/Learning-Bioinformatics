## Trinity Assembler

### 12/4/2019
---
Current version (12/4/2019): [v2.8.6](https://github.com/trinityrnaseq/trinityrnaseq/releases)

---

###### Relevant articles:
Trinity is referenced as
* Grabherr, M., Haas, B., Yassour, M. et al. Full-length transcriptome assembly from RNA-Seq data without a reference genome. Nat Biotechnol 29, 644–652 (2011) doi:10.1038/nbt.1883

Protocol for using Trinity for de novo transcriptome assembly and downstream analyses:
* Haas, B., Papanicolaou, A., Yassour, M. et al. De novo transcript sequence reconstruction from RNA-seq using the Trinity platform for reference generation and analysis. Nat Protoc 8, 1494–1512 (2013) doi:10.1038/nprot.2013.084

Performance tuning of Trinity is described in:
* Henschel R, Lieber M, Wu L, Nista, PM, Haas BJ, LeDuc R. Trinity RNA-Seq assembler performance optimization. XSEDE 2012 Proceedings of the 1st Conference of the Extreme Science and Engineering Discovery Environment: Bridging from the eXtreme to the campus and beyond. ISBN: 978-1-4503-1602-6 doi: 10.1145/2335755.2335842.

---

###### Useful Links

Information about Trinity can be found [here](https://github.com/trinityrnaseq/trinityrnaseq/wiki) and [here](https://github.com/trinityrnaseq/trinityrnaseq/wiki/Trinity-Transcript-Quantification#filtering-transcripts).

Videos describing Trinity's use can be found [here](https://www.broadinstitute.org/broade/trinity-screencast).

[ELI5 de Bruijn graphs](https://www.reddit.com/r/explainlikeimfive/comments/2mifiz/eli5_de_bruijn_graphs/)

---
###### Details

**De novo transcriptome assembly**

Trinity was developed to create de novo transcriptomes for species without sequenced genomes (or whose genomes are incomplete or fragmented). In particular the goal of Trinity is to address the following issues:

1. Some transcripts have low coverage, whereas others are highly expressed
2. read coverage may be uneven across the transcript's length, owing to sequencing biases
3. reads with sequencing errors derived from a highly expressed transcript may be more abundant than correct reads from a transcript that is not highly expressed
4. transcripts encoded by adjacent loci can overlap and thus can be erroneously fused to form a chimeric transcript
5. data structures need to accommodate multiple transcripts per locus, owing to alternative splicing
6. sequences that are repeated in different genes introduce ambiguity



Trinity consists of three software modules: Inchworm, Chrysalis and Butterfly (Grabherr et al. 2011, Fig. 1)
![Figure from Grabherr et al. 2011](https://media.springernature.com/full/springer-static/image/art%3A10.1038%2Fnbt.1883/MediaObjects/41587_2011_Article_BFnbt1883_Fig1_HTML.gif?as=webp)
Inchworm (a) assembles reads into the unique sequences of transcripts
Chrysalis (b) clusters related contigs that correspond to portions of alternatively spliced transcripts or otherwise unique portions of paralogous genes
Butterfly (c) analyzes the paths taken by reads and read pairings in the context of the corresponding de Bruijn graph and reports all plausible transcript sequences, resolving alternatively spliced isoforms and transcripts derived from paralogous genes.


**Trinity Transcript Quantification**

Several methods available for estimating transcript abundance in a genome-free manner

-include alignment-based methods (aligning reads to the transcript assembly) and alignment-free methods (typically examining k-mer abundances in the reads and in the resulting assemblies)

Alignment-based methods

-RSEM and eXpress

Alignment-free methods

-kallisto and salmon

Any of the abundance estimation methods will provide transcript-level estimates of the count of RNA-Seq fragments that were derived from each transcript, in addition to a normalized measure of transcript expression that takes into account the transcript length, the number of reads mapped to the transcript, and the the total number of reads that mapped to any transcript.


The estimated fragment counts are generally needed by many differential expression analysis tools that use count-based statistical models, and the normalized expression values (FPKM or TPM) are used almost everywhere else, such as plotting in heatmaps.

**FPKM**: fragments per kilobase transcript length per million fragments mapped  

**TPM**: transcripts per million transcripts. The TPM metric is generally preferred to FPKM, given the property that all values will always sum up to 1 million (FPKM values will tend to not sum up to the same value across samples).




##### Important notes:

If you have multiple RNA-Seq data sets that you want to compare (eg. **different tissues sampled from a single organism** ), be sure to generate a single Trinity assembly and to then run the abundance estimation separately for each of your samples.

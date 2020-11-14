# Analysis of Splicing Efficiency based on RNA-seq data

This repository contains the script I used for my custom analysis of RNA-seq data, i.e., computation of Splicing Efficiency and scores of sequences similarity.

## Computation of Splicing Efficiency

The method used to measure splicing efficiency (SE) of detected yeast introns was adapted and modified from Xia, 2020 [2]. The implementation of the algorithm was developed in a collaboration with dr. Marcin Magnus.
Reads mapped to a particular location in the genome were counted using SAMtools (v1.9, Li et al., 2009 [1]) and a custom Python script.
The splice-junction locations were defined by the coordinates generated earlier via JunctionSeq, and only SJs, which passed JunctionSeq cut-off (annotated and novel, including 5'UTR introns; n = 321 after filtering out non-GT-AG introns) were considered in calculations.

Briefly, SE is calculated as:

*SE= N_spliced/N_total*  

where:

*N_total= N_spliced + N_EI*


N_spliced is a number of reads that map to splice-junction (reads spanning from exon to exon with a gap in the middle, i.e., spliced RNAs). 
N_EI is a number of reads that map to exon-intron junction (unspliced RNAs) at either 5'SS or 3'SS (often not equal, which can be related to e.g., used library preparation protocol – read below).
The assumption is that N_spliced and N_EI will change simultaneously with different read coverage for a given gene, so there is no need to use complex methodology to normalize the results.

The number of reads aligned to 5'SS and to 3'SS often differ (Zhang et al. [3], 2018; Xia, 2020 [2]) with the poorer coverage at 5'SS, especially when utilizing poly(A) enriched library preparation protocol. The reason is that the 5' ends of mRNAs may have been lost (e.g., due to degradation or reverse transcriptase failing to reach the 5' end of a transcript), and so are excluded from sequencing. 
To compensate for the bias between N_EI5 (exon-intron junction at 5'SS) and N_EI3 (intron-exon junction at 3'SS) the strategy proposed by X. Xia in his work was applied (Xia, 2020). Namely, the proportion *p*, which corrects for the potential differences in splice sites coverage was introduced: 

*p =   N_EI5/(N_EI5 +  N_EI3)*
    
and it was applied when computing N_total:

*N_total  = N_EE  + p * N_EI5  + (1 – p)* N_EI3*

Most of the first exons in yeast are very short, e.g., they comprise only AUG codon. Differences in read coverage at 5'SS and 3'SS of such transcripts are even more pronounced. Therefore, for all the genes with first exon length below 30 nt, the *p* parameter was set as 0 and Ntotal  was calculated as:

*N_total  = N_EE + N_EI3*

Splicing efficiency (SE) of each individual intron was calculated independently for each biological triplicate. Then, for each intron a median of three SE values was calculated and used as the final SE value of a given intron in all downstream analyses. 


## Scores of sequence similarity

To define and assess the deviation of all identified splice site sequences from the consensus sequence, the Levenshtein distance algorithm was used. The implementation of the algorithm was done in collaboration with dr. Marcin Magnus. 

The Levenshtein distance value describes the minimal number of deletions, insertions or substitutions that are required to transform one string, i.e., splice site sequences of a given intron, into another, i.e., canonical splice site sequence. The consensus sequence for such comparisons was set as a 16-nt long string of the canonical 5' splice site, branch site and 3' splice site sequences combined, i.e., GTATGT / TACTAAC / YAG (Y is any pyrimidine). For instance, the Levenshtein distance between sequences “GTATGTTAtTAACaAG” and “GTATGTTACTAACTAG”, is 2 and the Levenshtein distance ratio (i.e., Levenshtein distance / alignment length) is 0.875.




## Literature

* [1] Li, Heng, Bob Handsaker, Alec Wysoker, Tim Fennell, Jue Ruan, Nils Homer, Gabor Marth, Goncalo Abecasis, and Richard Durbin. 2009. “The Sequence Alignment/Map Format and SAMtools.” Bioinformatics 25 (16): 2078–79. https://doi.org/10.1093/bioinformatics/btp352.*

* [2] Xia, Xuhua. 2020. “RNA-Seq Approach for Accurate Characterization of Splicing Efficiency of Yeast Introns.” Methods 176 (March): 25–33. https://doi.org/10.1016/j.ymeth.2019.03.019.*

* [3] Zhang, Albert Y., Shian Su, Ashley P. Ng, Aliaksei Z. Holik, Marie-Liesse Asselin-Labat, Matthew E. Ritchie, and Charity W. Law. 2018. “A Data-Driven Approach to Characterising Intron Signal in RNA-Seq Data.” BioRxiv. https://doi.org/10.1101/352823.*

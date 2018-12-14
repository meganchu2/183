# 3.2.2 GITAR Pipeline 
1. [Overview](#1)
2. [Data Preprocessing](#2)
3. [Data Normalization](#3)
4. [Data Visualization](#4)
5. [TAD Analysis and Visualization](#5)


## Overview<a name="1"></a>

GITAR (Genome Interaction Tools and Resources) is a software to perform a comprehensive Hi-C data analysis.

> The entire protocol is outlined at [https://doc.genomegitar.org](https://doc.genomegitar.org).  Please visit this link if you are interested in learning GITAR in detail, or if you are interested in analyzing your own Hi-C data with GITAR.

In order to analyze and visualize data generated from Hi-C, the following steps are followed:
- Data preprocessing
- Data Normalization
- Data Visualization
- Topologically associated domain (TAD) Analysis and Visualization


## Data Preprocessing<a name="2"></a>

**Goal**: Improve the genome mapping outcome of Hi-C data.

<table>
  <tr>
    <td><img src="https://www.pastepic.xyz/images/2018/12/14/Untitled04d01bad2783e292.png" "Hi-C fragment"" width="700"/></td>
    <td>##### Figure 1. Schematic of two interacting DNA fragments (called DNA fragment 1 and DNA fragment 2) from the Hi-C protocol. **CHANGE THIS!!Figure by Sotelo-Silveira, Mariana, et al. Trends in Plant Science (2018).**</td>
  </tr>
  <tr>
    <td><img src="https://www.pastepic.xyz/images/2018/12/14/Untitled3d84a95c29cb5ccd2.png" "ligation site"" width="700"/></td>
    <td>##### Figure 2. Schematic of reads generated from interacting DNA fragments from Figure 1 **borrowed from slides**</td>
  </tr>
</table>
<!---
![Untitled04d01bad2783e292.png](https://www.pastepic.xyz/images/2018/12/14/Untitled04d01bad2783e292.png "Hi-C fragment")
![Untitled3d84a95c29cb5ccd2.png](https://www.pastepic.xyz/images/2018/12/14/Untitled3d84a95c29cb5ccd2.png "ligation site")
 -->

**Problem 1**: Hi-C reads have the form depicted by Figure 1; each read pair contains one read for DNA fragment 1 and one read for DNA fragment 2.  However, it is also possible that each read in the read pair contains parts of the ligation site (in theory the ligation site is not a naturally occurring sequence in the genome), and this ligation site sequence would not map to anywhere in the genome.

***Solution 1 "Pre-truncation":***
	Remove any parts of the ligation site sequence contained in the reads.

**Problem 2**: We also realize that two interacting DNA fragments may be many kilobases away from each other, thus a read pair (one read for each interacting fragment) may not map well to the genome as "one sequence".

***Solution 2 "Independent Alignment":***
	Map each read in the read pair separately.
	*Output: SAM file* (solution1+2 output)

**Problem 3**: After mapping the reads, we realize it is possible that some reads are unable to be mapped to the genome, some read pairs map to the same DNA fragment (which doesn't make sense since this implies the DNA fragment interacts with itself), or some reads have a very low quality mapping (doesn't really match with the genome sequence it mapped to).

***Solution 3 "Filtering Reads":***
	Remove read pairs that fall within the same fragment or have MAPQ score less than a specified threshold.
	*Output: BAM file*


## Data Normalization<a name="3"></a>
**Goal**: Normalize the data for technical biases while taking into account biologial biases.

*What are technical biases?* Spurious ligation products, fragment length, GC content, mappability

*What are biological biases?* A biological bias is due to genomic features such as transcription start sites (TSSs) and CTCF binding sites.  This bias is evident in the figure below.  

![Untitled2940c6daf03d9f546.png](https://www.pastepic.xyz/images/2018/12/14/Untitled2940c6daf03d9f546.png)
##### Figure *: Notice that at DNA regions at around a distance of 100K (upstream and downstream) from a transcription start site have more contacts(interactions/read pairs) with that transcription start site, and DNA regions at around a distance of 200K (upstream and downstream) from a CTCF binding site have more contacts (interactions/read pairs) with that CTCF binding site.

The **learned correction parameters** are determined using a model from Yaffe and Tanay, and they will normalize the technical biases while trying to avoid normalizing the biological biases.

Our mapped read pairs can be organized in a format called the contact matrix.

![Untitled4e5d67cfb0178b41c.png](https://www.pastepic.xyz/images/2018/12/14/Untitled4e5d67cfb0178b41c.png "contact matrix")
##### Figure *: The left shows the general structure of a contact matrix where the rows represent loci (bins) on Chromosome i and the columns represent the loci (bins) on Chromosome j.  The entries of the contact matrix contain the number of contacts between each pair of loci.  The contact matrix can be used to generate a heatmap (example on the right of the the figure), where more contacts between a pair of loci is represented by a more intense color (in this example, white = 0 and red = 5).
**borrowed from slides**

**How to calculate normalized data**
- Observed contact matrix O[i,j]: observed read count b/w loci identified by bins i and j.
- Correction matrix C[i,j]: sum of corrections for read pairs b/w bins i and j.
- Normalized contact matrix N[i,j]: corrected/normalized contact counts calculated by N[i,j] = O[i,j] / C[i,j]

**Other relevant contact matrices**
- Expected contact matrix E[i,j]: contains expected read counts between loci identified by bins i and j, calculated by considering correction parameters and the linear distance between read pairs (the larger the distance, the lower the contact probability). 
- Observed over Expected contact matrix O_over_E[i,j]: As its name implies, it is calculated by dividing the observed read counts by the expected read counts, O_over_E[i,j] = O[i,j]/E[i,j]


## Data Visualization<a name="4"></a> 
**Goal**: Give visual representation of interchromosomal and intrachromosomal interactions.
- Heatmaps show contacts based on data from contact matrices
- Histograms show distribution of contact frequency

A heatmap and a histogram can be plotted 

<table>
  <tr>
    <td><img src="https://www.pastepic.xyz/images/2018/12/14/Untitled51f762f1893d848d8.png" "heatmap+histograms" width="800")></td>
    <td>##### Figure *: Example of heatmap and histogram visualizations for the Normalized and Observed over Expected contact matrices.</td>
  </tr>
</table>
<!---
![Untitled51f762f1893d848d8.png](https://www.pastepic.xyz/images/2018/12/14/Untitled51f762f1893d848d8.png "heatmap+histograms" width="500")
-->


## TAD Analysis and Visualization<a name="5"></a> 
<table>
  <tr>
    <td>**Goal**: Calculate the coordinates (location) of topologically associated domains (TADs).</td>
    <td align="right"><img src="https://www.pastepic.xyz/images/2018/12/14/Untitled6f6657b45148d4158.png" width="600"/></td>
  </tr>
  <tr>
    <td></td>
    <td>##### Figure***:</td>
  </tr>
</table>

<!---
![Untitled6f6657b45148d4158.png](https://www.pastepic.xyz/images/2018/12/14/Untitled6f6657b45148d4158.png "TADs")
-->

# Reference
[1] Erez Lieberman-Aiden et al. “Comprehensive mapping of long-range interactions reveals
folding principles of the human genome”. In: science 326.5950 (2009), pp. 289–293

[2] Yaffe E, Tanay A. Probabilistic modeling of Hi-C contact maps eliminates systematic biases to characterize global chromosomal architecture. Nat. Genet. 2011;43:1059–1065.

[3] Ay F, Noble WS. Analysis methods for studying the 3D architecture of the genome. Genome Biol 2015;16:183.

[4] Erez Lieberman-Aiden et al. “Comprehensive mapping of long-range interactions reveals folding principles of the human genome”. In: science 326.5950 (2009), pp. 289– 293 

[5] Riccardo Calandrelli's slides: [https://drive.google.com/file/d/1nxP13mOIer_6yiLi7OFXHbQ4-D3Eqq4k/view](https://drive.google.com/file/d/1nxP13mOIer_6yiLi7OFXHbQ4-D3Eqq4k/view)



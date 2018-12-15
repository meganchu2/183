# 3.2.2 GITAR Pipeline 
1. [Overview](#1)
2. [Data Preprocessing](#2)
3. [Data Normalization](#3)
4. [Data Visualization](#4)
5. [TAD Analysis and Visualization](#5)


<a name="1"></a>
## Overview

GITAR (Genome Interaction Tools and Resources) is a software to perform a comprehensive Hi-C data analysis.

> The entire protocol is outlined at [https://doc.genomegitar.org](https://doc.genomegitar.org).  Please visit this link if you are interested in learning GITAR in detail, or if you are interested in analyzing your own Hi-C data with GITAR.

Hi-C data is analyzed and visualized in the following steps:
- Data preprocessing
- Data Normalization
- Data Visualization
- Topologically associated domain (TAD) Analysis and Visualization


<a name="2"></a>
## Data Preprocessing

**Goal**: Improve the genome mapping outcome of Hi-C data.

<table>
  <tr>
    <td><img src="https://www.pastepic.xyz/images/2018/12/14/Untitled04d01bad2783e292.png" "Hi-C fragment"" width="1000"/></td>
    <td><sub>Figure 1. Schematic of two interacting DNA fragments (called DNA fragment 1 and DNA fragment 2) resulting from the Hi-C protocol. [3]</sub></td>
  </tr>
</table>
<table>
  <tr>
    <td><img src="https://www.pastepic.xyz/images/2018/12/14/Untitled3d84a95c29cb5ccd2.png" "ligation site"" width="1000"/></td>
    <td><sub>Figure 2. Schematic of reads generated from the interacting DNA fragments in Figure 1. [5]</sub></td>
  </tr>
</table>
<!---
![Untitled04d01bad2783e292.png](https://www.pastepic.xyz/images/2018/12/14/Untitled04d01bad2783e292.png "Hi-C fragment")
![Untitled3d84a95c29cb5ccd2.png](https://www.pastepic.xyz/images/2018/12/14/Untitled3d84a95c29cb5ccd2.png "ligation site")
 -->

<table>
  <tr>
    <th>Challenges</th>
    <th>Solutions</th>
  </tr>
  <tr>
    <td><b>Problem 1</b>: Hi-C reads have the form depicted by Figure 1; each read pair contains one read for DNA fragment 1 and one read for DNA fragment 2.  However, it is also possible that each read in the read pair contains parts of the ligation site (in theory the ligation site is not a naturally occurring sequence in the genome), and this ligation site sequence would not map to anywhere in the genome.</td>
    <td><b><i>Solution 1 "Pre-truncation"</i></b>: Remove parts of the ligation site sequence contained in the reads (if there are any). [1]</td>
  </tr>
  <tr>
    <td><b>Problem 2</b>: Any two interacting DNA fragments may be many kilobases away from each other, thus a read pair (one read for each interacting fragment) may not map well to the genome as "one sequence".</td>
    <td><b><i>Solution 2 "Independent Alignment"</i></b>: Map/align each read in the read pair to the genome separately.</td>
  </tr>
  <tr>
    <td><b>Problem 3</b>: After mapping the reads, we realize it is possible that some reads are unable to be mapped to the genome, some read pairs map to the same DNA fragment (which doesn't make sense since this implies the DNA fragment interacts with itself), or some reads have a very low quality mapping (doesn't really match with the genome sequence it mapped to).</td>
    <td><b><i>Solution 3 "Filtering Reads"</i></b>: Remove read pairs that fall within the same fragment or have MAPQ score (measurement of how "good" the alignment of the read to the genome is) less than a specified threshold.</td>
  </tr>
</table>


<a name="3"></a>
## Data Normalization
**Goal**: Normalize the data for technical biases while taking into account biologial biases.

*What are technical biases?* Biases in the data due to spurious ligation products, fragment length, GC content, and mappability.

*What are biological biases?* A biological bias is due to genomic features such as transcription start sites (TSSs) and CTCF binding sites.  This bias is evident in Figure 3.  

![Untitled2940c6daf03d9f546.png](https://www.pastepic.xyz/images/2018/12/14/Untitled2940c6daf03d9f546.png)
<sub>Figure 3: Notice that at DNA regions around a distance of 100K (upstream and downstream) from a transcription start site have more contacts with that transcription start site, and DNA regions around a distance of 200K (upstream and downstream) from a CTCF binding site have more contacts with that CTCF binding site. [4]</sub>

The **learned correction parameters** are determined using an explicit-factor model from Yaffe and Tanay, and they will normalize the technical biases without normalizing the biological biases.

Our mapped read pairs can be organized in a format called the contact matrix.

![Untitled4e5d67cfb0178b41c.png](https://www.pastepic.xyz/images/2018/12/14/Untitled4e5d67cfb0178b41c.png "contact matrix")
<sub>Figure 4: The left matrix shows the general structure of a contact matrix where the rows represent loci (bins) on Chromosome i and the columns represent the loci (bins) on Chromosome j.  The entries of the contact matrix contain the number of contacts between each pair of loci.  The contact matrix can be used to generate a heatmap (right matrix), where more contacts between a pair of loci is represented by a more intense color (in this example, white = 0 and red = 5). [5]</sub>

**How to calculate normalized data**
- Observed contact matrix O[i,j]: observed read count between loci identified by bins i and j.
- Correction matrix C[i,j]: sum of corrections for read pairs between bins i and j.
- Normalized contact matrix N[i,j]: corrected/normalized contact counts calculated by 
<p align="center">N[i,j] = O[i,j] / C[i,j]</p>

**Other relevant contact matrices** [3]
- Expected contact matrix E[i,j]: expected read counts between loci identified by bins i and j (calculated by considering correction parameters and the linear distance between read pairs)
- Observed over Expected contact matrix O_over_E[i,j]: calculated by dividing the observed read counts by the expected read counts;
<p align="center">O_over_E[i,j] = O[i,j] / E[i,j]</p>


<a name="4"></a>
## Data Visualization 
**<u>Goal</u>**: Give visual representation of interchromosomal and intrachromosomal interactions.

**Types of Visual Representation**
- Heatmaps show contacts based on data from contact matrices
- Histograms show distribution of contact frequency

Heatmaps and histograms are generated for the observed, expected, normalized, and "observed over expected" contact matrices using matplotlib, so that we can customize the parameters of our plots, such as:
- plotting the full matrix or selecting only a portion
- plotting the full range of the data or using a cut-off for contact counts to enhance local chromatin structures
- colormap
- adding labels and chromosomal coordinates to the heatmaps [5]


<table>
  <tr>
    <td><img src="https://www.pastepic.xyz/images/2018/12/14/Untitled51f762f1893d848d8.png" "heatmap+histograms" width="800")></td>
    <td><sub>Figure 5: Example of heatmap and histogram visualizations for the Normalized and Observed over Expected contact matrices.  Notice that the diagonals in the heatmaps have 0 contacts because it represents interactions of any DNA fragment with itself, and we made sure to remove any reads that suggested this in the data pre-processing step. [5]</sub></td>
  </tr>
</table>
<!---
![Untitled51f762f1893d848d8.png](https://www.pastepic.xyz/images/2018/12/14/Untitled51f762f1893d848d8.png "heatmap+histograms" width="500")
-->


<a name="5"></a>
## TAD Analysis and Visualization 
<table>
  <tr>
    <td>
<b><u>Goal</u></b>: Calculate the coordinates (location) of topologically associated domains (TADs).<br><br>

<i>What are TADs?</i> They are highly self-interacting regions at the level of hundreds of kilobases (~10^5 bases) to a few megabases (~10^6 bases).  They are separated by boundaries that prevent interactions with the neighboring regions. [2]<br>

Looking at Figure 6 we see at the very top that the chromatin is bunched up into red and black regions (where each "blob" is a TAD).  We see that these regions correspond to the ends of the triangles in Figure 6A, so each triangle corresponds to one TAD.  In Figure 6B, we see that the Directionality index changes sign at the ends of each triangle, thus this sign change marks the boundaries of each TAD.<br>

<b>How to calculate TAD coordinates</b><br>
<ol>
<li> Calculate the Directionality Index (DI)</li><br>
  <p><ul>
  <li> quantifies degree of upstream or downstream bias of a given bin</li>
  <li> DI formula: <img src="http://pastepic.xyz/images/2018/12/14/ Untitled70cc6753d2bf21933.png" width="200"/> </li>
  </ul></p>
<li> Use a Hidden Markov Model (HMM) to determine the underlying biased state for each locus (upstream, downstream or none).</li><br>
<li> Determine TAD Coordinates</li><br>
  <p><ul>
  <li> Shifts in true DI between negative and positive determines the TAD boundaries</li>
  <li> The TAD boundaries give us the TAD Coordinates</li>
  </ul></p>
</ol>
</td>
    <td><img src="https://www.pastepic.xyz/images/2018/12/14/Untitled6f6657b45148d4158.png" width="1000"/><sub>Figure 6: Plot A slices a normalized contact matrix/heatmap along its diagonal and uses this diagonal as the new x-axis.  Each triangle boundary on the heatmap lines up with one of the red/black TADs at the very top of the figure.  Plot B gives us the value of the Directionality Index at each loci in the chromosome, where each time the DI crosses the x-axis lines up with a TAD boundary.</sub></td>
  </tr>
</table>

<!---
![Untitled6f6657b45148d4158.png](https://www.pastepic.xyz/images/2018/12/14/Untitled6f6657b45148d4158.png "TADs")
-->

# Reference
[1] Ay F, Noble WS. Analysis methods for studying the 3D architecture of the genome. Genome Biol 2015;16:183.

[2] Dixon, Jesse R., et al. “Topological domains in mammalian genomes identified by analysis of chromatin interactions.” Nature 485.7398 (2012): 376-380 

[3] Erez Lieberman-Aiden et al. “Comprehensive mapping of long-range interactions reveals folding principles of the human genome”. In: science 326.5950 (2009), pp. 289– 293 

[4] Yaffe E, Tanay A. Probabilistic modeling of Hi-C contact maps eliminates systematic biases to characterize global chromosomal architecture. Nat. Genet. 2011;43:1059–1065.

[5] Riccardo Calandrelli's slides: [https://drive.google.com/file/d/1nxP13mOIer_6yiLi7OFXHbQ4-D3Eqq4k/view](https://drive.google.com/file/d/1nxP13mOIer_6yiLi7OFXHbQ4-D3Eqq4k/view)



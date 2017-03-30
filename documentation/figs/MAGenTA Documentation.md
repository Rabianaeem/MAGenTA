# <center> Tn-Seq Analysis on the Command-line </center>

## Overview

MAGenTA is a tool for Transposon Insertion Sequencing (Tn-Seq) data analysis. It was developed to analyze data generated by the method outlined in [van Opijnen 2009](http://www.nature.com/nmeth/journal/v6/n10/abs/nmeth.1377.html), which unlike other transposon insertion sequencing methods (cite here), assesses organismal fitness by profiling the mutant library at two time points, typically before and after a challenge (i.e. an antibiotic) is presented. The MAGenTA pipeline tools differ from other Tn-Seq analysis tools in that they perform comparative analyses that include this fitness measurement, rather than only looking at insertion representation. MAGenTA is available as a command-line based tool and through a Galaxy web-interface. This manual covers the command-line based workflow, beginning with data pre-processing and then novel analysis tools in MAGenTA.
</break>

</break>
## Workflow

Note: The following workflow is the same for running MAGenTA analyses on the command-line or through the Galaxy implementation. 

#### Part 1: Data Prep
A fastq file containing the reads (over insertion sites) from a Tn-Seq experiment are trimmed for quality and divided by training transposon sequence. Next, the trailing transposon sequences are trimmed, resulting reads are concatenated, and then split by barcode. Once reads are divided by barcode, the barcodes are trimmed and the reads are filtered by quality, collapsed, and mapped to the reference genome. This procedure differs for cases where random shearing is performed (see below).

#### Part 2: Fitness Analysis
Once reads are successfully mapped to the reference genome, the proportion of single insertion mutants that were present at two time points is known (by the number of reads mapping to a single location, which represents a transposon insertion site). This data is used to calculate the fitness cost of a single insertion mutation on the organism. Fitness costs of single mutations can be aggregated over genes and non-gene regions as an assessment of the importance of that entire region to the organism's fitness. Here, the fitness cost of a single insertion mutation is simply referred to as the "fitness". In other words the fitness at genome position 33452 of some organism means the fitness cost of a mutation at that position. Similarly, the "fitness" of a gene or region is the fitness of the organism if it were to "lose" that gene or region. A fitness of 1 indicates a neutral effect a mutation at the respective position/region would have on the organism. A fitness below 1 indicates that the organism's fitness would decrease as a result of a mutation at that position/region. Likewise, a fitness above 1 indicates that a mutation in that position/region would increase the organism's fitness.

#### Part 3: Comparative anlayses and other tools
Single insertion, gene, and regional (custom or sliding window) fitness data from part 2 can be further analyzed in this part. The DataOverview tool provides aggregate data information over all libraries inputted as well as genome-wide information. This can be used to understand possible biases in data outputted by downstream tools that use the single insertion input files. Comparative analyses for genes and regions for experiments can be performed using the CompGenes, CompStrains, and CompWindows tools.

![alt text](https://github.com/antmarge/magenta/blob/master/figures/tnseq_workflow.png?raw=true "Workflow")


## <center> Part 1: Data Pre-processing</center>

1. Download and install the [FASTX Toolkit](http://hannonlab.cshl.edu/fastx_toolkit/) and [Bowtie](http://bowtie-bio.sourceforge.net/index.shtml). For more on the usage of the FASTX Toolkit, see its [manual](http://hannonlab.cshl.edu/fastx_toolkit/commandline.html), and for more on Bowtie see its [manual](http://bowtie-bio.sourceforge.net/manual.shtml).
2. Split the data by experimental condition (i.e. barcodes) with fastx barcode splitter and remove all non-genomic DNA (e.g. barcodes) with fastx trimmer. These steps may be done in whatever order best suits your data.
3. Filter reads by quality (e.g. a minimum quality of 8) using fastq quality filter.
4. Collapse reads using fastq collapser.
5. Map reads using Bowtie. We recommend using the following flags: -n 1 (maximum 1 mismatch) --best (guarantee that reported singleton alignments are 'best' in terms of stratum and quality) -y (try hard) and -m 1 (supress all alignments if more than one can be found)

## <center> Part 2: Fitness Analysis </center>

Download the MAGenTA fitness analysis scripts: [CalcFitness](https://github.com/vanOpijnenLab/magenta-p2/blob/master/tools/Calculate%20Fitnesses/calc_fitness.py) and [AggregateFitness](https://github.com/vanOpijnenLab/magenta-p2/blob/master/tools/Aggregate%20Fitnesses/aggregate.py). 

***
### CalcFitness
Calculate the fitness cost of a single insertion mutation for an organism
***

For every pair of t1/t2 reads, calculate the fitnesses of insertion locations using the calc_fitness.py script as follows:

#### USAGE

```perl
python calc_fitness.py -ref <genome file> -t1 <map file from timepoint 1> -t2 <map file from timempoint 2> -out <name of outputfile>	
```
	REQUIRED
	-ref	The name of the reference genome file, in GenBank format.  
	-t1	The name of the bowtie mapfile from time 1.  
	-t2	The name of the bowtie mapfile from time 2.  
	-out	Name of a file to enter the .csv output.  
	
	OPTIONAL
	-expansion	Expansion factor (default: 250)  
	-reads1		The number of reads to be used to calculate the correction factor for time 0.    (default counted from bowtie output)  
	-reads2		The number of reads to be used to calculate the correction factor for time 6.    (default counted from bowtie output)  
	-cutoff1	Discard any positions where the average of counted transcripts at time 0 and time 1 is below this number (default 0)  
	-cutoff2	Discard any positions within the normalization genes where the average of counted transcripts at time 0 and time 1 is below this number (default 10)  
	-strand		Use only the specified strand ( or -) when counting transcripts (default: both)  
	-normalize	A file that contains a list of genes that should have a fitness of 1 - used for normalization and bottleneck calculations.  
	-b		Calculate bottleneck value (the percentage of insertions randomly lost) from all genes (rather than only normalization genes)  
	-maxweight	The maximum weight a transposon gene can have in normalization calculations  
	-multiply	Multiply all fitness scores by a certain value (e.g., the fitness of a knockout). You should normalize the data.  
	-ef		Exclude insertions that occur in the first N amount (%) of gene--becuase may not affect gene function.  
	-el		Exclude insertions in the last N amount (%) of the gene--considering truncation may not affect gene function.  
	-wig		Create a wiggle file for viewing in a genome browser. Provide a filename.  
	-uncol		Use if reads were uncollapsed when mapped.  

***
### <center>Summary of MAGenTA Tools</center>
***


| Tool           | Function                                                                                                                                                    | Research Questions Addressed                                                                                                                       |
|----------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------|
| DataOverview   | Produces a summary file of information about TA sites and insertions in the genome, libraries, and gene/intergenic regions                                  | What is the TA and insertion distribution in the genome for genes and non-gene regions?                                                            
| SlidingWindow  | Scans the genome data with set window size and step to perform aggregate fitness and essentiality assessments at each window.                               | Which unannotated (intergenic or gene domain) regions are important or essential?                                                                  |
| CustomWindow   | Given a list of start and end coordinates, will return aggregate fitness and essentiality test results. Useful for recalculating info for merged regions    | What is the importance or essentiality of specific regions of interest?                                                                            |
| CompGene   | Creates a gene comparison file with significant stats for two Tn-Seq experiments of the same ref genome. Need aggregate output                              | How can we compare genes for a given strain in two different conditions? Which genes are conditionally important?                                  |
| CompStrains | Creates a gene comparison file with significant stats for two Tn-Seq experiments of different strains (genomes). Need conversion file and aggregate output. | How can we compare homologous genes for two strains? Which genes have a strain-dependent importance?                                               |
| CompWindows | Creates a comparison file with significance statistics for two Tn-Seq experiments of different strains (genomes). Need sliding window output                | How can we compare intergenic or gene domain regions for the same strain in two conditions?                                                        |
| Grouper        | Group consecutive sliding window regions that have the same importance/essentiality                                                                         | If consecutive unannotated regions have the same characteristics of importance and/or essentiality, how can we group them to form a single region? |
| GvizTnseq      | Tn-seq visualization functions based on an adpatation of the Gviz R package                                                                                 | How can we visualize multiple Tn-Seq data features for a given region of interest and highlight special regions?                                   |



***
### <center>DataOverview</center>
<center>Characterizing the reference genome and genome-wide insertion data</center>
***

Prior to Tn-seq data analysis it is important to understand the genome-wide scale and context of the data. Large differences in sequencing coverage, insertion representation, and GC content between genomes can affect the robustness of the comparative analyses performed by MAGenTA. Tn-Seq is also generally performed with replicate libraries, where differences (if they exist) would be important to know. The DataOverview tool produces a summary of genome-wide insertion representation and inherent genome characteristics. The input data are queried for TA sites and insertions with respect to

    1. The genome
    2. Open reading frames denoted by start and stop codons
    3. Annotated genes from the Genbank record
    4. Intergenic regions

#### USAGE

```perl
perl dataOverview.pl -d inputs/ -f genome.fasta -r genome.gbk
```
    REQUIRED
    -d  Directory containing all input files (results files from calc fitness script)
        OR In the command line (without a flag), input the file names
    -f  Filename for genome sequence, in fasta format
    -r  Filename for genome annotation, in GenBank format

    OPTIONAL
    -h  Print usage
    -c  Cutoff average(c1+c2)>c. Default: 15
    -o  Filename for output. Default: standard output


#### Output

The DataOverview tool outputs a summary of genome and gene-wide insertion representation for each library, including how many insertions would be filtered out given a cutoff requirement for the number of reads representing each insertion. 

|                                          | Strain1      | Strain2 |
|------------------------------------------|--------------|-----------|
| Genbank Record                           | NC_003028 | NC_0012469 |
| Genome Size (bp)                         | 2160842     | 2112148    |
| GC content                               | 39.70%      | 39.77%     |
| Number of Genes                          | 2390        | 2204       |
| Total number of TA sites                 | 141459      | 137693     |
| Genome coverage by TA sites              | 6.55%       | 6.52%      |
| Largest gap between TA sites (bp)        | 252         | 206        |
| Genome coverage by insertions            | 1.33%       | 3.68%      |
| TA site coverage by insertions           | 32.80%      | 73.12%     |
| TA site coverage by filtered insertions* | 20.28%      | 56.45%     |
| Largest gap between insertions (bp)      | 14996       | 6708       |
| Smallest ORF (bp)                        | 6           | 6          |
| Largest ORF (bp)                         | 220         | 220        |
| Max number TA sites in an ORF            | 10          | 10         |
| Number of ORFs without TA sites          | 6997        | 6701       |
| Average gene length                      | 803         | 834        |
| Minimum gene length                      | 30          | 65         |
| Max gene length                          | 14330       | 6701       |
| Average insertions in a gene             | 9.84        | 28.4       |
| Minimum insertions in a gene             | 0           | 0          |
| Maximum insertions in a gene             | 237         | 420        |
| Genes with insertions                    | 65%         | 77%        |
| Number of genes without insertions       | 564         | 257        |
| Insertions outside of annotated genes    | 15773       | 43266      |
| Insertions in intergenic regions**       | 54.98%      | 55.66%     |


***
### <center>GeneCompare: Comparative gene analysis for the same organism</center>
***

Comparative gene analysis for two experiments of the same organism (i.e. One strain tested in Glucose and in Glucose+Antibiotic) involves merging two output files from the fitness script, then performing a t-test and calculating average fitness difference for each gene. This process can be performed in a spreadsheet applciation, such as Excel. However, GeneCompare can perform it in less than five seconds for 2000 gene.

#### Usage

```perl
perl compGenes.pl <options> <aggregate1.csv> <aggregate2.csv> 
```

    REQUIRED
    -d  Directory containing all input files (results files from calc fitness script)
        OR In the command line (without a flag), input the file names

    OPTIONAL
    -h  Print usage
    -o  Filename for output
    -l  Labels for input files. Default: filenames
        Two strings, comma separated (i.e. -l expt1,expt2).
        Order should match file order.
        

***
### <center>StrainCompare: Comparative gene analysis for different organisms</center>
***

The StrainCompare tool, a variation of GeneCompare, is specifically for gene comparisons across two experiments from <i>different</i> organisms/strains. Since gene identification numbers will be different at the organism/strain level, a file containing the homologous genes is required. The same output as GeneCompare will be produced and can be used in downstream analysis tools. An automated method for extracting homologous genes will be a useful future improvement to this tool.

#### USAGE

```perl
perl compStrains.pl <options> -c conversion.csv [aggregateFile1.csv aggregateFile2.csv OR -d directory/]
```

    REQUIRED
    -d  Directory containing all input files (results files from calc fitness script)
        OR In the command line (without a flag), input the file names
    -c  Tab delimited file with homologous genes

    OPTIONAL
    -h  Print usage
    -o  Filename for output. Default: compFile1File2.csv
    -l  Labels for input files. Default: filenames
        Two strings, comma separated (i.e. -l expt1,expt2).
        Order should match file order.


***
### <center>SlidingWindow: Discovering important unannotated regions</center>
***

The method for calculating the aggregate fitness value of a gene relies on the genomic start and end coordinates of the region, which is retrievable from the organism’s Genbank record. However, not all official genome records are complete or well annotated beyond genes. At the same time, it has been shown that non-coding RNAs have an important role in virulence and other functions. Therefore, to enable discovery of such regions that may have an important fitness value to the organism, this tools uses a  sliding window approach to mine Tn-Seq data in a non-gene-centric manner. 

#### Determining window size and step
The entire genome and ordered Tn-Seq data is scanned by performing assessments of regions at a set size (base pair length of window) and step (base pairs between the start of each window). Based on the genome used, typically a window size between 250 bp and 500 bp is small enough to pick up intergenic regions or gene domains but large enough to contain enough TA sites and insertions. However, this can vary depending on organism and insertion density. The DataOverview tool can help determine these factors.
</break>

#### Method
In each regional profile, the aggregate fitness cost for insertions and the insertion representation in that region is calculated. To assess insertion representation, we follow the method used by Zhang et. al.14. For a given region, with x insertions and n TA sites, 10,000 random sets of n TA sites are selected and the average of insertions over TA sites is calculated for each. The 10,000 resulting means create a null distribution. The p-value can be derived for the original region of interest by ranking it on the distribution for x/n. A statistically underrepresented number of insertions for a region will be significant if the p-value is below the critical value. Gene annotations are also recorded if they are contained in the region. In the future, a function to identify promoter regions based on -10 and -35 box sequences and domain regions will be useful. 

#### Run time
SlidingWindow performs the fitness and insertion representation assessments in approximately 8 minutes for a two-thousand base pair genome with ~70% TA site saturation. For the Tn-Seq data from two strains of Streptococcus pneumoniae grown in two conditions, 211,151 regions (300 bp) across the 2,112,148 bp genome were profiled using this approach and later used for comparative analyses. 

#### Usage

```perl
perl slidingWindow.pl -d inputs/ -o slidWind_output/ -f genome.fasta -r genome.gbk
```

      REQUIRED:
      -d  Directory containing all input files (output files from
          calcFitness tool)
          OR
          In the command line (without a flag), input filename(s)
      -f  Filename for genome sequence, in fasta format
      -r  Filename for genome annotation, in GenBank format
      
      OPTIONAL:
      -h  Print usage
      -size  The size of the sliding window. Default=50
      -step  The window spacing. Default=10
      -x  Exclude values with avg. counts less than x where (c1+c2)/2<x
          Default=15
      -log  Send all output to a log file instead of the terminal
      -max  Expected max number of TA sites in a window.
            Used for creating null distribution library. Default=100
      -o  Specify name of new directory for all output files
          Default=sw_out/
      -w  Do weighted average for fitness per insertion
      -wc  Integer value for weight ceiling. Default=50


    

***
### <center>WindowCompare: Comparative analysis of small, unannotated regions </center>
***

WindowCompare takes results from the SlidingWindow tool and performs a similar comparative analysis as done on genes by GeneCompare and StrainCompare for genes. Since start and end coordinates define windows (or regions), only comparisons between datasets from the <i>same</i> organism (same genome) can be performed. 
</break>

#### USAGE

```perl
perl compWindows.pl <options> [slidingWindows1.csv slidingWindows2.csv OR -d directory]
```

    REQUIRED
    -d  Directory containing all input files (results files from slidingWindow)
        OR In the command line (without a flag), input the file names

    OPTIONAL
    -h  Print usage and exits program
    -l  Labels for input files. Default: filenames
        Two strings, comma separated (i.e. -l expt1,expt2).
        Order should match file order.
    -o  Filename for output. Default: label1label2.csv


#### Filtering comparison results
Beginning with the comparison file for sliding windows (output of WindowCompare), filter based on cutoffs, then the Grouper tool can group consecutive regions that are labeled as important and the CustomWindow tool can recalculate the fitness and insertion representation in these expanded regions for higher quantitative resolution. Regions are classified as intergenic if there are no gene ids recorded for that entry. A frequency distribution of all genes for the selected regions illustrates how many regions there are for each gene and how many are intergenic. 
</break>

#### Example analysis of slidingwindow output

![alt text](https://github.com/antmarge/magenta/blob/master/figures/fig2.png?raw=true "MAGenTA fig2")

In the example shown, over 200,000 300-bp windows for Tn-Seq data from Taiwan-19F grown in glucose and the presence of daptomycin are assessed for conditionally important regions using the adjusted p-value (FDR) and fitness difference between the two conditions. As shown in the yellow highlighted area, 583 regions met the criteria for having adjusted p-values below a critical value of 0.01 and a fitness difference greater than 0.1. Distributions of regions in 20 unique genes and two intergenic regions are shown in the table. 
	


***
### <center>GvizTnSeq: Track visualization of Tn-Seq multi-feature data</center>
***
 	
To visualize a given region’s sequence, annotation, and insertions we use an adapted R package, Gviz, in R. The MAGenTA pipeline adaptation consists of three main functions that draw upon ones already built in the package. However, first file types must be formatted for input as track features in Gviz. For the main data track, multiple library files containing the insertion coordinates and fitness values are aggregated to produce a file containing unique insertions with a single fitness value using the SingleFit tool. For the annotation track, gene coordinates, transcription direction, and id are extracted from the Genbank record using the GetGeneFeatures tool and features specifically formatted for Gviz input.  Finally, the sequence track accepts the genome fasta file. Once these three files are uploaded in Rstudio, the customized Tn-Seq functions can be used to visualize regions by gene id or start and end coordinates with at most two experiments in the same window (Fig.3). 
</break>

#### USAGE


```r

getTracks<-function(fromCoord,toCoord,x1,x2)
```
    
    fromCoord  start genomic coordinate for the highlighted region
    toCoord    end genomic coordinate for the highlighted region
    x1         start coordinate for the viewing window (default: fromCoord)
    x2         end coordinate for the viewing window (default: toCoord)
    

#### Example Output

MAKE THE OUTPUT PICTURE MORE REAL

![alt text](https://github.com/antmarge/magenta/blob/master/figures/fig3A.png?raw=true "MAGenTA fig3A")

![alt text](https://github.com/antmarge/magenta/blob/master/figures/fig3B.png?raw=true "MAGenTA fig3B")



***
### <center>SingleVal: single Fitness or insertion count per insertion as for visualization inptut (IGV or GvisTnSeq)</center>
***

#### USAGE

```bash
perl singleVal.pl -d inputs/ -v count -n NC_XXX0X
```

    REQUIRED
    -d  Directory containing all input files (results files from slidingWindow)
        OR In the command line (without a flag), input the file names

    OPTIONAL
    -h  Print usage and exits program
    -v  String value for output: 'fit' for fitness OR 'count' for count
    print. Default: "fit" for fitness
    -n  Name of the reference genome, to be included in the wig header. Default: genome
    -o  Output file for comparison data. Default: singleVal.wig
    
    

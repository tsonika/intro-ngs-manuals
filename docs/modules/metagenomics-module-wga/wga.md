#Whole Genome Analysis

##Key Learning Outcomes
---------------------

After completing this module the trainee should be able to:

-   Understand the main approaches to perform metagenomics assembly

-   Be able to perform assembly on your data and assess the quality of your assembly

##Resources You’ll be Using
-------------------------

### Tools Used

Meta-Velvet:   
https://github.com/hacchy/MetaVelvet

###Useful Links
------------

Meta-Velvet:   
http://metavelvet.dna.bio.keio.ac.jp/

##Introduction
------------

Performing genomic assembly aims at generating a genome-length sequence
using the sequence information obtained from short reads. In the case of
metagenomics sample, the task is complicated by the number of different
genomes present in the sample and the fact that their sequences are
sometimes very similar to each other. There are two main approaches to
perform de novo assembly (genomic or metagenomic): building a consensus
and generating De Bruijn k-mer graph.

The k-mers represent the nodes of the de Bruijn graph. Nodes are linked
together if they overlap by k-1 nucleotides. Determining the correct
k-mer is important. You can use tools such as Velvet Advisor:
<http://dna.med.monash.edu.au/~torsten/velvet_advisor/>

Building a de Bruijn graph is computationally demanding but navigation
through it to identify path (to generate contigs of continuous
sequences) is quick and memory efficient. Ideally other information,
biological or distance-based, would be used to help build the contigs.

### Assessing the quality of an assembly

For genomic assembly, the accepted criterion of assembly quality is the
number of contigs obtained: the lower this number, the longer the
contigs and therefore the higher genome reconstitution. This number is
often expressed as N50, which is defined as the weighted median such
that 50% of the entire assembly is contained in contigs equal to or
larger than this value. It is calculated by ranking the contigs by
decreasing length and adding their size sequentially until 50% of the
total number of nucleotides is reached: the N50 is defined by the number
of contigs included in this sum. The N50 is also generally used for
metagenomics

##Practical
---------

The purpose of this exercise is to perform an assembly using Meta-Velvet
and illustrate how k-mer choices influence the output quality. The
starting dataset will be a metagenomic dataset. To ensure better
assembly, rather than using the raw reads, we will only assemble the
sequences having passed the EMG QC steps. Meta-Velvet is an extension of
Velvet, a popular genomic assembler. Therefore to perform our assembly,
we will first run Velvet and then Meta-Velvet using the Velvet output as
input. Both programs are run from the command line.

### Investigation of the input sequence file

Open a terminal window (Applications/Accessories/Terminal, a link is
also provided on your desktop) and navigate to the “data” folder in
“AssemblyTutorial” and look at the first lines of the sequence file. The
file is in fasta format and contains sequences of at least 100 nt each

    cd ~/Desktop/AssemblyTutorial/data
    head A7A_processed.fasta
    # What is the total number of sequences?
    grep -c ">" A7A_processed.fasta
    # What is the total number of nucleotides?
    ~/AssemblyTutorial/stats A7A_processed.fasta

The output indicates that the input file contains about 2.2 billion
(2,164,714,530) nucleotides distributed among   21 million (20,975,212)
sequences. The average sequence length was 103.2 nucleotides. The
sequence file also contains 12,942 “N” indicating that some sequences
have ambiguous bases. In addition, the script displays the N50 to N100
values: N50 = 101, n = 10256045 for example means that a cumulated sum
of, at least, half of the total nucleotides is reached after adding the
length of 10,256,045 sequences and that the last sequence added had a
length of 101 nucletotides.

### Performing the assembly using Velvet and Meta-Velvet

Velvet and Meta-Velvet had already been installed on your computer.
However they need to be configured by indicating the k-mer value and the
number of read categories to use. We already have seen what k-mer is
(reminder in page 5 above). The number of read categories is the maximum
number of libraries of different insert lengths. As in our case the
reads are all coming from the same library, we will use the default
value (2). The version of Velvet and MetaVelvet installed on the virtual
machine you will be using as already been configured with k-mer = 59.

We will run the applications from the AssemblyTutorial folder. The
software has been installed in your path so no need to copy/link these
files:

    # first run velveth to generate the k-mers:
    velveth A7A-59 59 -fasta data/A7A_processed.fasta

Then run velvetg to construct the de Bruijn graph. The “- exp\_cov auto”
parameter indicates to the software that the sequence coverage is
considered uniform across the submitted set and that the expected
coverage (i.e. number of reads per sequence) should be the median
coverage value:

    velvetg A7A-59 –exp_cov auto

Finally run meta-velvetg to generate the assembly output in the A7A-59
directory:

    meta-velvetg A7A-59 | tee logfile

### Assessing the quality of the assembly

The main assembly output is a list of contigs provided as a fasta file.
We will know look at these in more details. First we need to navigate to
the output directory:

Finally run meta-velvetg to generate the assembly output in the A7A-59
directory. We can obtain the number of contigs by running the function
grep to only count the lines containing the contig names (identified by
“&gt;”). Then run the stats script, seen earlier, to also obtain the N50
value:

    cd A7A-59
    grep -c ">" meta-velvetg.contigs.fa
    ~/AssemblyTutorial/stats meta-velvetg.contigs.fa

It shows that the sequences had been assembled in 9,182 contigs of an
average length equal to about 1,230 nucleotides. The longest contigs
contains 95,305 nucleotides. The N50 line indicates that half of the
nucleotides are comprised in the first 275 longest contigs and that the
275th contigs is 9,145 nucleotides long. Comparing these stats to the
one obtained before assembly also reveal that the number of nucleotides
involved in the assembly represents only slightly more than 0.5% of the
nucleotides submitted. This reflects, of course, the amount of
overlapping sequences identified by Velvet and MetaVelvet. This also
explains the reduction of the number of ambiguous base (N\_count).

Changing the k-mer value can have a dramatic effect on the quality of
the assembly. Reducing the k-mer to 31 for example yields the following
statistics:

    sum = 15527668, n = 42343, ave = 366.71, largest = 93835
    N50 = 1208, n = 2151
    N60 = 649, n = 3903
    N70 = 328, n = 7388
    N80 = 193, n = 13589
    N90 = 115, n = 24202
    N100 = 61, n = 42343
    N_count = 263

The number of contigs is almost 5 times larger than with k-mer equal 59.
However, increasing the k-mer alone does not systematically lead to
better stats. A k-mer of 63 produces an assembly with higher number of
contigs (Note that the N50 value is also increased):

    sum = 11579884, n = 10062, ave = 1150.85, largest = 80291
    N50 = 7440, n = 365
    N60 = 4524, n = 563
    N70 = 2251, n = 921
    N80 = 827, n = 1795
    N90 = 326, n = 4155
    N100 = 125, n = 10062
    N_count = 1850

Without extra information, it could be challenging, using the N50
parameter, to judge the quality of a metagenomics assembly. We could use
blast or other tools to infer taxonomy to different sections of the
contigs: obtaining similar affiliation for all fragments originating
from the same contigs would be indicative of a good assembly.

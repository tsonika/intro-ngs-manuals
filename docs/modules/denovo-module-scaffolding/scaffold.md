## Key Learning Outcomes

After completing this module the trainee should be able to:

-   Run an assembly using long mate pair (LMP) reads.

-   Compare results of LMP assemblies with those from the short reads.

-   Run scaffolding and assess the assembly statistics.

***
## Resources You’ll be Using

### Tools Used

Kmer Analysis Tool kit:  
https://github.com/TGAC/KAT

Nextclip:  
https://github.com/richardmleggett/nextclip

Abyss:  
http://www.bcgsc.ca/platform/bioinfo/software/abyss

Soap Denovo:  
http://soap.genomics.org.cn/soapdenovo.html

SOAPec:  
http://soap.genomics.org.cn/about.html

***
## Author Information

*Primary Author(s):*  
Bernardo Clavijo, TGAC Bernardo.Clavijo@tgac.ac.uk

*Contributor(s):*  
Sonika Tyagi, AGRF sonika.tyagi@agrf.org.au  
Zhiliang Chan zhiliang@unsw.edu.au


***
## Scaffolding with long mate pair libraries: Chalara scaffolding using LMP

Scaffold is made of two or more contigs joined together using the read
pair information. In the absence of a high-quality reference genome, new
genome assemblies are often evaluated on the basis of the number of
scaffolds and contigs required to represent the genome. Other criteria
such as the alignment of reads back to assemblies, N50, and the length
of contigs and scaffolds relative to the size of the genome can also be
used to assess the quality of assemblies. In this exercise we will be
using a long mate pair data to perform assembly and scaffolding and we
will focus on how using LMP reads can affect the assemblies. Most part
of this tutorial has been precomputed and made available to you in
interest of time. We will spend more time exploring and discussing the
results.

### Pair-end assembly using *Chalara* dataset

Before doing a scaffolding, we need to build an assembly using the
pair-end reads first.

Let's go to the correct location where the files are:

    cd /home/trainee/scaffolding/cha_raw/test_1

<br>

!!! failure "STOP"
    You DO NOT need to run this command. This has already been done for you.

        abyss-pe name=cha1 k=27 in="../LIB2570_raw_R1.fastq ../LIB2570_raw_R2.fastq" np=4

<br>
Let’s look at the stats by doing:

    less cha1-stats.tab

<br>
**Table 1**: Statistics of *Chalara* assembly by ABySS using k=27.

|**n** | **n:500** | **L50** | **min** | **N80**| **N50**| **N20**| **E-size**| **max** | **sum**| **name**|
|----|----|----|----|----|----|----|----|----|----|----|
|394789 | 11681 | 2094 | 500 | 2880 | 6254 | 12303 | 7936 | 44414 | 44.07e6 | cha1-unitigs.fa|
|394199 | 11673 | 2097 | 500 | 2887 | 6255 | 12303 | 7937 | 44414 | 44.11e6 | cha1-contigs.fa|
|394161 | 11647 | 2095 | 500 | 2898 | 6269 | 12303 | 7944 | 44414 | 44.12e6 | cha1-scaffolds.fa|


    cd /home/trainee/scaffolding/cha_raw/test_2

<br>

!!! failure "STOP"
    This is also a pre-computed example for you:

         abyss-pe name=cha2 k=61 in="../LIB2570_raw_R1.fastq ../LIB2570_raw_R2.fastq" np=4

<br>
Again, we need to look at the statistics of our new assembly using k=61:

    less cha2-stats.tab

<br>
It should be looking like this:

**Table 2**: Statistics of *Chalara* assembly by ABySS using k=61.

|**n** | **n:500** | **L50** | **min** | **N80**| **N50**| **N20**| **E-size**| **max** | **sum**| **name**|
|----|----|----|----|----|----|----|----|----|----|----|
|130547 | 12596 | 1770 | 500 | 3352 | 8379 | 16380 | 10518 | 54300 | 50.75e6 | cha2-unitigs.fa|
|130210 | 12575 | 1771 | 500 | 3363 | 8382 | 16380 | 10525 | 54300 | 50.78e6 | cha2-contigs.fa|
|130182 | 12555 | 1769 | 500 | 3377 | 8394 | 16380 | 10534 | 54300 | 50.78e6 | cha2-scaffolds.fa|

<br>

!!! note "Question"
    The assembly contiguity is worse than *Fusarium*, why?

    !!! success ""
        ??? "**Answer**"
            * Genome characteristics.
            * Data
            * Coverage
            * Error distributions
            * Read sizes
            * Fragment sizes
            * Kmer spectra


### Chalara scaffolding using long mate-pair reads (LMP)

Let’s put some LMP in there.

    cd /home/trainee/scaffolding/cha_raw/test_3
<br>

!!! failure "STOP"
    This is also a pre-computed example for you:

        abyss-pe name=chalmp1 k=61 in="../LIB2570_raw_R1.fastq ../LIB2570_raw_R2.fastq" mp="lmp1" lmp1="../LIB8209_raw_R1.fastq ../LIB8209_raw_R2.fastq" np=4

<br>
  ```
  less chalmp1-stats.tab
  ```

<br>
**Table 3**: Statistics of *Chalara* assembly by ABySS using k=27.

|**n** | **n:500** | **L50** | **min** | **N80**| **N50**| **N20**|**E-size**| **max** | **sum**| **name**|
|----|----|----|----|----|----|----|----|----|----|----|
|130548 | 12596 | 1770 | 500 | 3352 | 8379 | 16380 | 10518 | 54300 | 50.75e6 | chalmp1-unitigs.fa|
|130211 | 12575 | 1771 | 500 | 3363 | 8382 | 16380 | 10525 | 54300 | 50.78e6 | chalmp1-contigs.fa|
|130148 | 12545 | 1769 | 500 | 3380 | 8400 | 16380 | 10535 | 54300 | 50.78e6 | chalmp1-scaffolds.fa|


So... what happened? Let’s check the data by:

  - Kmer spectra
  - Fragment sizes

<br>
Any hints on the protocol? And a not-so-obvious property:

    kat plot spectra-mx --intersection -x 30 -y 15000000 -o pe_vs_lmp-main.mx.spectra-mx.png pe_vs_lmp-main.mx

<br>
Do you remember LMP required pre-processing? Let’s try with processed LMP.

Prior task (already made) preprocess the LMP with `nextclip`.

    cd /home/trainee/scaffolding/cha_proc/test_1

<br>

!!! failure "STOP"
    This is also a pre-computed example for you:

        abyss-pe name=chalmpproc1 k=61 in="../../cha_raw/LIB2570_raw_R1.fastq ../../cha_raw/LIB2570_raw_R2.fastq" mp="proclmp1" proclmp1="../LIB8209_preproc_R1.fastq ../LIB8209_preproc_R2.fastq" np=4
<br>

  ```
  less chalmpproc1-stats.tab
  ```

<br>
**Table 4**: Statistics of *Chalara* assembly by ABySS using k=61 with LMP.

|**n** | **n:500** | **L50** | **min** | **N80**| **N50**| **N20**| **E-size**| **max** | **sum**| **name**|
|----|----|----|----|----|----|----|----|----|----|----|
|130548 | 12596 | 1770 | 500 | 3352 | 8379 | 16380 | 10518 | 54300 | 50.75e6 | chalmpproc1-unitigs.fa|
|130211 | 12575 | 1771 | 500 | 3363 | 8382 | 16380 | 10525 | 54300 | 50.78e6 | chalmpproc1-contigs.fa|
|122061 | 6679 | 167 | 500 | 18609 | 87510 | 187171 | 106178 | 397967 | 51.15e6 | chalmpproc1-scaffolds.fa|

<br>
That’s much better!

***
## Chalara: beyond first pass

Do you remember these?

Genome characteristics. Data:

  -   Coverage
  -   Error distributions
  -   Read sizes
  -   Fragment sizes
  -   Kmer spectra


Let’s think a bit and try to improve the assembly.

!!! note "Question"
    If you look at the pre-processed LMP, do you notice anything peculiar?


Testing the inclusion of heavily pre-processed LMP coverage into the DBG

    cd /home/trainee/scaffolding/cha_proc/test_2
<br>

!!! failure "STOP"
    This is also a pre-computed example for you:

        abyss-pe name=chalmp2 k=61 se="../LIB8209_preproc_single.fastq" lib="lmp1" lmp1="../LIB8209_preproc_R1.fastq ../LIB8209_preproc_R2.fastq" np=4 >chaproclmp2.log 2>&1
<br>

  ```
  less chalmp2-stats.tab
  ```

<br>
Let’s look at the stats.  

**Table 5**: Statistics of improved *Chalara* assembly by ABySS using k=61 with LMP.

|**n** | **n:500** | **L50** | **min** | **N80**| **N50**| **N20**| **E-size**| **max** | **sum**| **name**|
|----|----|----|----|----|----|----|----|----|----|----|
|128306 | 10150 | 1182 | 500 | 4954 | 12585 | 24777 | 15693 | 68684 | 51.03e6 | chaproclmp4-unitigs.fa|
|118939 | 5772 | 362 | 500 | 15717 | 41870 | 82033 | 52819 | 252066 | 52.24e6 | chaproclmp4-contigs.fa|
|116139 | 4014 | 141 | 500 | 41145 | 109273 | 224986 | 133450 | 460979 | 52.56e6 | chaproclmp4-scaffolds.fa|

***
## References

1.  Robert Ekblom\* and Jochen B. W. Wolf. "A field guide to whole-genome
    sequencing, assembly and annotation". Evolutionary Applications
    Special Issue: Evolutionary Conservation Volume 7, Issue 9, pages
    1026 – 1042, November 2014

2.  De novo genome assembly: what every biologist should know. Nature
    Methods 9, 333 – 337 (2012) doi:10.1038/nmeth.1935

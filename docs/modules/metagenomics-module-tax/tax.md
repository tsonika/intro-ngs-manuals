#An introduction to taxonomic analysis of amplicon and shotgun data using QIIME

##Key Learning Outcomes
---------------------

After completing this practical the trainee should be able to:

-   Understand the open source software package QIIME for analysis

-   Perform a taxonomic analysis on a 16S rRNA amplicon dataset

-   Conduct 16S taxonomic analysis on shotgun data

##Resources You’ll be Using
-------------------------

### Tools Used

QIIME:  
http://qiime.org/

rRNASelector:   
http://www.ezbiocloud.net/sw/rrnaselector

MEGAN:  
http://ab.inf.uni-tuebingen.de/software/megan5

###Useful Links
------------

FASTQ Encoding:   
http://en.wikipedia.org/wiki/FASTQ_format#Encoding

Sommer data:   
https://www.ebi.ac.uk/metagenomics/projects/ERP013648#samples_id

### Sources of Data

-   Felix Sommer et al. (2016). The Gut Microbiota Modulates Energy
    Metabolism in the Hibernating Brown Bear Ursus arctos

-   Sutcliffe et al. (2013). Draft Genome Sequence of Thermotoga
    maritima A7A Reconstructed from Metagenomic Sequencing Analysis of a
    Hydrocarbon Reservoir in the Bass Strait, Australia. Genome Announc.
    1(5): e00688-13.

-   Li et al. (2013). Draft Genome Sequence of Thermoanaerobacter sp.
    Strain A7A, Reconstructed from a Metagenome Obtained from a
    High-Temperature Hydrocarbon Reservoir in the Bass Strait,
    Australia. Genome Announc. 1(5): e00701-13.

-   Lee et al. (2011). rRNASelector: a computer program for selecting
    ribosomal RNA encoding sequences from metagenomic and
    metatranscriptomic shotgun libraries. J. Microbiol. 49(4):689-691.

###Introduction
------------

In this tutorial we will look at the open source software package QIIME
(pronounced ’chime’).

QIIME stands for Quantitative Insights Into Microbial Ecology. The
package contains many tools that enable users to analyse and compare
microbial communities. QIIME was originally developed to analyse of
Roche 454 amplicon sequencing data. In the latest versions workflows
have been added to analyze data from different sequencing platforms,
such as Illumina, and different types of data, such as shotgun data. In
this course we will use QIIME 1.8, which is the latest version.

After completion of this tutorial, you should be able to perform a
taxonomic analysis on a Illumina pair end MiSeq 16S rRNA amplicon
dataset. In addition you should be able to do 16S taxonomic analysis on
shotgun data using the tool rRNASelector in combination with QIIME

###De novo OTU picking and diversity analysis using Illumina data
--------------------------------------------------------------

We will re-analyze the data from Sommer et al. (2016). The Gut
Microbiota Modulates Energy Metabolism in the Hibernating Brown Bear
Ursus arctos. Sommer et al., 2016, Cell Reports 14, 1655–1661 February
23, 2016. An electronic copy of the paper can be found in your Taxonomy
folder. When you have to wait a few minutes for commands to complete,
use the time to acquaint yourself with the study. It is a good example
of a study that combines the power of next-generation sequencing with
environmental observations/measurements.

The analysis we do follows the pipeline described in the QIIME general
tutorial (http://qiime.org/tutorials/tutorial.html). Feel free to look
at this tutorial for further background information. The dataset used in
the tutorial is a subset of the Sommer data. In some parts of the
analysis steps we have precomputed analysis on the complete data set for
comparison.

### Join directory of PE reads

For this tutorial we will analyse the bear data dataset to complete the
necessary steps in a reasonable amount of time.

First we need to join the Illumina Pair End (PE) reads contained in the
bear read directory.


    cd ~/taxonomy
    multiple_join_paired_ends.py -i bear_reads -o bear_join --read1_indicator _1  --read2_indicator _2

### The Mapping File

We will use the mapping file ’mapping.txt’ to associate read data with
one of 23 samples. Note that this file has to be created for each
analysis, as the information is specific for an experiment. The mapping
file can also contain information on your experimental design. The
format is very strict; columns are separated with a single TAB
character; the header names have to be typed exactly as specified in the
documentation. A good sample description is useful as it is used in the
legends of the figures QIIME generates. We could also specify the
reverse primer and remove it from the reads. Unfortunately, the reverse
primer sequence was not in the paper, and we’ll ignore it though we
could probably deduce it from longer reads: as a 466 bp region of the
16S ribosomal RNA gene flanking the V3 and V4 regions was amplified,
you’ll have a clue where to look for the reverse primer.

Execute the following command and test the mapping file for potential
errors:

    less -S mapping.txt
    validate_mapping_file.py -m mapping.txt -o mapping_output

There shouldn’t be any errors. If there are errors, a corrected mapping
file will be written to the directory mapping\_output

### Assign samples to the reads

This step sets up and quality filters the sample reads by sample
identities

Next we will assign reads to the samples and quality filter at phred
quality threshold &gt;= Q20 to joined reads for analysis


    multiple_split_libraries_fastq.py -i bear_join -o bear_split -m sampleid_by_file --include_input_dir_path  -p parameter_file.txt

When completed please enter the directory ’bear\_split’ and have a look
at the log file. It contains detailed information what was done during
the step we’ve just performed. Note that the number of reads assigned to
the different samples varies considerably. Knowing where the individual
samples were taken may give a clue why this may be! Next have a quick
look at the file seqs.fna. What has changed to the header of the reads?

Look at the split libraries results

You will see the reads are now batched to their sample.

    cd split_library_output
    ls -l
    less split_library_log.txt
    less -S seqs.fna

###Picking Operational Taxonomic Units (OTUs)
------------------------------------------

We will now use a workflow for de novo OTU picking, taxonomy assignment,
phylogenetic tree construction, and OTU table construction QIIME has
several workflows to pick OTUs, we will be using the one described in
the general overview tutorial
(http://qiime.org/tutorials/tutorial.html) It has 7 steps, which are
described in some detail in this tutorial. The described procedure is
run with the command from the Taxonomy directory. This step takes about
12mins to run. Please read through the different steps
(<http://qiime.org/tutorials/tutorial.html>) and try to understand the
procedure. Remember that an OTU is not the same as a species, but a
’bag/cluster’ of highly similar sequences (at least 97% is common for
bacteria/archaea), or a single sequence in case of rare OTUs.

Pick de novo OTUs:

    pick_de_novo_otus.py -i bear_split/seqs.fna -o otus   

Please do spend some time looking at the output of this pipeline. In
particular the file ’seqs\_rep\_set\_tax\_assignments.txt’ in the
’uclust\_assigned\_taxonomy’ directory. By default QIIME uses the
Greengenes 16S reference database to assign taxonomy.

It has the following levels: kingdom, phylum, class, order, family, genus, and species. It will be immediately clear that most reads cannot be classified up to species level. As described in step 6 of the QIIME overview tutorial, the pipeline creates a Newick-formatted phylogenetic
tree (rep\_set.tre) in the otus directory.

You can run the program ’figtree’ from the terminal, a graphic interface will be launched by
typing ’figtree’ then hit the return key.

```bash
figtree
```

View the tree by opening the file ’rep\_set.tre’ in the ’otus’ folder (Desktop->Taxonomy->otus). The tree that is produced is too complex to be of much use. We will look at a different tool, Megan 5, which produces a far more useful tree. In step 7 of the QIIME overview tutorial a file called otu\_table.biom is generated.

It is in biom-format, which is increasingly supported by taxonomic software developers. One of the tools that support the BIOM format is Megan
(http://ab.inf.uni-tuebingen.de/software/megan5/).

Megan is a standalone tool for analyzing both taxonomic and functional content of datasets. It is free for academic use, but you will need to request a licence first. We will use Megan version 5 to display a taxonomic tree using the BIOM output we have just produced.

Note: Sequence errors cangive rise to spurious OTUs, we can filter out OTUs that only contain a single sequence (singletons). QIIME allows you to do this quite easily, or you could also remove abundant taxa if you are more interested in rare taxa.

To remove singletons, run the following commands:

    cd otus
    filter_otus_from_otu_table.py -i otu_table.biom -o otu_table_no_singletons.biom -n 2

This removes OTUs with less than 2 sequences. If you use the –k option instead of the –n option, OTUs with more than the specified number of sequences will be removed.

Megan can be opened from the terminal by typing MEGAN. If you are asked for a licence select the following file /mnt/workshop/data/HT\_MEGAN5\_registration\_for\_academic_use.txt.

From the File menu select Import -> BIOM format.
Find your biom file and import it.

Megan will generate a tree that is far more informative than the one produced with FigTree. You can change the way Megan displays the data by clicking on the various icons and menu items. Please spend some time exploring your data.

The Word Cloud visualization is interesting, too, if you want to find out which samples are similar and which samples stand out.

###View OTU statistics
----------------------

You can generate some statistics, e.g. the number of reads assigned,
distribution among samples. Some of the statistics are useful for
further downstream analysis, e.g. beta-diversity analysis. Run the
following now, again from within the Taxonomy directory, and look at the
results. Write down the minimum value under Counts/sample summary. We
need it for beta-diversity analysis

    cd ../
    biom summarize-table -i otus/otu_table.biom -o otus/otu_table_summary.txt
    less otus/otu_table_summary.txt

###Visualize taxonomic composition
------------------------------------------------------------

We will now group sequences by taxonomic assignment at various levels.
The following command produces a number of charts that can be viewed in
a browser. The command takes about 5 minutes to complete

    summarize_taxa_through_plots.py -i otus/otu_table.biom -o  wf_taxa_summary -m mapping.txt

To view the output, open a web browser from the Applications ->Internet menu. You can use Google chrome, Firefox or Chromium. In Google chrome or Chromium, type CTRL-O, or in Firefox use the File menu to
select Desktop ->;Taxonomy ->; wf\_taxa\_summary ->;
taxa\_summary\_plots and open either area\_charts.html or
bar\_chars.html.
I prefer the bar charts myself. The top chart visualizes taxonomic composition at phylum level for each of the
samples.

The next chart goes down to class level and following charts go another level up again. The charts (particularly the ones more at the
top) are very useful for discovering how the communities in your samples
differ from each other. There is a similar plot in the paper, if you
have time, see how our analysis compares with the one described in the
paper.

###Alpha diversity within samples and rarefaction curves
-----------------------------------------------------

Alpha diversity is the microbial diversity within a sample. QIIME can
calculate a lot of metrics, but for our tutorial, we generate 3 metrics
from the alpha rarefaction workflow: chao1 (estimates species richness);
observed species metric (the count of unique OTUs); phylogenetic
distance. The following workflow generates rarefaction plots to
visualize alpha diversity.

Run the following command from within your taxonomy directory, this
should take a few minutes:

    alpha_rarefaction.py -i otus/otu_table.biom -m mapping.txt -o wf_arare -t otus/rep_set.tre

First we are going to view the rarefaction curves in a web browser by
opening
/home/trainee/Desktop/Taxonomy/sutton/wf\_arare/alpha\_rarefaction\_plots/rarefaction\_plots.html.
To start select as metric ’chao1’ and select as category ’Description’.
It is clear that the microbial diversity in some samples is much higher
than in other samples. Click around in the legend as this will help you
work out which line corresponds with which sample. If you have time you
could try to correlate species richness with environmental data from the
paper and establish whether our analysis confirms the findings of the
authors. Next view the precomputed rarefaction curves which show an
increased sequencing depth.

In general the more reads you have, the more OTUs you will observe. If a
rarefaction curve start to flatten, it means that you have probably
sequenced at sufficient depth, in other words, producing more reads will
not significantly add more OTUs. If on the other hand hasn’t flattened,
you have not sampled enough to capture enough of the microbial diversity
and by extrapolating the curve you may be able to estimate how many more
reads you will need. Consult the QIIME overview tutorial for further
information.

Run the following command from within your taxonomy directory, this
should take a few minutes to generate a heatmap of the level three
taxonomy:

    make_otu_heatmap.py -i taxa_summary/otu_table_L3.biom -o taxa_summary/otu_table_L3_heatmap.pdf -c Treatment -m mapping.txt

###Beta diversity and beta diversity plots
---------------------------------------

Before we have a quick look at taxonomic analysis of shotgun data, we
have a quick look at beta diversity analysis, which is the assessment of
differences between microbial communities. As we have already observed,
our samples contain different numbers of sequences. The first step is to
remove sample heterogeneity by randomly selecting the same number of
reads from every sample. This number corresponds to the ’minimum’ number
recorded when you looked at the OTU statistics.

Now run the following command

    beta_diversity_through_plots.py -i otus/otu_table.biom -m mapping.txt -o bdiv_even -t otus/rep_set.tre -e 23183

Read through the beta diversity compute section of the QIIME overview
tutorial and try to understand this workflow. Tomorrow we will look at
visualization of beta diversity analysis results in more detail.
Unfortunately we cannot view the PCoA plots that we have just generated
using the NeCTAR image as WebGL is not supported. Precomputed plots can
be viewed using the browser on your computer, we will make the link
available.

###Closed reference OTU picking of 16S ribosomal rRNA fragments selected from a shotgun data set
---------------------------------------------------------------------------------------------

In a closed-reference OTU picking process, reads are clustered against a
reference sequence collection and any reads, which do not hit a sequence
in the reference sequence collection, are excluded from downstream
analyses. In QIIME, pick\_closed\_reference\_otus.py is the primary
interface for closed-reference OTU picking in QIIME. If the user
provides taxonomic assignments for sequences in the reference database,
those are assigned to OTUs. We could use this approach to perform
taxonomic analysis on shotgun data. We need to perform the following
steps:

1. Extract those reads from the data set that contain 16S ribosomal RNA
sequence. If there are less than (e.g.) 100 nucleotides of rRNA
sequence, the read should be discarded.
2. Remove non-rRNA sequence (flanking regions) from those reads
3. Run closed-reference OTU picking workflow
4. Visualise the results, e.g. in Megan

###Extraction of 16S rRNA sequence-containing reads with rRNASelector
------------------------------------------------------------------

We will analyze an Illumina paired-end dataset that has been drastically
reduced in size for this tutorial, while preserving the majority of the
16S containing reads. The dataset is from the metagenome described at
<http://www.ncbi.nlm.nih.gov/pmc/articles/PMC3772140/>. There is a pdf
in the working directory for this part of the tutorial. This is a paired
end dataset, and where read pairs overlapped, they were merged into a
single sequence. If read pairs did not overlap, both reads were included
in the analysis. QC was performed using the EBI Metagenomics pipeline.
We will use a tool called rRNASelector, which is freely available
(<http://www.ncbi.nlm.nih.gov/pubmed/21887657>) to select our 16S rRNA
sequence containing reads. The tool invokes hmmsearch and uses trained
hidden Markov models to detect reads with 16S rRNA sequence. The tool
also trims the reads so that only 16S rRNA sequence is present in the
fasta file we will feed into the QIIME workflow.

First, we need to go to our working directory. You will find a file
called A7A-paired.fasta containing the sequence reads. Fire up
rRNASelector from the command line.

    cd ~/Desktop/Taxonomy/A7A/
    rRNASelector

A graphical interface should appear. Note interaction with the interface
may have a few seconds lag. Load the sequence file by clicking on ’File
Choose’ at the top and navigate to the file A7A-paired.fasta. Select the
file and click ’Open’. The tool will automatically fill in file names
for the result files. Change the Number of CPUs to ’2’, select
Prokaryote 16S (to include both bacterial and archaeal 16S sequences)
and specify the location of the hmmsearch file by clicking the second
’File Choose’ button. Type in manually the location
’/usr/bin/hmmsearch’, then click process. The run should take a few
minutes to complete.

If all went well, you can close rRNASelector by clicking on Exit. You
will have 3 new files in your directory, one containing untrimmed 16S
reads, one containing trimmed 16S reads (A7A-paired.prok.16s.trim.fasta;
that’s the one we want) and a file containing reads that do not contain
(sufficient) 16S sequence.

###Closed-reference OTU picking workflow and visualization of results in Megan 5
-----------------------------------------------------------------------------

We are now ready to pick our OTUs. We do that by running the following
command (all on one line and no space after gg\_otus-12-10):

    pick_closed_reference_otus.py -i A7A-paired.prok.16s.trim.fasta -o ./cr_uc -r /mnt/workshop/tools/qiime_software/gg_otus-12_10-release/rep_set/97_otus.fasta -t /mnt/workshop/tools/qiime_software/gg_otus-12_10-release/taxonomy/97_otu_taxonomy.txt

We need to specify the following options. The command will take several
minutes to run. When finished open Megan as described before, import the
otu\_table.biom file and explore the results.

    -i input_file.fasta
    -o output_directory
    -r /path/to/reference_sequences
    -t /path/to/reference_taxonomy

###Bonus
-----

If there is time left you could go back to the Polar bear study. The aim
of this study was to understand interrelationship among microbial
community composition, in hibernating bears and effects on laboratory
mice. With additional information from the paper, could you come up with
some conclusions?

Check the results from the ribosomal database classifier, how do these
differ from the QIIME default GreenGenes (gg\_otus\_97) database
classifier results? What can you conclude?

The QIIME overview tutorial at
(http://qiime.org/tutorials/tutorial.html) has a number of additional
steps that you may find interesting; so feel free to try some of them
out. Note hat we have not installed Cytoscape, so we cannot visualize
OTU networks.

We will end this tutorial with a 15-minute summary of what we have done
and how well our analysis compares with the one in the paper.

Hopefully you will have acquired new skills that allow you to tackle
your own taxonomic analyses. There are many more tutorials on the QIIME
website that can help you pick the best strategy for your project
(http://qiime.org/tutorials/). We picked QIIME for this tutorial as it
is widely used and supported, but there are alternatives that might suit
your need better (e.g. VAMPS at http://vamps.mbl.ed>; mothur at
http://www.mothur.org and others).

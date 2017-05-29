#Analysis Visualisation

##Key Learning Outcomes
---------------------

After completing this module the trainee should be able to:

-   Visualise between sample comparisons, using Emperor PCA

-   Understand the difference between weighted and unweighted analysis

##Resources You’ll be Using
-------------------------

### Tools Used

Emperor:   
http://qiime.org/emperor/tutorial_index.html

###Data sets
---------

Sutton et al. (2013). Impact of Long-Term Diesel Contamination on Soil Microbial Community Structure:   
http://www.ncbi.nlm.nih.gov/pmc/articles/PMC3553749/pdf/zam619.pdf

Fierer et al. (2010). Forensic identification using skin bacterial communities:   
http://www.ncbi.nlm.nih.gov/pmc/articles/PMC2852011/

##Introduction
------------

Good data quality and sample metadata is important for visualising
metagenomics analysis.

For this tutorial we are using Emperor a browser enabled scatter plot
visualisation tool. We will be using the Sutton and Fierer data sets for
viewing the principal components analysis (PCoA) from the QIIME Beta
diversity analysis (16S).

##Beta Diversity computation and plots
------------------------------------

Beta diversity represents between sample comparisons based on their
composition. The output of these comparisons is a square matrix where a
“distance” or dissimilarity is calculated between every pair of
community samples, reflecting the dissimilarity between those samples.
The data distance matrix can be then visualized with analyses such as
PCoA and hierarchical clustering. PCoA is a technique that maps the
samples in the distance matrix to a set of axes that show the maximum
amount of variation explained. The principal coordinates can be plotted
in two or three dimensions to provide an intuitive visualization of the
data structure to look at differences between the samples, and look for
similarities by sample conditions.

The Beta diversity workflow you ran earlier contained a number of steps:

-   Rarifying the OTU table to remove sample heterogeneity. A rarified
    OTU table should be used so that artificial diversity induced due to
    different sampling effort is removed.

-   Calculating the Beta diversity, the unifrac weighted and unweighted
    generated principal coorodinates are created here by default.

The following are the required options and inputs for Beta diversity
analysis:

    1.  Option -i, OTU table (*.biom)
    2.  Option –t, phylogeny tree (*.tre) from pick_de_novo_otus.py
    3.  Option –m, the user-defined mapping file
    4.  Option –o, the output directory
    5.  Option –e, the number of sequences per sample (sequencing depth)

You do not need to run the following command, it may overwrite the
pre-computed analysis. This is the command you ran on the Sutton sub
dataset.

    beta_diversity_through_plots.py -i otus/otu_table.biom -m mapping.txt -o wf_bdiv_even122/ -t otus/rep_set.tre -e 122

### Prepare the environment

The data for this practical can be found in the Taxonomy directory on
your desktop. Go to the pre-computed Beta diversity analysis in the
Taxonomy directory.

Open the Terminal and go to where the data is stored. Investigate the
directories available.

    cd ~/Desktop/Taxonomy/sutton_full_denoised/
    ls –lhtr

Open the following files: wf\_bdiv\_even394/unweighted\_unifrac\_pc.txt
and wf\_bdiv\_even394/weighted\_unifrac\_pc.txt using the ‘less’
command. The option –S with ‘less’ allows you to view files with
unwrapped lines, to escape type q (for quit). To scroll left and right,
use the arrow keys.

    less –S wf_bdiv_even394/unweighted_unifrac_pc.txt
    less –S wf_bdiv_even394/weighted_unifrac_pc.txt

In each file every sample is listed in the first column, and the
subsequent columns contain the value for the sample against the noted
principal coordinate. At the bottom of each Principal Coordinate column,
you will find the eigen value and percent of variation explained by the
coordinate. Note the major difference between weighted and unweighted
analysis is the inclusion of OTU abundance when calculating distances
between communities. You should use weighted if the biological question
you are trying to ask takes OTU abundance of your groups into
consideration. If some samples are forming groups with weighted, then
likely the larger or smaller abundances of several OTUs are the primary
driving force in PCoA space, but when all OTUs are considered at equal
abundance, these differences are lost (unweighted).

Note discrete vs. continuous only has to do with coloring of the points,
and not the calculation of UniFrac distances.

### Emperor

We can now run Emperor to prepare the PCoA plots for visualisation.

    make_emperor.py -i wf_bdiv_even394/unweighted_unifrac_pc.txt -m sutton_mapping_file.txt –b "Source,Condition,Source&&Condition" –o emporer_output_unweighted

    ls –l emporer_output_unweighted

Run the last make\_emporer.py command with the weighted\_unifrac, using
the same command. Make sure you rename the output directory to
“emporer\_output\_weighted“. Look in the Emporer output directories and
note the index.html file outputs.

To view the index.html files created from the ‘make\_emperor.py’ steps
above a browser like Firefox or Chrome can be used.

Important - To view the Emporer plots we need to switch from the VM to
your local machine. Open the Firefox browser and view the pre-computed
Emporer directories available at the following URL.
<http://www.ebi.ac.uk/~sterk/emperor/>

View the unweighted analysis on the Sutton sub data set:

<http://www.ebi.ac.uk/~sterk/emperor/sutton_subset_emporer_output_unweighted/>

Check the ‘Colors’ options to view your samples. What can you determine
from the PCoA plot?

To change the colouring scheme click the ‘Colors’ tab. The available
metadata categories are sorted in alphabetical order. Extra information
on the features can be found at
<http://qiime.org/emperor/description_index.html> Try changing the
scaling, visibility, label etc. to improve the display. Note that
Emperor generates publication quality figures in scalable vector
graphics (SVG) format.

Now look at the weighted unifrac analysis,
<http://www.ebi.ac.uk/~sterk/emperor/sutton_subset_emporer_output_weighted/>

You can compare the sub data analysis against the full-denoised dataset.
<ttp://www.ebi.ac.uk/~sterk/emperor/sutton_full_denoised/>

What are the main differences between the weighted and unweighted plots?

Did you notice any differences between the analysis on the sub data set
and the full denoised?

What are your final conclusions on the Sutton dataset?

What did each of the visualisation methods describe?

Taxonomy summary plots:

Alpha diversity plots:

Beta diversity PCoA plots:

Now that you are familiar with Emperor we will next view the forensic
analysis of the Fierer et. al. data. This experiment matches the
bacteria on surfaces to the skin-associated bacteria of the individual
who touched the surface. This study shows that skin-associated bacteria
can be readily recovered from surfaces (including single computer keys
and computer mice) and that the structure of these communities can be
used to differentiate objects handled by different individuals, even if
those objects have been left untouched for up to 2 weeks at room
temperature.

Please look the information contained in the mapping file and identify
those fields that can be plotted.

    cd ~/Desktop/Taxonomy/fierer/
    less –S fierer_mapping_file.txt

List any fields that can be plotted for this study. What is the
remainder of the information for?

Run Emperor to prepare the PCoA plots for visualisation

    make_emperor.py -i fierer_unweighted_unifrac_pc.txt -m fierer_mapping_file.txt -b \ "HOST_SUBJECT_ID,ENV_FEATURE,HOST_SUBJECT_ID&&ENV_FEATURE” -o emporer_output_unweighted

Go to
http://www.ebi.ac.uk/~sterk/emperor/fierer_emporer_output_unweighted/

What did you observe about the clustering?

What fields have been selected?

The on-line Qiime tutorials are very good and growing in numbers. It is
highly recommended checking out in future the many analysis options.

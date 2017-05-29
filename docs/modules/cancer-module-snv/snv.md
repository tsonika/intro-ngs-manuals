Key Learning Outcomes
---------------------

After completing this practical the trainee should be able to:

-   Prepare raw BAM alignments for variant detection

-   Perform QC measures on BAM files

-   Understand and perform simple variant detection on paired NGS data

-   Add annotation information to raw variant calls

-   Visualise variant calls using IGV

Resources You’ll be Using
-------------------------

### Tools Used

SAMTools:  
http://sourceforge.net/projects/samtools/

IGV:  
http://www.broadinstitute.org/igv/

Genome Analysis Toolkit:  
http://www.broadinstitute.org/gatk/

Picard:  
http://picard.sourceforge.net/

MuTect:  
http://www.broadinstitute.org/cancer/cga/mutect/

Strelka:  
https://sites.google.com/site/strelkasomaticvariantcaller/

VarScan2:  
https://github.com/dkoboldt/varscan/

Variant Effect Predictor:  
http://www.ensembl.org/info/docs/tools/vep

GEMINI:  
http://gemini.readthedocs.org

### Sources of Data

http://sra.dnanexus.com/studies/ERP001071  
http://www.ncbi.nlm.nih.gov/pubmed/22194472

Introduction
------------

This talk is based on Introduction to DNA-Seq processing for cancer data
by Mathieu Bourgey, Ph.D

\*https://bitbucket.org/mugqic/muqpic\_pipelines\*

The goal of this hands-on session is to present the main steps that are
commonly used to process and to analyze cancer sequencing data. We will
focus only on whole genome data and provide command lines that allow
detecting Single Nucleotide Variants (SNV). This workshop will show you
how to launch individual steps of a complete DNA-Seq SNV pipeline using
cancer data.

In the second part of the tutorial we will also be using IGV to
visualise and manually inspect candidate variant calls.

Prepare the Environment
-----------------------

We will use a dataset derived from whole genome sequencing of a
33-yr-old lung adenocarcinoma patient, who is a never-smoker and has no
familial cancer history.

The data consists of whole genome sequencing of liver metastatic lung
cancer (frozen), primary lung cancer (FFPE) and blood tissue of a lung
adenocarcinoma patient (AK55).

The BAM alignment files are contained in the subdirectory called
`alignment` and are located in the following subdirectories:

`normal/normal.sorted.bam` and `normal/normal.sorted.bam.bai`
:   \

`tumor/tumor.sorted.bam` and `tumor/tumor.sorted.bam.bai`
:   \

These files are based on subsetting the whole genomes derived from blood
and liver metastases to the first 10Mb of chromosome 4. This will allow
our analyses to run in a sufficient time during the workshop, but it’s
worth being aware that this is less \< 0.5% of the genome which
highlights the length of time and resourced required to perform cancer
genomics on full genomes!

The initial structure of your folders should look like this:

    -- alignment/             # bam files
      -- normal/              # The blood sample directory containing bam files
      -- tumour/               # The tumour sample directory containing bam files \\
    -- ref/                   # Contains reference genome files      
    -- commands.sh            # cheat sheet

Now we need to set some environment variables to save typing lengthy
file paths over and over. Copy and paste the following commands into
your terminal.

    export APP_ROOT=/home/trainee/snv/Applications

    export IGVTOOLS_PATH=$PATH:$APP_ROOT/igvtools/

    export PICARD_JAR=$APP_ROOT/picard/picard.jar

    export SNPEFF_HOME=$APP_ROOT/snpeff/

    export GATK_JAR=$APP_ROOT/gatk/GenomeAnalysisTK.jar

    export BVATOOLS_JAR=$APP_ROOT/bvatools/bvatools-1.6-full.jar

    export TRIMMOMATIC_JAR=$APP_ROOT/trimmomatic/trimmomatic-0.33.jar

    export STRELKA_HOME=$APP_ROOT/strelka/

    export MUTECT_JAR=$APP_ROOT/mutect/muTect-1.1.4.jar

    export VARSCAN_JAR=$APP_ROOT/varscan/VarScan.v2.4.1.jar

    export REF=/home/trainee/snv/ref

    export SNV_BASE=/home/trainee/snv

    export JAVA7=/usr/lib/jvm/java-7-openjdk-amd64/jre/bin/java

    export IGV=$APP_ROOT/igv/igv.sh

Open the Terminal and go to the base `/`home/trainee/snv working
directory:

    cd $SNV_BASE

All commands entered into the terminal for this tutorial should be from
within the **`/`home/trainee/snv** directory.

Check that the `alignment` directory contains the above-mentioned files
by typing:

    ls alignment

BAM Files
---------

Let’s spend some time to explore bam files.

### Step 1: Exploring BAM files

    samtools view alignment/normal/normal.sorted.bam | head -n4

Here you have examples of alignment results. A full description of the
flags can be found in the SAM specification
<http://samtools.sourceforge.net/SAM1.pdf>

Another useful bit of information in the SAM is the CIGAR string. It’s
the 6th column in the file.

This column explains how the alignment was achieved.

M == base aligns **but doesn’t have to be a match.** A SNP will have an
M even if it disagrees with the reference.\
I == Insertion\
D == Deletion\
S == soft-clips. These are handy to find un removed adapters, viral
insertions, etc.

An in depth explanation of the CIGAR can be found
<http://genome.sph.umich.edu/wiki/SAM>

The exact details of the cigar string can be found in the SAM spec as
well.

We won’t go into too much detail at this point since we want to
concentrate on cancer specific issues now.

Now, you can try using picards explain flag site to understand what is
going on with your reads
<http://broadinstitute.github.io/picard/explain-flags.html>

There are 3 unique flags, what do they mean? The flag is the second
column.

**129:**\
read paired\
second in pair\
\
**113:**\
read paired\
read reverse strand\
mate reverse strand\
first in pair\
\
**161:**\
read paired\
mate reverse strand\
second in pair\

There are lots of possible different flags, let’s look at a few more

    samtools view alignment/normal/normal.sorted.bam | head -n100

Let’s take the last one, which looks properly paired and find it’s mate
pair.\
HINT: Instead of using ’head’ what unix command could we pipe the output
to? HINT2: Once we’ve found both reads to stop the command running we
type CTRL-C

    samtools view alignment/normal/normal.sorted.bam | grep HWI-ST478_0133:4:2205:14675:32513

Using the cigar string, what can we tell about the alignment of the mate
pair?

The mate pair has a less convincing alignment with two insertions and
soft clipping reported.

How might the alignment information from the original read be used by
the aligner?

Even though the alignment of the mate pair is questionable the presence
of it’s properly paired mate helps the aligner in deciding where to put
the less-certain read.

You can use samtools to filter reads as well.

How many reads mapped and unmapped were there?\
HINT: Look at the samtools view help menu by typing samtools view
without any arguments

    samtools view -c -f4 alignment/normal/normal.sorted.bam

77229

    samtools view -c -F4 alignment/normal/normal.sorted.bam

22972373

### Step 2: Pre-processing: Indel Realignment

The first step for this is to realign around indels and snp dense
regions.\
The Genome Analysis toolkit has a tool for this called IndelRealigner.\
It basically runs in 2 steps:\
1. Find the targets\
2. Realign them\

    $JAVA7 -Xmx2G  -jar ${GATK_JAR} \
      -T RealignerTargetCreator \
      -R ${REF}/human_g1k_v37.fasta \
      -o alignment/normal/realign.intervals \
      -I alignment/normal/normal.sorted.bam \
      -I alignment/tumour/tumour.sorted.bam \
      -L ${REF}/human_g1k_v37.intervals

    $JAVA7 -Xmx2G -jar ${GATK_JAR} \
      -T IndelRealigner \
      -R ${REF}/human_g1k_v37.fasta \
      -targetIntervals alignment/normal/realign.intervals \
      --nWayOut .realigned.bam \
      -I alignment/normal/normal.sorted.bam \
      -I alignment/tumour/tumour.sorted.bam \
      -L ${REF}/human_g1k_v37.intervals

      mv normal.sorted.realigned.ba* alignment/normal/
      mv tumour.sorted.realigned.ba* alignment/tumour/

Explanation of parameters

-I
:   BAM file(s)

-T
:   GATK algorithm to run

-R
:   the reference genome used for mapping (b37 from GATK here)

-jar
:   Path to GATK jar file

-L
:   Genomic intervals to operate on

Why did we use both normal and tumor together?

Because if a region needs realignment, maybe one of the sample in the
pair has less reads or was excluded from the target creation.\
This makes sure the normal and tumor are all in-sync for the somatic
calling step.

How many regions did it think needed cleaning ?

    wc -l alignment/normal/realign.intervals -> 27300

Indel Realigner also makes sure the called deletions are left aligned
when there is a microsatellite or homopolymer.

    This
    ATCGAAAA-TCG
    into
    ATCG-AAAATCG

    or
    ATCGATATATATA--TCG
    into
    ATCG--ATATATATATCG

Why it is important ?

This makes it easier for down stream analysis tools

For NGS analysis, the convention is to left align indels.

This is only really needed when calling variants with legacy locus-based
tools such as samtools or GATK UnifiedGenotyper. Otherwise you will have
worse performance and accuracy.

With more sophisticated tools (like GATK HaplotypeCaller) that involve
reconstructing haplotypes (eg through reassembly), the problem of
multiple valid representations is handled internally and does not need
to be corrected explicitly.

### Step 3: Pre-processing: Fixmates

Some read entries don’t have their mate information written properly.\
We use Picard to do this:

    $JAVA7 -Xmx2G -jar ${PICARD_JAR} FixMateInformation \
      VALIDATION_STRINGENCY=SILENT \
      CREATE_INDEX=true \
      SORT_ORDER=coordinate \
      MAX_RECORDS_IN_RAM=500000 \
      INPUT=alignment/normal/normal.sorted.realigned.bam \
      OUTPUT=alignment/normal/normal.matefixed.bam

    $JAVA7 -Xmx2G -jar ${PICARD_JAR} FixMateInformation \
      VALIDATION_STRINGENCY=SILENT \
      CREATE_INDEX=true \
      SORT_ORDER=coordinate \
      MAX_RECORDS_IN_RAM=500000 \
      INPUT=alignment/tumour/tumour.sorted.realigned.bam \
      OUTPUT=alignment/tumour/tumour.matefixed.bam

### Step 4: Pre-processing: Mark Duplicates

What are duplicate reads ?

Different read pairs representing the same initial DNA fragment.

What are they caused by ?

PCR reactions (PCR duplicates)\
Some clusters that are thought of being separate in the flowcell but are
the same (optical duplicates)

What are the ways to detect them ?

Picard and samtools uses the alignment positions:\
- Both 5’ ends of both reads need to have the same positions.\
- Each reads have to be on the same strand as well.\
\
Another method is to use a kmer approach:\
- take a part of both ends of the fragment\
- build a hash table\
- count the similar hits\
\
Brute force, compare all the sequences.

Here we will use picards approach:

    $JAVA7 -Xmx2G -jar ${PICARD_JAR} MarkDuplicates \
      REMOVE_DUPLICATES=false \
      CREATE_MD5_FILE=true \
      VALIDATION_STRINGENCY=SILENT \
      CREATE_INDEX=true \
      INPUT=alignment/normal/normal.matefixed.bam \
      OUTPUT=alignment/normal/normal.sorted.dup.bam \
      METRICS_FILE=alignment/normal/normal.sorted.dup.metrics

    $JAVA7 -Xmx2G -jar ${PICARD_JAR} MarkDuplicates \
      REMOVE_DUPLICATES=false \
      CREATE_MD5_FILE=true \
      VALIDATION_STRINGENCY=SILENT \
      CREATE_INDEX=true \
      INPUT=alignment/tumour/tumour.matefixed.bam \
      OUTPUT=alignment/tumour/tumour.sorted.dup.bam \
      METRICS_FILE=alignment/tumour/tumour.sorted.dup.metrics

We can look in the metrics output to see what happened.

    less alignment/normal/normal.sorted.dup.metrics

What percent of reads are duplicates?

0.046996%

Often, we have multiple libraries and when this occurs separate measures
are calculated for each library. Why is this important to do not combine
everything ?

Each library represents a set of different DNA fragments.

Each library involves different PCR reactions

So PCR duplicates can not occur between fragment of two different
libraries.

But similar fragment could be found between libraries when the coverage
is high.

### Step 4: Pre-processing: Base Quality Recalibration

Why do we need to recalibrate base quality scores ?

The vendors tend to inflate the values of the bases in the reads. The
recalibration tries to lower the scores of some biased motifs for some
technologies.

It runs in 2 steps,\
1- Build covariates based on context and known snp sites\
2- Correct the reads based on these metrics

GATK BaseRecalibrator:

    for i in normal tumour
    do
      $JAVA7 -Xmx2G -jar ${GATK_JAR} \
        -T BaseRecalibrator \
        -nct 2 \
        -R ${REF}/human_g1k_v37.fasta \
        -knownSites ${REF}/dbSnp-138_chr4.vcf \
        -L 4:1-10000000 \
        -o alignment/${i}/${i}.sorted.dup.recalibration_report.grp \
        -I alignment/${i}/${i}.sorted.dup.bam

        $JAVA7 -Xmx2G -jar ${GATK_JAR} \
          -T PrintReads \
          -nct 2 \
          -R ${REF}/human_g1k_v37.fasta \
          -BQSR alignment/${i}/${i}.sorted.dup.recalibration_report.grp \
          -o alignment/${i}/${i}.sorted.dup.recal.bam \
          -I alignment/${i}/${i}.sorted.dup.bam
    done

BAM QC
------

Once your whole bam is generated, it’s always a good thing to check the
data again to see if everything makes sense.

### Step 1: BAM QC: Compute Coverage

If you have data from a capture kit, you should see how well your
targets worked. Both GATK and BVATools have depth of coverage tools.\
Here we’ll use the GATK one


    for i in normal tumour
    do
      $JAVA7  -Xmx2G -jar ${GATK_JAR} \
        -T DepthOfCoverage \
        --omitDepthOutputAtEachBase \
        --summaryCoverageThreshold 10 \
        --summaryCoverageThreshold 25 \
        --summaryCoverageThreshold 50 \
        --summaryCoverageThreshold 100 \
        --start 1 --stop 500 --nBins 499 -dt NONE \
        -R ${REF}/human_g1k_v37.fasta \
        -o alignment/${i}/${i}.sorted.dup.recal.coverage \
        -I alignment/${i}/${i}.sorted.dup.recal.bam \
        -L 4:1-10000000
    done

Explanation of parameters

omitBaseOutput
:   Do not output depth of coverage at each base

summaryCoverageThreshol
:   Coverage threshold (in percent) for summarizing statistics

dt
:   down sampling

L
:   Genomic intervals to operate on

Coverage is expected to be  25x in these project Look at the coverage:

    less -S alignment/normal/normal.sorted.dup.recal.coverage.sample_interval_summary
    less -S alignment/tumour/tumour.sorted.dup.recal.coverage.sample_interval_summary

Is the coverage fit with the expectation ?

Yes the mean coverage of the region is 25x:

summaryCoverageThreshold is a useful function to see if your coverage is
uniform.

Another way is to compare the mean to the median. If both are quite
different that means something is wrong in your coverage.

### Step 2: BAM QC: Insert Size

It corresponds to the size of DNA fragments sequenced.

Different from the gap size (= distance between reads) !

These metrics are computed using Picard:

    for i in normal tumour
    do
      $JAVA7 -Xmx2G -jar ${PICARD_JAR} CollectInsertSizeMetrics \
        VALIDATION_STRINGENCY=SILENT \
        REFERENCE_SEQUENCE=${REF}/human_g1k_v37.fasta \
        INPUT=alignment/${i}/${i}.sorted.dup.recal.bam \
        OUTPUT=alignment/${i}/${i}.sorted.dup.recal.metric.insertSize.tsv \
        HISTOGRAM_FILE=alignment/${i}/${i}.sorted.dup.recal.metric.insertSize.histo.pdf \
        METRIC_ACCUMULATION_LEVEL=LIBRARY
    done

look at the output

    less -S alignment/normal/normal.sorted.dup.recal.metric.insertSize.tsv
    less -S alignment/tumour/tumour.sorted.dup.recal.metric.insertSize.tsv

How do the two libraries compares?

The tumour sample has a larger median insert size than the normal sample
(405 vs 329)

### Step 3: BAM QC: Alignment metrics

It tells you if your sample and you reference fit together

For the alignment metrics, samtools flagstat is very fast but with
bwa-mem since some reads get broken into pieces, the numbers are a bit
confusing.

We prefer the Picard way of computing metrics:

    for i in normal tumour
    do
      $JAVA7 -Xmx2G -jar ${PICARD_JAR} CollectAlignmentSummaryMetrics \
        VALIDATION_STRINGENCY=SILENT \
        REFERENCE_SEQUENCE=${REF}/human_g1k_v37.fasta \
        INPUT=alignment/${i}/${i}.sorted.dup.recal.bam \
        OUTPUT=alignment/${i}/${i}.sorted.dup.recal.metric.alignment.tsv \
        METRIC_ACCUMULATION_LEVEL=LIBRARY
    done

explore the results

    less -S alignment/normal/normal.sorted.dup.recal.metric.alignment.tsv
    less -S alignment/tumour/tumour.sorted.dup.recal.metric.alignment.tsv

Do you think the sample and the reference genome fit together ?

Yes, 99% of the reads have been aligned\
Usually, we consider:\
- A good alignment if \> 85%\
- Reference assembly issues if [60-85]%\
- Probably a mismatch between sample and ref if \< 60 %

Variant Calling
---------------

Most of SNV caller use either a Baysian, a threshold or a t-test
approach to do the calling

Here we will try 3 variant callers.\
- Varscan 2\
- MuTecT\
- Strelka

Other candidates\
- Virmid\
- Somatic sniper

many, MANY others can be found here: <https://www.biostars.org/p/19104/>

In our case, let’s create a new work directory to start with (from base
directory):

    cd $SNV_BASE
    mkdir variant_calling

varscan 2 VarScan calls somatic variants (SNPs and indels) using a
heuristic method and a statistical test based on the number of aligned
reads supporting each allele.

Varscan somatic caller expects both a normal and a tumour file in
SAMtools pileup format from sequence alignments in binary alignment/map
(BAM) format. To build a pileup file, you will need:

- A SAM/BAM file ("myData.bam") that has been sorted using the sort
command of SAMtools. - The reference sequence ("reference.fasta") to
which reads were aligned, in FASTA format. - The SAMtools software
package.

    for i in normal tumour
    do
    samtools mpileup -L 1000 -B -q 1 \
      -f ${REF}/human_g1k_v37.fasta \
      -r 4:1-10000000 \
      alignment/${i}/${i}.sorted.dup.recal.bam \
      > variant_calling/${i}.mpileup
    done

    $JAVA7 -Xmx2G -jar ${VARSCAN_JAR} \
    somatic variant_calling/normal.mpileup \
    variant_calling/tumour.mpileup \
    variant_calling/varscan \
    --output-vcf 1 \
    --strand-filter 1 \
    --somatic-p-value 0.001

Notes on samtools arguments

-L
:   = max per-sample depth for INDEL calling [1000] ;

-B
:   = disable BAQ (per-Base Alignment Quality) ;

-q
:   = skip alignments with mapQ smaller than 1 ;

-g
:   = generate genotype likelihoods in BCF format

Notes on bcftools arguments

-v
:   = output potential variant sites only

-c
:   = SNP calling (force –e : likelihood based analyses)

-g
:   = call genotypes at variant sites

Now let’s try a different variant caller, MuTect\

Note MuTecT only works with Java 6, 7 will give you an error\
if you get "Comparison method violates its general contract!\
you used java 7"

    java -Xmx2G -jar ${MUTECT_JAR} \
      -T MuTect \
      -R ${REF}/human_g1k_v37.fasta \
      -dt NONE -baq OFF --validation_strictness LENIENT -nt 2 \
      --dbsnp ${REF}/dbSnp-138_chr4.vcf \
      --input_file:normal alignment/normal/normal.sorted.dup.recal.bam \
      --input_file:tumor alignment/tumour/tumour.sorted.dup.recal.bam \
      --out variant_calling/mutect.call_stats.txt \
      --coverage_file variant_calling/mutect.wig.txt \
      -pow variant_calling/mutect.power \
      -vcf variant_calling/mutect.vcf \
      -L 4:1-10000000

And finally let’s try Illumina’s Strelka

    cp ${STRELKA_HOME}/etc/strelka_config_bwa_default.ini .

    sed 's/isSkipDepthFilters =.*/isSkipDepthFilters = 1/g' -i strelka_config_bwa_default.ini

    ${STRELKA_HOME}/bin/configureStrelkaWorkflow.pl \
      --normal=alignment/normal/normal.sorted.dup.recal.bam \
      --tumor=alignment/tumour/tumour.sorted.dup.recal.bam \
      --ref=${REF}/human_g1k_v37.fasta \
      --config=${SNV_BASE}/strelka_config_bwa_default.ini \
      --output-dir=variant_calling/strelka/

      cd variant_calling/strelka/
      make -j2
      cd ../..

      cp variant_calling/strelka/results/passed.somatic.snvs.vcf variant_calling/strelka.vcf

Now we have variants from all three methods. Let’s compress and index
the vcfs for future visualisation.

    for i in variant_calling/*.vcf;do bgzip -c $i > $i.gz ; tabix -p vcf $i.gz;done

Let’s look at a compressed vcf.

    zless -S variant_calling/varscan.snp.vcf.gz

Details on the spec can be found here:
<http://vcftools.sourceforge.net/specs.html>

Fields vary from caller to caller.

Some values are are almost always there:\
- The ref vs alt alleles,\
- variant quality (QUAL column)\
- The per-sample genotype (GT) values.

Note on vcf fields

DP
:   = "Raw read depth"

GT
:   = "Genotype"

PL
:   = "List of Phred-scaled genotype likelihoods" (min is better)

DP
:   = "\# high-quality bases"

SP
:   = "Phred-scaled strand bias P-value"

GQ
:   = "Genotype Quality"

Looking at the three vcf files, how can we detect only somatic variants?

some commands to find somatic variant in the vcf file

varscan

    grep SOMATIC variant_calling/varscan.snp.vcf

MuTecT

    grep -v REJECT variant_calling/mutect.vcf | grep -v "^#"

Strelka

    grep -v "^#" variant_calling/strelka.vcf

Variant Visualisation
---------------------

The Integrative Genomics Viewer (IGV) is an efficient visualization tool
for interactive exploration of large genome datasets.

Before jumping into IGV, we’ll generate a track IGV can use to plot
coverage:

    for i in normal tumour
    do
      $JAVA7 -jar ${IGVTOOLS_PATH}/igvtools.jar count \
        -f min,max,mean \
        alignment/${i}/${i}.sorted.dup.recal.bam \
        alignment/${i}/${i}.sorted.dup.recal.bam.tdf \
        b37
    done

Open IGV

    $IGV

Then: 1. Chose the reference genome corresponding to those use for
alignment (b37)\
2. Load bam files (tumour.sorted.dup.recal.bam and
normal.sorted.dup.recal.bam)\
3. Load vcf files (from variant\_Calling directory)

Explore/play with the data:\
-find germline variants\
-find somatic variants\
-Look around...

Variant Annotation
------------------

Following variant calling, we end up with a VCF file of genomic
coordinates with the genotype(s) and quality information for each
variant. By itself, this information is not much use to us unless there
is a specific genomic location we are interested in. Generally, we next
want to annotate these variants to determine whether they impact any
genes and if so what is their level of impact (e.g. are they causing a
premature stop codon gain or are they likely less harmful missense
mutations).

The sections above have dealt with calling somatic variants from the
first 10Mb of chromosome 4. This is important in finding variants that
are unique to the tumour sample(s) and may have driven both tumour
growth and/or metastasis. An important secondary question is whether the
germline genome of the patient contains any variants that may have
contributed to the development of the initial tumour through
predisposing the patient to cancer. These variants *may not* be captured
by somatic variant analysis as their allele frequency may not change in
the tumour genome compared with the normal.

For this section, we will use **all** variants from the first 60Mb of
chromosome 5 that have been pre-generated using the GATK HaplotypeCaller
variant caller on both the normal and tumour genomes. The output of this
was GVCF files which were fed into GATK GenotypeGVCFs to produce a
merged VCF file. We will use this pre-generated file as we are primarily
interested in the annotation of variants rather than their generation.
The annotation method we will use is called **Variant Effect Predictor**
or VEP for short and is available from Ensembl here:
<http://ensembl.org/info/docs/tools/vep/index.html>.

Our pre-generated VCF file is located in the `variants` folder. Let’s
have a quick look at the variants:

    zless variants/HC.chr5.60Mb.vcf.gz

Notice how there are two genotype blocks at the end of each line for the
normal (Blood) and tumour (liverMets) samples.

Let’s now run VEP on this VCF file to annotate each variant with its
impact(s) on the genome.

    perl Applications/ensembl-tools/scripts/variant_effect_predictor/variant_effect_predictor.pl -i variants/HC.chr5.60Mb.vcf.gz --vcf -o variants/HC.chr5.60Mb.vep.vcf --stats_file variants/HC.chr5.60Mb.vep.html --format vcf --offline -fork 4 --fasta ref/human_g1k_v37.fasta --fields Consequence,Codons,Amino_acids,Gene,SYMBOL,Feature,EXON,PolyPhen,SIFT,Protein_position,BIOTYPE

VEP will take approximately 10 minutes to run and once it is finished
you will have a new VCF file with all of the information in the input
file but with added annotations in the INFO block. VEP also produces an
HTML report summarising the distribution and impact of variants
identified.

Once VEP is done running, let’s first look at the HTML report it
produced with the following command:

    firefox variants/HC.chr5.60Mb.vep.html

This report shows information on the VEP run, the number of variants,
the classes of variants detected, the variant consequences and the
distributions of variants through the genome. Close Firefox to resume
the terminal prompt.

Now let’s look at the variant annotations that VEP has added to the VCF
file by focussing on a single variant. Let’s fetch the same variant from
the original VCF file and the annotated VCF file to see what has been
changed.

    zcat variants/HC.chr5.60Mb.vcf.gz | grep '5\s174106\s'
    grep '5\s174106\s' variants/HC.chr5.60Mb.vep.vcf

These commands give us the original variant:

    5   174106  .   G   A   225.44  .   AC=2;AF=0.500;AN=4;BaseQRankSum=1.22;ClippingRankSum=0.811;DP=21;FS=0.000;GQ_MEAN=127.00;GQ_STDDEV=62.23;MLEAC=2;MLEAF=0.500;MQ=60.00;MQ0=0;MQRankSum=0.322;NCC=0;QD=10.74;ReadPosRankSum=0.377;SOR=0.446   GT:AD:DP:GQ:PL  0/1:7,6:13:99:171,0,208 0/1:5,3:8:83:83,0,145

and the same variant annotated is:

    5   174106  .   G   A   225.44  .   AC=2;AF=0.500;AN=4;BaseQRankSum=1.22;ClippingRankSum=0.811;DP=21;FS=0.000;GQ_MEAN=127.00;GQ_STDDEV=62.23;MLEAC=2;MLEAF=0.500;MQ=60.00;MQ0=0;MQRankSum=0.322;NCC=0;QD=10.74;ReadPosRankSum=0.377;SOR=0.446;CSQ=missense_variant|cGg/cAg|R/Q|ENSG00000153404|PLEKHG4B|ENST00000283426|16/18|||1076|protein_coding,non_coding_transcript_exon_variant&non_coding_transcript_variant|||ENSG00000153404|PLEKHG4B|ENST00000504041|5/8||||retained_intron  GT:AD:DP:GQ:PL  0/1:7,6:13:99:171,0,208 0/1:5,3:8:83:83,0,145

You can see that VEP has added:

    CSQ=missense_variant|cGg/cAg|R/Q|ENSG00000153404|PLEKHG4B|ENST00000283426|16/18|||1076|protein_coding,non_coding_transcript_exon_variant&non_coding_transcript_variant|||ENSG00000153404|PLEKHG4B|ENST00000504041|5/8||||retained_intron

This is further composed of two annotations for this variant:

    missense_variant|cGg/cAg|R/Q|ENSG00000153404|PLEKHG4B|ENST00000283426|16/18|||1076|protein_coding

and

    non_coding_transcript_exon_variant&non_coding_transcript_variant|||ENSG00000153404|PLEKHG4B|ENST00000504041|5/8||||retained_intron

The first of these is saying that this variant is a missense variant in
the gene PLEKHG4B for the transcript ENST00000283426 and the second that
it is also a non\_coding\_transcript\_exon\_variant in the transcript
ENST00000504041.

Variant Filtration
------------------

We now have a VCF file where each variant has been annotated with one or
more impacts for one or more genes. In a typical whole cancer genome,
you will have about 4-5 million variants, and therefore rows, in a VCF
file which takes up gigabytes of space. In our small example, we have
just 100,000 variants which is already too large to make any kind of
meaningful sense out of by just opening up the VCF file in a text
editor. We need a solution that allows us to perform intelligent queries
on our variants to search this mass of noise for the signal we are
interested in.

Luckily, such a free tool exists and is called GEMINI. GEMINI takes as
an input your annotated VCF file and creates a database file which it
can then query using Structured Query Language (SQL) commands. Not only
does GEMINI make your variants easily searchable, it also brings in many
external annotations to add more information about your variants (such
as their frequencies in international databases).

To get started with GEMINI, let’s make a database out of our annotated
VCF file.

    gemini load -v variants/HC.chr5.60Mb.vep.vcf --cores 4 --skip-gerp-bp --skip-cadd -t VEP variants/HC.chr5.60Mb.vep.vcf.db

This will take approximately 10 minutes. You will see a few errors due
to multiallelic sites, normally these sites are decomposed and
normalized before creating the GEMINI database but this is outside the
scope of this workshop.

Once the database has been created let’s run a basic query to see what
kind of information we get out of GEMINI.

    gemini query -q "SELECT *, (gts).(*), (gt_types).(*), (gt_depths).(*), (gt_ref_depths).(*), (gt_alt_depths).(*), (gt_quals).(*) FROM variants LIMIT 10;" --header variants/HC.chr5.60Mb.vep.vcf.db

This will output a bunch of ordered information for your query to the
command line, this is usually saved to a TSV file and opened in a
spreadsheet as we will do for the next query. In the mean time, let’s
dissect this query to understand the syntax we need to use to filter our
variants. First, we have a SELECT statement which simply specifies that
we want to select data from the database. The following comma-separated
values are the columns that we want to output from the database, in this
case we are selecting all columns with the star character and then all
sub-columns for each sample with the other values. Then, we have a "FROM
variants" statement which is specifying the table within the database
that we want to fetch information from. Finally, the "LIMIT 10"
statement specifies that no more than 10 rows should be returned. In
summary then, we are asking for all columns for 10 rows from the table
`variants`. If you haven’t used SQL before don’t worry, the GEMINI
website is very helpful and provides many examples for how to query your
database.

Let’s now perform a more interesting query to find variants that have a
medium or high impact on a gene and are rare or not present in existing
international allele frequency databases. We will save the output of
this query to a file and open it up in a spreadsheet.

    gemini query -q "SELECT *, (gts).(*), (gt_types).(*), (gt_depths).(*), (gt_ref_depths).(*), (gt_alt_depths).(*), (gt_quals).(*) FROM variants WHERE (impact_severity = 'HIGH' OR impact_severity = 'MED') AND (aaf_1kg_all < 0.01 OR aaf_1kg_all is null) AND (aaf_esp_all < 0.01 OR aaf_esp_all is null) AND (aaf_exac_all < 0.01 OR aaf_exac_all is null);" --header variants/HC.chr5.60Mb.vep.vcf.db > variants/gemini-result.tsv

Notice that we have added a WHERE statement which restricts the rows
that are returned based on values that we specify for specific columns.
Here, we are asking to return variants where their impact on the gene
(impact\_severity column) is medium or high and the allele frequency in
1000Genomes, ESP and EXaC is less than 1% or the variant is not present
in any of these databases.

Now let’s open the result in a spreadsheet to look at the annotations:

    libreoffice --calc variants/gemini-result.tsv
    Tick the "Tab" under "Separated by" on the dialog window that comes up.

You can see that the first 14 columns contain information on the variant
including its location, ref, alt, dbSNP ID, quality and type. Slowly
scroll to the right and look at the columns of data that are provided.
Most importantly, column BD includes the gene this variant impacts, BN
the impact itself and BP the impact severity. Scroll towards the end of
the spreadsheet until you get to columns ED and EE, these contain the
genotype for each of the samples. Columns EH and EI contain the total
depth for each variant in each sample and the 4 following columns
contain the reference and alternate depths for each sample. Finally,
columns EN and EO contain the genotype qualities (from the GQ field in
the VCF) for each sample. As you scroll back and forth through this
spreadsheet, you will see that GEMINI brings in information from a
variety of sources including: OMIM, ClinVar, GERP, PolyPhen 2, SIFT,
ESP, 1000 Genomes, ExAC, ENCODE, CADD and more! We are only looking at a
small number of variants from the start of a chromosome so not many of
these annotations will be present but in a full genome database they are
incredibly useful.

GEMINI allows you to filter your variants based on any column that you
see in this results file. For example, you may want all variants in a
specific gene, in which case you would simply add "WHERE gene = ’BRCA1’"
to your query. For complete documentation with many examples of queries,
see the GEMINI documentation here: http://gemini.readthedocs.org.

References
----------

1.  Paila U, Chapman BA, Kirchner R and Quinlan AR. "GEMINI: Integrative
    Exploration of Genetic Variation and Genome Annotations". PLoS
    Comput Biol, 2013, 9(7): e1003153. doi:10.1371/journal.pcbi.1003153

2.  McLaren W, Pritchard B, Rios D, Chen Y, Flicek P and Cunningham F.
    "Deriving the consequences of genomic variants with the Ensembl API
    and SNP Effect Predictor". Bioinformatics, 2010, 26(16):2069-70,
    doi:10.1093/bioinformatics/btq330

Acknowledgements
----------------

This tutorial is an adaptation of the one created by Louis letourneau
https://github.com/lletourn/Workshops/tree/ebiCancerWorkshop201407doc/01-SNVCalling.md.
I would like to thank and acknowledge Louis for this help and for
sharing his material. The format of the tutorial has been inspired from
Mar Gonzalez Porta. I also want to acknowledge Joel Fillon, Louis
Letrouneau (again), Francois Lefebvre, Maxime Caron and Guillaume
Bourque for the help in building these pipelines and working with all
the various datasets.

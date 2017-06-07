This module was written by Mathieu Bourgey and the original on-line version is available
[here.](https://github.com/mbourgey/EBI_cancer_workshop_visualization/tree/australia_2015)

***
##Key Learning Outcomes

After completing this practical the trainee should be able to:

-   Generate Circos like graphics using R

***
## Resources You’ll be Using

### Tools Used

R:  
https://cran.r-project.org/

R package circlize:  
https://cran.r-project.org/web/packages/circlize/index.html

***
##Introduction

This short workshop will show you how to visualize your data.

We will be working on 3 types of somatic calls:

-   SNV calls from MuTect (vcf)

-   SV calls from DELLY (vcf)

-   CNV calls from SCoNEs (tsv)

***
##Prepare the Environment

We will use a dataset derived from the analysis of whole genome sequencing
paired normal/tumour samples.

The call files are contained in the folder `visualization`:

* `mutect.somatic.vcf`  
* `delly.somatic.vcf`    
* `scones.somatic.tsv`   


Many tools are available to do this and the most commonly known is Circos.
Circos is a really not user-friendly. In this tutorial, we show you an
easy alternative to build a circular representation of genomic data.

First we need to go in the folder `visualization` to do the analysis:

    cd /home/trainee/visualization/

Let's see what is in this folder:

    ls

`circos.R  delly.somatic.vcf  mutect.somatic.vcf  scones.somatic.tsv`

Take a look at the data files.

This data has *not* been restricted to a short piece of a chromosome and SNVs
have already been filtered:

  ```
  less mutect.somatic.tsv
  less delly.somatic.vcf
  less data/scones.somatic.30k.tsv
  ```
<br>

!!! note "Question"
    What can you see from this data?

    !!! success ""
        ??? "**Answer**"
            - the filtered output of 3 different software: Mutect (SNVs),
            Delly (SVs), SCoNEs (CNVs).

            - The 3 files show 2 different formats (vcf, tsv).

            - Almost all type of variants are represented here: mutations,
            deletion, inversion, translocation, large amplification and deletion
            (CNVs).

!!! note "Question"
    Why don’t we use the vcf format for all types of calls?

    !!! success ""
        ??? "**Answer**"
            The 1000 Genomes project tries to use/include SV calls in the vcf format.
            Some tools like Delly use this format for SVs. It is a good idea to try to
            include everything together but this not the be the best way to handle SVs and
            CNVs.

!!! note "Question"
    Why?

    !!! success ""
        ??? "**Answer**"
            Due to the nature of these calls, you can not easily integrate the
            positional information of the two breakpoints (that could be located
            far away or in an other chromosome) using a single position format.
<br>

The analysis will be done using the R program:

    R

We will use the circlize package from the cran R project. This package generates
circular plots and has the advantage of being able to provide
pre-built functions for genomic data. One of the main advantages of this
tool is the use of bed format as input data.

    library(circlize)

<br>
Let’s import the variants:

  ```R
  snp=read.table("mutect.somatic.vcf")
  sv=read.table("somatic.sv.vcf")
  cnv=read.table("data/scones.somatic.tsv",header=T)
  ```

<br>
We need to set up the generic graphical parameters:

  ```R
  par(mar = c(1, 1, 1, 1))
  circos.par("start.degree" = 90)
  circos.par("track.height" = 0.05)
  circos.par("canvas.xlim" = c(-1.3, 1.3), "canvas.ylim" = c(-1.3, 1.3))
  ```

<br>
Let’s draw hg19 reference ideograms:

  ```R
  circos.initializeWithIdeogram(species = "hg19")
  ```

<br>
Unfortunately circlize does not support hg38 yet. So we will need to
reformat our data to fit the hg19 standards. As we are working only on autosomes,
we won’t need to lift-over and we could simply add *chr* at the beginning
of the chromosome names.

We can now draw 1 track for somatic mutations:

  ```R
  snv_tmp=read.table("data/mutec.somatic.vcf",comment.char="#")
  snv=cbind(paste("chr",as.character(snp[,1]),sep=""),snp[2],snp[,2]+1)
  circos.genomicTrackPlotRegion(snv,stack=TRUE, panel.fun = function(region, value, ...) {
      circos.genomicPoints(region, value, cex = 0.05, pch = 9,col='orange' , ...)
  })
  ```

<br>
Let’s draw the 2 tracks for CNVs. One track for duplications in red and
one blue track for deletions.

  ```R
  dup=cnv[cnv[,5]>2,]
  dup[,1]=paste("chr",as.character(dup[,1]),sep="")
  del=cnv[cnv[,5]<2,]
  del[,1]=paste("chr",as.character(del[,1]),sep="")
  circos.genomicTrackPlotRegion(dup, stack = TRUE,panel.fun = function(region, value, ...) {
          circos.genomicRect(region, value, col = "red",bg.border = NA, cex=1 , ...)
  })
  circos.genomicTrackPlotRegion(del, stack = TRUE,panel.fun = function(region, value, ...) {
          circos.genomicRect(region, value, col = "blue",bg.border = NA, cex=1 , ...)
  })
  ```

We can clearly see a massive deletion in chromosome 3.

<br>
To finish we just need to draw 3 tracks + positional links to represent
SVs.

Unfortunately the vcf format has not been designed for SVs. SVs are
defined by 2 breakpoints and the vcf format stores the second one in the
info field. So we will need to extract this information to draw these
calls.

  ```R
  chrEnd=NULL
  posEnd=NULL
  for (i in 1:dim(sv)[1]) {
      addInfo=strsplit(as.character(sv[i,8]),split=";")
      chrInf=strsplit(addInfo[[1]][3],split="=")
      chrEnd=c(chrEnd,chrInf[[1]][2])
      posInf=strsplit(addInfo[[1]][4],split="=")
      posEnd=c(posEnd,posInf[[1]][2])
  }
  svTable=data.frame(paste("chr",sv[,1],sep=""),as.numeric(sv[,2]),as.numeric(posEnd),paste("chr",chrEnd,sep=""),as.character(sv[,5]))
  ```

<br>
Now that we have reformatted the SV calls, let’s draw them.

  ```R
  typeE=c("<DEL>","<INS>","<INV>")
  colE=c("blue","black","green")
  for (i in 1:3) {
          bed_list=svTable[svTable[,5]==typeE[i],]
          circos.genomicTrackPlotRegion(bed_list,stack=TRUE, panel.fun = function(region, value, ...) {
                  circos.genomicPoints(region, value, cex = 0.5, pch = 16, col = colE[i], ...)
          })
  }

  bed1=cbind(svTable[svTable[,5]=="<TRA>",1:2],svTable[svTable[,5]=="<TRA>",2]+5)
  bed2=cbind(svTable[svTable[,5]=="<TRA>",c(4,3)],svTable[svTable[,5]=="<TRA>",3]+5)

  for (i in 1:dim(bed1)[1]) {
      circos.link(bed1[i,1],bed1[i,2],bed2[i,1],bed2[i,2])
  }
  ```

<br>
A good graph needs a title and legend:

  ```R
  title("Somatic calls (SNV - SV - CNV)")
  legend(0.7,1.4,legend=c("SNV", "CNV-DUPLICATION","CNV-DELETION","SV-DELETION","SV-INSERTION","SV-INVERSION"),col=c("orange","red","blue","blue","black","green","red"),pch=c(16,15,15,16,16,16,16,16),cex=0.75,title="Tracks:",bty='n')
  legend(0.6,0.95,legend="SV-TRANSLOCATION",col="black",lty=1,cex=0.75,lwd=1.2,bty='n')
  ```

<br>
You should obtain a plot like this one:

![image](images/circos.png)

<br>
!!! note "Question"
    Could you generate the graph and save it into a pdf file?

    !!! success ""
        ??? "**Answer**"
            ```R
            circos.clear()
            pdf("circos.pdf")
            ...
            dev.off()
            ```

Finally exit R
  ```R
  q("yes")
  ```

***
## Acknowledgements

Mathieu Bourgey would like to thank and acknowledge Louis Letourneau for this help and
for sharing his material. The format of the tutorial has been inspired
from Mar Gonzalez Porta. I also want to acknowledge Joel Fillon, Louis
Letrouneau (again), Francois Lefebvre, Maxime Caron and Guillaume
Bourque for the help in building these pipelines and working with all
the various datasets.

## License

This work is licensed under a Creative Commons Attribution-ShareAlike
3.0 Unported License. This means that you are able to copy, share and
modify the work, as long as the result is distributed under the same
license.

# SCIE4002 Genomics Workshops

This repository contains the three core workshops for SCIE4002 at UWA, focused on hands-on genomics analysis using real data and standard tools.

## Contents
### Workshops
* [1. Genome Annotation](#1-genome-annotation)
* [2. Phylogeny](#2-phylogeny)
* [3. Intraspecific Variance](#3-intraspecific-variance)
* [4. Research Projects](#4-research-projects)


### 1. Genome Annotation

In this practical, students annotate the chloroplast genome of *Aldrovanda vesiculosa* using two tools: GeSeq and Chloë. The aim is to compare their annotation outputs, extract gene features, and evaluate consistency across species. Students learn to wrangle GFF3 files, apply regular expressions, automate sequence extraction, and interpret gene loss patterns across related taxa. 

[→ View Practical 1: Genome Annotation](prac1_annotation.md)

**Key tools**: GeSeq, Chloë, awk, sed, grep, MAFFT, . 

In the link you find a streamlined summary of this section as well as a code along.

## 2. Phylogeny

This practical introduces students to phylogenetic tree building using two approaches: MinHash with neighbour-joining and maximum likelihood with IQ-TREE2. A third method, Bayesian inference with MrBayes, is discussed conceptually to highlight differences in statistical frameworks and runtime considerations. Students align chloroplast genes, build and interpret trees, and evaluate method accuracy and speed. Skills include working with Newick format, model selection, and phylogenetic visualisation.

[→ View Practical 2: Phylogeny](prac2_phylogeny.md)

**Key tools**: Mashtree.jl, IQ-TREE2, MAFFT, iTOL, standard shell utilities.
**Mentioned**: MrBayes (concept only).

In the link you find a streamlined summary of this section as well as a code along.

## 3. Intraspecific Variance

This practical focuses on detecting and interpreting chloroplast genome variation across *Aldrovanda* specimens. Students learn to align sequencing reads to a reference, perform variant calling before and after contamination filtering, and assess data quality impacts on results. The final step maps high-confidence variants to annotated genes. Key skills include short-read mapping, VCF interpretation, basic filtering logic, and critical evaluation of technical artefacts in variant detection.

[→ View Practical 3: Intraspecific Variance](prac3_variants.md)

**Key tools**: BBMap, `samtools`, `bcftools`, `bedtools`, Unix shell utilities


## 4. Research Projects

You will complete a guided research project in small groups of 4 members, working independently to analyse a real genomic dataset. Each group is provided with the background and hypotheses around the data that was created. The focus is on conducting reproducible analyses and interpreting results using skills learned in practicals.

Working effectively in a research team is a critical professional skill in science. This project is designed to help you develop collaboration, communication, time management, and problem-solving skills that are highly valued by employers. Active participation and equitable contribution from all members are expected.

While collaboration is essential, each student is responsible for writing their own report, demonstrating their understanding of the dataset and findings. 

**Expected outcomes**:

* Independent, reproducible analysis
* Individual written report addressing the project aims
* Group accountability and evidence of participation
* Clear biological interpretation of results

**Onboarding of projects will happen very soon!**

[→ View Project Briefs ](project_onboarding.md
)


## Resources 
>**Note:** This course avoids custom scripts. Standard tools are used wherever possible. Some custom script were unavoidable.


Here the list of resources I installed for this prac. So if you go work or study somewhere and you need these tools, they are all freely available. (Installation may vary depending on your set up)

### Chloe local install
In julia REPL install:

```julia
using Pkg
Pkg.add("https://github.com/ian-small/Chloe.jl)
```

### Conda Packages:
You can install micromamba or anaconda to create the environment use in this prac and then add the packages into it. Most palcgae are available through the bioconda or conda-forge channel.

#### links
* https://anaconda.org/bioconda/getorganelle
* https://anaconda.org/bioconda/mafft
* https://anaconda.org/bioconda/mashtree
* https://anaconda.org/bioconda/iqtree
* https://anaconda.org/bioconda/mrbayes
* https://anaconda.org/bioconda/bbmap
* https://anaconda.org/bioconda/samtools
* https://anaconda.org/bioconda/bcftools
* https://anaconda.org/bioconda/bedtools


Install guide using micromamba:

```bash
#create envrionment
micromamba create -n scie4002 -c conda-forge -c bioconda mafft mashtree iqtree mrbayes bbmap samtools bcftools bedtools getorganelle
```
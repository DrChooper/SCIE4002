# Workshop 2: Phylogeny
## Introduction

The Phylogeny Workshop focuses on constructing and comparing phylogenetic trees using three widely used methods: Minhash & Neighbour-joining (Mashtree), Maximum Likelihood (IQTREE2), and Bayesian Inference (MrBayes). Each method makes different assumptions about evolutionary processes and varies in speed and accuracy.

Workshop 2 builds on your annotation work from Workshop 1 and is structured in three blocks. You will use established phylogenetic software to align sequences and generate trees, followed by visual inspection and comparison in iTOL.

Each block includes a clear goal and checkpoint. As before, code along as you go and alert teaching staff if you fall behind. A coffee/snack break is scheduled between each section.

## Table of Content

1. [Workshop Plan](#workshop-plan)
2. [Code along](#code-along)

   * [Block 1: Minhash and Neighbour-joining (25 min)](#block-1-minhash-and-neighbour-joining-25-min)
   * [Block 2: Maximum Likelihood Tree with IQTREE2 (60 min)](#block-2-maximum-likelihood-tree-with-iqtree2-60-min)
   * [Block 3: Bayesian Tree with MrBayes (60 min)](#block-3-bayesian-tree-with-mrbayes-60-min)
   * [Comparison: Mashtree - IQtree - MrBayes - Taxonomy](#comparison-mashtree---iqtree---mrbayes---taxonomy)
  


## Workshop Plan

### Recap: What do use phylogenetic trees for? (20 min)

* Phylogenetic trees are models of evolutionary relationships based on shared ancestry.
* Tree topology reflects how sequences or species diverged over time.
* Trees are used to infer:
  - Species relationships
  - Evolutionary innovation
  - Taxonomic placement
* Different methods make different assumptions:
  - **Minhash + NJ**: distance-based, fast, approximate
  - **ML (IQTREE2)**: statistical inference of most likely tree under a model
  - **Bayesian (MrBayes)**: posterior distribution of trees based on prior and likelihood
* All methods output trees in the Newick format or similar format, which can be visualised and compared using iTOL.

📖 *Resources:*

* [Mashtree](https://github.com/lskatz/mashtree)
* [IQ-TREE documentation](http://www.iqtree.org/doc/)
* [MrBayes manual](https://nbisweden.github.io/MrBayes/)
* [Newick format guide](https://en.wikipedia.org/wiki/Newick_format)
* [iTOL tree viewer](https://itol.embl.de)

---

### Blocks

### 1: Minhash & Neighbour-joining (25 min)

- Construct a tree from whole-genome assemblies using Mashtree  
- Upload and visualise the tree in iTOL  
- Root the tree and interpret relationships  

**Checkpoint + 10 min break:** Mashtree tree saved, renamed, and visualised in iTOL  

---

### 2: Maximum Likelihood Tree with IQTREE2 (60 min)

- Align sequences and run IQTREE2 using codon models and branch support metrics  
- Inspect run time and visualise the tree in iTOL and compare it to Mashtree  

**Checkpoint + 15 min break:** ML tree built, logged, and uploaded to iTOL  

---

### 3: Bayesian Tree with MrBayes (60 min)

If time:
- Convert codon alignment to NEXUS format using Julia  
- Run MrBayes interactively with codon model settings  
- Monitor convergence (split frequencies, diagnostics)  
- Generate a consensus tree and visualise it in iTOL  
- Evaluate posterior probabilities and compare to ML and NJ trees  

**Wrap-up (15 min):** Compare all three trees and methods  
- Which is fastest?  
- Which gives the most statistical confidence?  
- Are the topologies consistent or conflicting? 



## Code along
### Block 1: Minhash and Neighbour-joining (25 min)

Minhash is used to quickly estimate Jaccard similarity between genomic datasets based on k-mers (substrings of fixed length). This distance relationship can be used to construct relational trees by methods such as NJ. 

Step 1: Set yourself up in the server
Log onto the server and activate the software environment

```bash
conny activate
#set the source directory
sourcedir=/mnt/s-ws/everyone/annotation/
```


Step 2: load Mashtree and contruct the Newick string

```bash
# use conda Mashtree
# kmer size = 16, minhash sketch size = 100000
# Run mashtree on all FASTA files in the directory
mashtree $sourcedir/*.fasta --kmerlength 16 --sketch-size 100000 > mashtree.nwk
```

- How long did it take to make this? Remember how long it felt and now


- The tree has sequence IDs. But you can rename the sequence IDs to with the NCBI taxid:
    
    1. replace `xxx` with your Newick string
    2. select the accession from the metadata file
    3. replace Accession with taxa ID

```bash
tree="xxx"

while IFS=$'\t' read -r latin name taxid accession carnivorous australian
do
    # escape special characters in accession (for safe sed replacement)
    escaped_acc=$(printf '%s\n' "$accession" | sed 's/[][\.*^$/]/\\&/g')
    tree=$(sed "s/\b$escaped_acc\b/$taxid/g" <<< "$tree")
done < <(tail -n +2 "$sourcedir/genome_metadata.tsv")

echo "$tree"
```



#### iTOL browser 
- paste the Newick tree string into the [iTOL](https://itol.embl.de/) browser tool and click upload. This will show you the tree with the taxaid numbers.

- **to display the species go to the control panel>Advanced and then scroll down to find the 'Other functions' section. Choose 'Autoassign taxonomy' 'NCBI'.** When the scary red panel pops up click 'assign taxonomy' and then 'reload page'.  

- Root the tree with *Liriodendron tulipifera* by clicking on the name. Go to the 'tree structure' in the drop down menu and choose 'reroot tree ...' *why: this is a basal angiosperm and the others are more evolved. This makes it an outgroup.* 


- What species is Aldrovanda most closely related to in this tree?
- You can try different values for the kmer size (10-32) and the sketch size (1000-10000).Do you get different trees?

### Block 2: Maximum Likelihood Tree with IQTREE2 (60 min)

IQTREE2 is a versatile software tool used for constructing phylogenetic trees based on maximum likelihood (ML) methods. It employs codon models for nucleotide sequence evolution, making it suitable for analyzing protein-coding genes

- First we will make a codon alignment using the multiple sequence alignment of proteins you made in the 'Annotation' module to guide the alignment of the corresponding nucleotide sequences

```bash
#go to your folder
cd rpoC2
# combine all nuclotide sequences
cat *.rpoC2.nt.fa > allrpoC2.nt.fa
#the below calls to run the julia script to guide alignment to codons
nt2codon.jl rpoC2.protein.msa allrpoC2.nt.fa rpoC2.codon.msa
less rpoC2.codon.msa
```

- Now we can use iqtree2 to construct a phylogenetic tree by maximum likelihood using this data.
- `-T 8` tells iqtree2 to use 8 parallel 'threads' so that it can use multiple processors in the server simultaneously (makes it quicker).
- `-st` CODON tells iqtree2 that we are using sequence data in codon alignment.
- `-o` is telling iqtree2 what the outgroup taxon is so it can root the trees correctly.
- `--alrt 1000` and `-B 1000` tell iqtree2 to estimate the confidence in the branches in the tree using 1000 replicates using an approximate likelihood ratio test (`--alrt`) and a conventional bootstrap test (`-B`). Much more info on [IQ-TREE here](http://www.iqtree.org/doc/)


```bash
iqtree2 -T 8 -s rpoC2.codon.msa -st CODON -o NC_008326.rpoC2 --alrt 1000 -B 1000
```

- you can use top to see what the server is doing

```bash
top
```

- The output log - How long did iqtree take (CPU time is what matters)?

```bash
less rpoC2.codon.msa.log

# Total number of iterations: 102
# CPU time used for tree search: 609.736 sec (0h:10m:9s)
# Wall-clock time used for tree search: 76.756 sec (0h:1m:16s)
# Total CPU time used: 1042.838 sec (0h:17m:22s)
# Total wall-clock time used: 131.379 sec (0h:2m:11s)
```

*So this took about 2 minutes when using 8 threads*

----
**Let's have a look at the tree making information**

```bash
less rpoC2.codon.msa.iqtree
```
- Codon model used: MG+F1X4+G4 (selected by ModelFinder)
- Alignment: 11 sequences, 1421 codon sites
- Support: 1000 SH-aLRT and 1000 ultrafast bootstrap replicates
- Execution: 8 threads, total wall time 2 min 11 sec

*[For more detailed info on the tree output](supplement/iqtree_output.md)

**Tree vs Consensus Tree**

IQ-TREE produces multiple tree files. Here are the key differences:

- `*.treefile`: This is the best maximum likelihood (ML) tree. It is the single most likely tree based on your data and model.(has not support values)
- `*.contree`: This is the consensus tree from the bootstrap replicates. It shows only the relationships supported by a majority (e.g. 95%) of bootstrap trees.

Summary:
- Use the `.treefile` for downstream analysis or further inference.
- Use the `.contree` for visualisation and reporting with support values.

Use the consensus treefile and replace IDs as previously and look at it in iTOL.

```bash
tree="xxx"

while IFS=$'\t' read -r latin name taxid accession carnivorous australian
do
    # escape special characters in accession (for safe sed replacement)
    escaped_acc=$(printf '%s\n' "$accession" | sed 's/[][\.*^$/]/\\&/g')
    tree=$(sed "s/\b$escaped_acc\b/$taxid/g" <<< "$tree")
done < <(tail -n +2 "$sourcedir/genome_metadata.tsv")

echo "$tree"
```

- paste the Newick tree into https://itol.embl.de/
- replace labels and root to *Liriodendron tulipifera* 
- display the bootstrap values: Go to *Advanced>Branch metadata display>Bootstrap/metadata>Text*
- Is the tree the same or different to the one generated by Mashtree? Let's compare while MB is running


### Block 3: Bayesian Tree with MrBayes (60 min)

- Use the same data as for iqtree2, but it needs to be converted to the Nexus format
- MSA formats (e.g. FASTA) store only the aligned sequences.  
- NEXUS includes the alignment plus extra info like taxa, partitions, trees, and analysis commands (used in MrBayes, BEAST, etc.).

- we have a custom little script to help convert this

```bash
fasta2nex.jl rpoC2.codon.msa rpoC2.codon.nex

less rpoC2.codon.nex
```

1. Run MrBayes interactively by typing `mb`
2. Load the alignment file 

```bash
exe rpoC2.codon.nex
```
3. Set the codon model using the `lset` command:

```bash
lset nucmodel=codon omegavar=M3 nst=mixed rates=invgamma
```
Explanation of parameters:

* `nucmodel=codon`: tells MrBayes the data are coding sequences (codons, not just nucleotides).
* `omegavar=M3`: allows variation in the dN/dS ratio (ω) across sites using a discrete distribution (M3).
* `nst=mixed`: allows MrBayes to sample across different nucleotide substitution models during the run.
* `rates=invgamma`: models rate variation across sites using a mix of invariant sites and gamma-distributed rates.


4. Set MCMC parameters for quick testing:

```bash
mcmcp ngen=10000
mcmcp diagnfreq=1000
mcmcp samplefreq=100
mcmcp printfreq=100
```

*Meaning of MCMC Parameters in MrBayes*

- `mcmcp ngen=10000`: Run the MCMC for 10,000 generations.

- `mcmcp diagnfreq=1000`: Check convergence diagnostics every 1,000 generations.

- `mcmcp samplefreq=100`: Save one sampled tree every 100 generations.

- `mcmcp printfreq=100`: Display progress updates every 100 generations.

5. Now we can start the run

```bash
mcmc
```
**Warning:** ⏳ **Prepare for a wait that rivals a Netflix binge!**

### While MrBayes performs its Bayesian wizardry:

**Split frequencies**
 
 - a Bayesian measure of consistency between different runs of the analysis to determine if the chains have converged to the same solution.
 - a lower number means the chains are more consistent with each other
 - threshold: a value below 0.01 is considered good convergence (0.01 - 0.05 is moderate, >0.05 is poor)
 - if split frequencies are not above the threshold it will ask you if you want to run more.
```bash
 Average standard deviation of split frequencies: 0.009821

   Continue with analysis? (yes/no):       Additional number of generations: 

   Analysis completed in 30 mins 27 seconds
   Analysis used 5027.82 seconds of CPU time
   Likelihood of best state for "cold" chain of run 1 was -17869.13
   Likelihood of best state for "cold" chain of run 2 was -17903.72
   ```

- runtime: 30 min (MB), 2 min (IQtree), seconds (Mashtree) 

**Wrapping Up Your MrBayes Run**

* Use `sump` to **check if the run converged** and view general statistics:

  ```bash
  sump
  ```


  * Log likelihoods plateaued between gen 2500–3000 → stationarity likely reached
  * Average SD of split frequencies = 0.009821 → acceptable convergence
  * Harmonic mean of likelihoods: -18045.48 (run1), -18008.86 (run2)  
  * Best model: gtrsubmodel\[123454] with posterior prob. 0.375
    * GTR = General Time Reversible Model
    > *Note: In a GTR submodel `[123454]`, the six possible nucleotide substitutions share rates according to their numeric labels:*
    > - A↔C = rate1  
    > - A↔G = rate2  
    > - A↔T = rate3  
    > - C↔G = rate4  
    > - C↔T = rate5  
    > - G↔T = rate4


* Use `sumt` to **generate and view the consensus tree** with support values:

  ```bash
  sumt
  ```

  * 42 trees read, 32 sampled after discarding 25 percent burn-in
  * Most clades have posterior probability of 1.0, indicating strong support
    * >0.9 is considered strong support
    * <0.9 is considered to have uncertaincy
  * Split frequency 0.0098 and PSRF around 1.0 show good convergence across runs
  * shows the tree structures in terminal


* Exit MrBayes:

  ```bash
  quit;
  ```


**MrBayes tree files**


```bash
cat rpoC2.codon.nex.con.tre 
```

- Replace the accessions with taxa ID

```bash
sourcedir="/mnt/s-ws/everyone/annotation"

tree=$(<rpoC2.codon.nex.con.tre)

while IFS=$'\t' read -r latin name taxid accession carnivorous australian
do
    escaped_acc=$(printf '%s.rpoC2' "$accession" | sed 's/[][\.*^$/]/\\&/g')
    tree=$(sed "s/\b$escaped_acc\b/$taxid/g" <<< "$tree")
done < <(tail -n +2 "$sourcedir/genome_metadata.tsv")

echo "$tree" > rpoC2.codon.mb.nex
```

- paste the Nexus tree into iTOL (to display use the cat command)
```bash
cat rpoC2.codon.mb.nex
```


**If we do not get to the end today try it later and leave your computer running**

----
### Comparison: Mashtree - IQtree - MrBayes - Taxonomy
The main aim today is to understand that there are different methods and how they work and what they produce.

- Let's take a look at the current taxonomy tree that is saved in the folder for you. Use `less` to see the tree in Newick format

```bash
less /mnt/s-ws/everyone/annotation_phylogenetics/carnivores/carnivores.nw 
```
- paste the tree into https://itol.embl.de/ to visualise it in a graphical form
- Root the tree with *Liriodendron*
- What species is *Aldrovanda* most closely related to?


Now compare all 3 trees and all three computing times.
- Which of the three methods is the fastest? 
- Which is the most accurate?


### Wrap-up (15 min): Compare all three trees and methods

Which is fastest?

Which gives the most statistical confidence?

Are the topologies consistent or conflicting?

What did you learn about *Aldrovanda* and its place in the tree?

What method would you use for clesely related species?

What method would you use for a large data set?

Could you combined these methods?


### Tree File Cheatsheet

Use the links below to access and download the tree files used in this workshop. You can paste the contents into iTOL (https://itol.embl.de/) or other tree visualization tools.

- [Mashtree (accession labels)](assets/cheats/mashtree.nwk)
- [Mashtree (NCBI taxon labels)](assets/cheats/mashtree_NCBI_taxaID.nwk)
- [IQ-TREE (accession labels)](assets/cheats/iqtree.nwk)
- [IQ-TREE (NCBI taxon IDs)](assets/cheats/iqtree_NCBI_taxaID.nwk)
- [MrBayes (accession labels)](assets/cheats/mrbayes.nex)
- [MrBayes (NCBI taxon IDs)](assets/cheats/mrbayes_NCBI_taxaID.nex)
- [Taxonomy reference tree](assets/cheats/taxonomy.nwk)



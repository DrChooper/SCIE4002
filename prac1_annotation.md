# Workshop 1: Genome Annotation
## Introduction
The Genome Annotation Workshop focuses on annotating the Aldrovanda vesiculosa chloroplast genome using GeSeq and Chloë, highlighting their different approaches and comparing outputs. Participants will manipulate GFF files and use text-wrangling techniques to prepare data for analysis and phylogenetic studies.

This practical is divided into three blocks. Each block has a clear goal and checkpoint. You are expected to code along through each section. If you fall behind, please alert the staff so we can assist. If needed, checkpoint files are provided so you can continue with the analysis. We plan to have a coffee and snack break between each session.

## Table of Content
1. [Workshop Plan](#workshop-plan)  
2. [Code along](#code-along)  
   - [Block 1 code along: Run Annotations (20 min)](#block-1-code-along-run-annotations-20-min)  
   - [Block 2: Extract and Compare Gene Features (50 min)](#block-2-extract-and-compare-gene-features-50-min)  
   - [Block 3: Compare Across Genomes (55 min)](#block-3-compare-across-genomes-55-min)  



## Workshop Plan
### Recap: Why do we annotate (20 min)
* DNA sequences are not useful on their own — annotation reveals the structure and function within the sequence.
* **Structural annotation** identifies genes, exons, introns, and other genomic elements.
* **Functional annotation** links those features to known biological roles, usually through sequence comparison.
* Accurate annotation is critical for downstream analysis, including gene expression, variant calling, and phylogenetics.
* Annotation errors are common — examples include wrong start sites, incorrect strands, or missing genes 
* This prac focuses on **chloroplast genome annotation**, using two tools:

  * **GeSeq** – sensitive, based on feature comparisons (BLAST)
  * **Chloë** – precise, based on whole-genome alignment

📖 *Resources:*

* [GeSeq](https://chlorobox.mpimp-golm.mpg.de/geseq.html)
* [Chloë](https://chloe.plastid.org)
* [GFF3 format overview](https://github.com/The-Sequence-Ontology/Specifications/blob/master/gff3.md)
* [Nature article on annotation uncertainty](https://www.nature.com/articles/d41586-018-05462-w)

### Blocks
### 1: Run Annotations (20 min)  
- Annotate the genome using GeSeq and Chloë via their web interfaces  
- Download, rename, and upload both `.gff3` files to the server  

**Checkpoint + 10 min break:** Both annotations are saved to the server  



### 2: Extract and Compare Gene Features (50 min)  
- Extract gene features using `awk`, `cut`, and `sed`  
- Clean gene names and count unique entries  
- Compare GeSeq and Chloë annotations using `comm`  

**Checkpoint + 15 min break:** Cleaned gene lists and comparison summary saved  

### 3: Compare Across Genomes (55 min)  
- Investigate gene presence/absence across species  
- Extract a single gene (e.g. *rpoC2*) across all genomes  
- Align output for the phylogeny practical  

**Wrap-up (10 min):** Final files prepared for the next workshop  


## Code along

### Block 1 code along: Run Annotations (20 min)

From this point on, follow along with the commands. Ask for help if you fall behind — checkpoint files are available.

#### Getting the FASTA file

1. Set your source directory  
   ```bash
   #activate my environment
   conny activate
   #to save you typing the location of all the data on ther server
   sourcedir=/mnt/s-ws/everyone/annotation
   ```
   
2. You’ll need the FASTA file on your computer. You can either:
   - **Download it directly from GitHub:**  
     [Download Av.cp.final.fasta](https://github.com/DrChooper/SCIE4002/raw/main/assets/Av.cp.final.fasta)

   - **Copy it from the server using `scp`:**

     ```bash
     #replace with your username and server ip number
     scp studentaccount@yourserverip:$sourcedir/aldrovanda_vesiculosa/Av.cp.final.fasta .
     ```


#### GeSeq Annotation

1. Upload `Av.cp.final.fasta` to the GeSeq website:  
   https://chlorobox.mpimp-golm.mpg.de/geseq.html

2. Use the settings shown in the lecture slides.

3. When GeSeq finishes:
   - Download the GFF file
   - Rename it to something simple, e.g. `Av.geseq.gff3`
   - Upload it to your home directory on the server using either a GUI or:

     ```bash
     scp Av.geseq.gff3 studentaccount@yourserverip:~/
     ```


#### Chloë Annotation

1. Upload `Av.cp.final.fasta` to the Chloë annotation website:  
   https://chloe.plastid.org/annotate.html

2. Use the settings shown in the lecture slides.

3. When Chloë finishes:
   - Download the GFF file
   - Rename it to something simple, e.g. `Av.chloe.gff3`
   - Upload it to your home directory on the server using either a GUI or:

     ```bash
     scp Av.chloe.gff3 studentaccount@yourserverip:~/
     ```


#### Check annotations (on the server)

We now move to working **on the server**. Use `less` to view both GFF files. Press `q` to exit.

```bash
less Av.geseq.gff3
less Av.chloe.gff3
```

Both files follow the GFF3 format but use different naming styles.

#### ✅ **Checkpoint and 15 min break**

You should now have:

* `Av.geseq.gff3` and `Av.chloe.gff3` uploaded to the server
* Viewed both files using `less`

Ready to clean and compare in Block 2.



---
### Block 2: Extract and Compare Gene Features (50 min)

We’ll extract and clean the gene features from both annotations to make them comparable.

#### Step 1: Extract gene features from the GeSeq file

```bash
awk '$3 == "gene"' Av.geseq.gff3 | cut -f 4,5,9 | sort -n > Av.geseq.txt
head Av.geseq.txt
```

This filters lines with `gene` features, keeps columns 4 (start), 5 (end), and 9 (attributes), sorts them, and saves the result.

#### Step 2: Clean the final column with `sed`

```bash
sed -E -i 's/[^\t]*;gene=([^;]*);.*/\1/' Av.geseq.txt
head Av.geseq.txt
```

#### Step 3: Extract gene features from the Chloë file
*we join the two command with the `|` symbol and do it in one step*

```bash
awk '$3 == "gene"' Av.chloe.gff3 | cut -f 4,5,9 | sort -n | sed -E 's/[^\t]*;name=([^;]*)/\1/' > Av.chloe.txt
head Av.chloe.txt
```
👉 [Need help understanding these command? Click here!](supplement/regex.md)

#### Step 4: Count unique gene names

```bash
cut -f 3 Av.chloe.txt | uniq | wc -l
cut -f 3 Av.geseq.txt | uniq | wc -l
```

#### Step 5: Compare gene sets

Use `comm` to identify differences between gene lists:

```bash
comm -3 <(cut -f 3 Av.geseq.txt | sort) <(cut -f 3 Av.chloe.txt | sort)
```
This shows genes unique to GeSeq (left column) and unique to Chloë (right column).
We use [process substitution](https://www.gnu.org/software/bash/manual/html_node/Process-Substitution.html) to compare sorted output directly.

Some GeSeq annotations include genes marked as `-fragment`, which may represent pseudogenes, low confidence or partial hits.
To clean up the names but include the hits we delete `-fragment` from the name:

```bash
sed -E -i 's/-fragment//' Av.geseq.txt
```

Then re-run the comparison:

```bash
comm -3 <(cut -f 3 Av.geseq.txt | sort) <(cut -f 3 Av.chloe.txt | sort)
```
#### Step 6: Final Comparison and Interpretation

##### Discussion prompts

* Why did Chloë detect fewer genes?

  * GeSeq uses a more sensitive BLAST search, which may detect more genes but also more false positives (e.g. fragments, pseudogenes).
  * Chloë is more conservative and may miss borderline cases (false negatives).
* This highlights a broader challenge in genome annotation: distinguishing real, functional genes from pseudogenes or low-confidence matches.

#### Step 7: Detailed comparison using `diff`

```bash
diff -u Av.geseq.txt Av.chloe.txt
```
This highlights line-by-line differences between the two annotation outputs.
👉 [Need help understanding the output? Click here](supplement/diff_output.md)

#### Step 8: Visual comparison in IGV

1. Download and install IGV:
   [https://software.broadinstitute.org/software/igv/download](https://software.broadinstitute.org/software/igv/download)

2. Load the files:

   * `Av.cp.final.fasta` → Genome
   * `Av.geseq.gff3` → Annotation track
   * `Av.chloe.gff3` → Second annotation track

You’ll see that annotations can differ substantially, even on the same genome, depending on the tool used.

**Takeaway:** For consistent comparisons across multiple genomes, always use the same annotation pipeline. Differences in tools can introduce more noise than actual biological variation


#### ✅ **Checkpoint and 15 min break**

You should now have:

* `Av.geseq.txt` and `Av.chloe.txt` cleaned
* Gene counts for each
* A list of shared and unique genes

Ready for Block 3 after a short break.


---
### Block 3: Compare Across Genomes (55 min)

We’ll annotate and compare gene content across multiple related chloroplast genomes.

#### Step 1: Explore the data

View available genome metadata in tabular format:

```bash
cat $sourcedir/genome_metadata.tsv | column -t -s $'\t' | less -S
```
This is the core dataset for annotation and phylogenetics across diverse plant genomes. It includes carnivorous and non-carnivorous species, Australian natives, and distant relatives. We’ll use it to compare annotations and extract gene sequences in this and later workshops.

#### Step 2: Re-annotate with Chloë

Run Chloë on all genomes:

```bash
mkdir chloe
chloe annotate --no-transform -o chloe/ $sourcedir/*.fasta
```

Check that 11 `.gff3` files are generated in the `chloe/` directory.

#### Step 3: Load gene reference list

View the file:

```bash
cat $sourcedir/cp_proteins.csv | column -t -s, | sort | less -S
```

Extract gene names:

```bash
cut -d ',' -f 1 $sourcedir/cp_proteins.csv | tail -n +2 | uniq > gene_names.txt
less gene_names.txt
```

#### Step 4: Count genes across genomes

This step will check how often each gene is annotated across all 11 .gff files in the `chloe/` directory. We’ll focus on the gene features only (not introns, CDS, or tRNAs), using a curated list of chloroplast-encoded proteins.

Use `grep` to find how often each gene is annotated across all `.gff` files (using a loop to only count genes not introns etc):

```bash
while read -r gene
do
  numgenes=$(awk -F'\t' '$3 == "gene" && $9 ~ "gene="gene";"' gene="$gene" chloe/*.gff | wc -l)
  echo "$gene $numgenes" >> gene_numbers.txt
done < gene_names.txt
```

This script:

* Reads each gene name
* Searches only lines where the feature type is gene
* Looks for gene=GENENAME; in the attributes column
* Appends the count to gene_numbers.txt

View the results:

```bash
less gene_numbers.txt
```

Sort by least frequent:

```bash
sort -nk 2 gene_numbers.txt | head
```

Investigate specific gene absences:

```bash
grep "ndh" $sourcedir/cp_proteins.csv
```

Check which plants are missing these genes:
```bash
awk -F'\t' '$3 == "gene" && $9 ~ /gene=ndhA;/' chloe/*.gff
```
Reflection:
* Nearly all genes appear in all 11 genomes
* Some genes (like the ndh family) are missing in multiple genomes
* These encode parts of the plastoquinone oxidoreductase complex
* Are the losses random, or do the same plants lose the same genes?

#### Step 5: Calculate average gene lengths

We now want to calculate the **average length of each gene** across the 11 genomes. This helps identify genes that are long enough to be informative for downstream analyses such as phylogenetics.

```bash
while read -r gene
do
  count=0
  sumlength=0
  awk -F'\t' -v gene="$gene" '$3 == "gene" && $9 ~ "gene="gene";" {
    start = $4; end = $5;
    sumlength += (end - start + 1);
    count += 1
  } END {
    if (count > 0) print gene, int(sumlength / count);
    else print gene, 0;
  }' chloe/*.gff >> gene_lengths.txt
done < gene_names.txt
```

Then view the results:

```bash
sort -nk 2 gene_lengths.txt | less
```

This shows that gene lengths range from under 100 bp to over 6 kb. Longer genes are often more useful for phylogenetics, provided they are present in all genomes and align well.


#### Step 6: Extract sequence data

Pick a gene >1000bp present in all genomes (e.g. `rpoC2`).

Get coordinates:

```bash
awk -F'\t' '$3 == "gene" && $9 ~ /gene=rpoC2;/ { print FILENAME, $4, $5, $7 }' chloe/*.gff > rpoC2.tsv
less rpoC2.tsv
```

Check consistency:

```bash
while read -r id start end strand
  do
    length=$((end - start + 1))
    modulus=$((length % 3))
    echo "$id $length $modulus"
  done < rpoC2.tsv
```

Copy FASTA sequences:

```bash
cp $sourcedir/*.fasta chloe/
```

We'll use a homemade 'Julia' script to extract the gene sequences.

```bash
extract_gene_from_fasta.jl chloe rpoC2.tsv
```
* This should generate a nucleotide fasta file containing the rpoC2 gene sequence for each genome.
* Ideally we would use the nearly universally available samtools faidx for feature extraction instead of a homemade script.
* But samtools faidx cannot correctly extract features that cross the ends of the genome, which genes in circular genomes often do.

* Check that the extracted sequences start with a start codon and end with a stop codon


```bash
head Av.rpoC2.nt.fa
tail Av.rpoC2.nt.fa
```

#### Step 7: Translate and align sequences

Translate to protein:

```bash
translate_nt_aa.jl *.rpoC2.nt.fa
```

wTo keep it a bit easier to see maybe put all in the rpoC2 folder?
Concatenate and align:

```bash
mkdir rpoC2
#move all into folder
mv *.nt.fa rpoC2
mv *.protein.fa rpoC2
#go to folder and combine all sequences
cd rpoC2
cat *rpoC2.protein.fa > allrpoC2.protein.fa
#align using mafft
mafft --maxiterate 1000 --globalpair allrpoC2.protein.fa > rpoC2.protein.msa
```

View you alignment file by downloading the `.msa` file to your computer and upload it online:

* [https://www.ncbi.nlm.nih.gov/projects/msaviewer/](https://www.ncbi.nlm.nih.gov/projects/msaviewer/)
* [https://www.ebi.ac.uk/Tools/msa/mview/](https://www.ebi.ac.uk/Tools/msa/mview/)


#### MSA Colouring Guide for rpoC2 (Chloroplast)

Use the following settings in the NCBI MSA Viewer for meaningful interpretation of **rpoC2** alignments:

* **BLOSUM62**: 
  Highlights evolutionary substitutions (aa properties). Best general-purpose scheme for chloroplast genes.

* **Conservation**:
  Shows how conserved each position is. Use to identify critical functional sites.

* **Frequency-Based Difference**: 
  Highlights deviations from the consensus. Useful for spotting variation across taxa.


#### ✅ **Checkpoint and finish**

You should have:

* All 11 chloroplasts annotated with Chloë
* Counted gene presence across genomes
* Calculated gene lengths
* Extracted a multi-genome gene (e.g. `rpoC2`) as FASTA
* Aligned protein sequences

This completes the annotation practical. Retain your files for the next phylogenetics session.


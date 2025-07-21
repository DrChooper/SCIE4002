
# Workshop 3: Intraspecific Variance

## Introduction

This workshop focuses on detecting genetic variation within a species using chloroplast genome sequencing data. You will map reads, call variants, clean and filter your data, and analyze genetic variation across individuals. The example dataset includes *Aldrovanda* specimens from global locations. This complements the previous workshop on annotation by adding population-level resolution.

The workflow includes three rounds of variant calling:
- **Raw** reads
- **Cleaned** reads (contamination removed)
- **Filtered** high-confidence variants

We will explore how each step affects variant detection and what it tells us about biological vs technical variation.


## Table of Contents

1. [Workshop Plan](#workshop-plan)
2. [Code along](#code-along)

   * [Block 1: Mapping and Variant Calling on Raw Data (30 min)](#block-1-mapping-and-variant-calling-on-raw-data)
   * [Block 2: Quality Control and Re-calling with Cleaned Data (45 min)](#block-2-quality-control-and-re-calling-with-cleaned-data)   
   * [Block 3: High-Confidence Filtering and Variant Analysis (45 min)](#block-3-high-confidence-filtering-and-variant-analysis)
   * [Wrap-up of Comparison and Biological Interpretation (15 min)](#wrap-up-of-comparison-and-biological-interpretation-15-min)



## Workshop Plan

### Recap: What is Intraspecific Variance Analysis for (15 min)

- Detect within-species genetic variation for population or ecological analysis  
- Assess sequence quality, sample integrity, and contamination  
- Determine conserved vs variable regions (e.g. for barcoding)  

*Tools used*: `bbmap`, `samtools`, `bcftools`, `bedtools`  

--- 
### Blocks
### 1: Mapping and Variant Calling on Raw Data (30 min)

- Map trimmed reads to genetic reference using `bbmap`
- Sort and index BAM files
- Call variants (VCF format) using `bcftools`

**Checkpoint + 10 min break:** Raw VCF generated and `bcftools stats` run on uncleaned data

---
### 2: Quality Control and Re-calling with Cleaned Data (45 min)

- Remove Arabidopsis contamination using `bbmap` and output unmapped reads  
- Compare mapping summaries (mapped vs unmapped %)  
- Rerun mapping and variant calling on cleaned reads    

**Checkpoint + 10 min break:** Cleaned VCF created and variant counts compared to raw  

---
### 3: High-Confidence Filtering and Variant Analysis (45 min)

- Apply quality filters (QUAL>30, DP>20) to VCF  
- Compare filtered variants per sample  
- Identify outliers  
- Query variant positions and overlap with annotated genes using `bedtools`  

**Checkpoint + 5 min break:** High-confidence VCF analyzed, SNPs counted, gene variation summarized  

---
### Wrap-up and Comparison (15 min)

- Revisit outputs from:
  - Raw data > Cleaned data > Filtered high-quality data
- Evaluate:
  - Impact of contamination
  - True vs false variant signal
  - Which genes show intraspecific variation?

<!-- **Discussion prompts:**
- Which step had the biggest impact?  
- Do your samples form distinct clusters of similarity?  
- Could this pipeline be used for species ID?

--- -->

## Code along

### Block 1: Mapping and Variant Calling on Raw Data

**Background:**
- The reads have already been trimmed to remove adapter sequences.
- We are using `bbmap` as the alignment software. More information about `bbmap` can be found [here](https://sourceforge.net/projects/bbmap/).

**Step-by-Step Instructions:**

Step 1. Set yourself up
* Activate the software environment and set the source directory to the provided data (sequence reads)

    ```bash
    conny activate 
    sourcedir=/mnt/s-ws/everyone/annotation/Module_6_Variants
    ```
* make sure you are in the directory that contains the `Av.cp.final.fasta` file (we use it as the reference meaning we want to see what this one looks like compared to the other species that we know)
    
Step 2. Map the Reads to the Reference Genome:
* Use `bbmap` to align the reads and generate BAM files.

```bash
for sample in {1..2}; do
  bbmap.sh in1=$sourcedir/$sample.cp.R1.trimmed.fq.gz \
           in2=$sourcedir/$sample.cp.R2.trimmed.fq.gz \
           out=$sample.bam \
           ref=Av.cp.final.fasta \
           mappedonly=t
done
```
* *What is in a BAM file?: A BAM file is a compressed, binary version of a SAM file. It contains aligned sequencing reads, their positions on the reference genome, alignment details (CIGAR strings), mapping quality, flags (e.g. strand, paired), and optional tags.*


Step 3. Sort and Index BAM Files:
*  Use `samtools` to sort and index the BAM files. This step improves the efficiency of downstream analysis.
```bash
for sample in {1..2}; do
  samtools sort -T tmp -o $sample.sorted.bam $sample.bam
  samtools index $sample.sorted.bam
done
```
* *What does sorting and indexing do?: Sorting arranges reads by their position on the reference genome, enabling efficient downstream analysis. Indexing creates a companion file (.bai or .csi) that allows rapid retrieval of reads from specific genomic regions without scanning the entire file.*

Step 4. Comparing Sizes of Sorted and Unsorted BAM Files
* Use the `ls` command to list files and check their sizes:
```bash
ls -lh
```
* Compare the sizes of the .bam and .sorted.bam files.

* Sorted BAM files are typically smaller due to better compression of the sorted data.

Step 5. Remove Unsorted BAM Files
*  remove the unsorted BAM files.*(The sorted and unsorted files contain the same content. You want to keep the smaller files)*
```bash
rm [0-9].bam
```

Step 6: Variant Calling on Raw Data
* We use `bcftools` to call variants (meaning go through the aligner reads to detect differences in the sequence). This process will take several minutes.

* The output is in VCF format, a text-based format for storing variants ([specs here](http://samtools.github.io/hts-specs/)). We use VCF instead of the binary BCF since our files are small.

* `bcftools` assumes diploid samples by default, but chloroplast genomes are haploid. We set `--ploidy 1` to reflect this and avoid calling heterozygous variants.

```bash
bcftools mpileup -Ou -f Av.cp.final.fasta $sourcedir/bams/*.sorted.bam | bcftools call --ploidy 1 -mv -Ov -o Av.vcf
```
Step 7: Variant Summary using `bcftools stats`
* VCF files contain a lot of information. In this workshop, we will focus on the per-sample counts (PSC).
* Other sections of the output are also useful and can be explored [in the bcftools stats section of the documentation](http://samtools.github.io/bcftools/bcftools.html#stats).
* Use `grep` to extract the PSC lines from the `bcftools stats` output:

```bash
bcftools stats -s - Av.vcf | grep "PSC"
```

* The [summary output](supplement/Av_vcf_output.md) will show all columns (useful if you look at two alleles)

* Since we specified haploid genomes, only nHapRef (matches reference) and nHapAlt (variants) are relevant.

* Diploid-specific columns like nRefHom, nHets, etc., can be ignored.

To simplify the output for chloroplast analysis, extract only the relevant columns:

```bash
bcftools stats -s - Av.vcf | grep "PSC" | awk '{print $3, $10, $12, $13, $14}'
```
**Summary Conclusions**

- Most samples show **high coverage** (Avg Depth >100) and a **balanced number of variants** (nHapAlt ~70–140).
- **Sample 16** is an outlier:
  - Lowest coverage (52.7)
  - Highest number of variants (nHapAlt = 240)
  - Only 187 sites match the reference (very low nHapRef)
  - Contains missing data (24 sites) — all other samples have zero
- This suggests sample 16 may be:
  - Poor quality
  - Contaminated
  - Or genuinely divergent from the reference
- All other samples have **consistent nHapRef** (around 350–385) and **no missing data**, indicating good quality.


### Block 2: Quality Control and Re-calling with Cleaned Data

Step 1: Removal of Arabidopsis Contamination

In this step, we use `bbmap` to filter out reads that align to the *Arabidopsis thaliana* chloroplast genome. Arabidopsis is a frequent contaminant in plant sequencing data due to its widespread use in research laboratories. By mapping to the Arabidopsis reference and keeping only unmapped reads (`outu=`), we effectively remove contaminating sequences.

```bash
for sample in 6; do
  bbmap.sh \
    in1=$sourcedir/$sample.cp.R1.trimmed.fq.gz \
    in2=$sourcedir/$sample.cp.R2.trimmed.fq.gz \
    ref=/mnt/s-ws/everyone/NC_000932.fa \
    outu1=$sample.cp.R1.trimmed.clean.fq.gz \
    outu2=$sample.cp.R2.trimmed.clean.fq.gz \
    minratio=0.9 maxindel=3 bwr=0.16 bw=12 fast minhits=2 \
    qtrim=r trimq=10 untrim idtag printunmappedcount kfilter=25 \
    maxsites=1 k=14
done
```
Summary of BBMap Contamination Filtering Results

- *Reads Used*: 398,798 — total number of reads processed.
- *Unmapped Reads*: 88.6% — most reads did *not* match the Arabidopsis reference, which is expected and good.
- *Mapped Reads*: ~7% for each read direction — these are potential contaminants and were removed.
- **However**, Sample 16 did not show more Arabidopsis contamination than others, so its outlier status is likely due to poor quality or true divergence, not contamination.


Step 2: Re-map cleaned reads

*To avoid having to generate the maps during the prac, the cleaned BAM files are available in `$sourcedir/clean_bams`*

* use the same command as before but point it at the cleaned bam files:

```bash
bcftools mpileup -Ou -f Av.cp.final.fasta $sourcedir/clean_bams/*.sorted.bam | bcftools call --ploidy 1 -mv -Ov -o Av_clean.vcf
```

Step 3: Compare variant counts

*We want to see how the number of detected variants changes after cleaning.*

Each line in a VCF file (not the stats we lifted out) represents one variant. So, counting the number of lines gives us an estimate of how many variants were found.

*e.g. for our sample:*
```bash
wc -l Av.vcf
wc -l Av_clean.vcf
```
* cleaned VCF has fewer variants (−187 lines)
* removing contaminant reads reduced false variant calls

Step 4: Interpret variant shifts

* redo the stats using the cleaned vcf file

```bash
bcftools stats -s - Av_clean.vcf | grep "PSC"
```

or you can select the *relevant* columns only:

```bash
bcftools stats -s - Av_clean.vcf | grep "PSC" | awk '{print $3, $10, $12, $13, $14}'
```

* Sample 16: still has low depth and is missing data (persistent quality issues despite cleaning)
* Other samples: more consistent depth, and the total variant counts are more balanced.
* Cleaning removed potential contamination (causing spurious variants)


### Block 3: High-Confidence Filtering and Variant Analysis

In this final block, we focus on filtering variants for high confidence and analyzing their distribution across the genome to identify meaningful biological patterns.

Step 1: Filter for High-Confidence SNPs

* retain only high-quality SNPs with QC score (QUAL) > 30 and a depth (DP) > 20.

   *Reason for QC thresholds:*

   * *QUAL > 30: Variant has >99.9% confidence (Phred score), reducing false positives.*
   * *DP > 20: At least 20 reads support the variant, ensuring reliability and reducing errors from low*

```bash
bcftools filter -i 'TYPE="snp" && QUAL>30 && DP>20' Av_clean.vcf > Av.hiQ.vcf
```
* check how many variants we found in our sample    
```bash
wc -l Av.vcf #raw
wc -l Av_clean.vcf #decontaminated
wc -l Av.hiQ.vcf #QC filtered
```
What we have done:
* We started with 480 potential variants: Av.vcf (raw reads)
* After removing contaminant reads, we had 293 — many false variants were cleaned out: Av_clean.vcf (Arabidopsis removed)
* After filtering for high-quality, confident calls, we kept only 209 variants (Av.hiQ.vcf)

Step 2. Summarize Per-Sample Variant Stats

* Use `bcftools stats` to count and summarize the variants. The `grep "PSC"` command filters the output to show only per-sample counts.

```bash
bcftools stats -s - Av.hiQ.vcf | grep "PSC" | awk '{print $3, $10, $12, $13, $14}'
```

  - Do we have more true variants now?
  - what happened to the outlier?


Step 3: Biological interpretation Variant Count

- Which specimen is most or least different from the reference?  

```bash
# Most different from reference (highest nHapAlt)
bcftools stats -s - Av.hiQ.vcf | grep "PSC" | awk '{print $3, $13}' | sort -k2,2nr | head

# Least different from reference (lowest nHapAlt)
bcftools stats -s - Av.hiQ.vcf | grep "PSC" | awk '{print $3, $13}' | sort -k2,2n | head
```

* These counts reflect how many positions differ from the chloroplast reference genome. Higher counts may indicate biological divergence (e.g., different species or populations) or.... potential issues like residual contamination or mapping errors.

* Despite cleaning, **sample 16** still shows low depth and missing genotype calls, suggesting persistent quality issues. This highlights the importance of both **technical quality control** and **biological interpretation** in variant analysis.


Step 4:  Query Specific SNPs:
* Use the `bcftools` to extract SNPs from cleaned or hQC file and then compare

```bash
bcftools query -i 'TYPE="snp"' -f '%CHROM %POS %REF %ALT\n' Av_clean.vcf
```
* The output lists each SNP where *at least one* of the aligned samples has a different base

* If you want to have a look at how many samples (out of 16) differed and what is different:
```bash
bcftools query -i 'TYPE="snp"' -f '%CHROM %POS %REF %ALT [\t%GT]\n' Av_clean.vcf | \
awk '{
  count=0;
  for(i=5;i<=NF;i++) if($i != "0" && $i != "0/0") count++;
  print $1, $2, $3, $4, count
}'
```
Each line shows a variant site with:

- **CHROM**: `Av.cp.final` — the reference sequence name
- **POS**: Genomic position of the SNP
- **REF** / **ALT**: Reference and alternate alleles
- **COUNT**: Number of samples (out of 16) with the alternate allele

Example:
- `Av.cp.final 1349 G T 2` → 2 samples have T instead of the reference G

Step 5: Query variation in specific genes
* Use: Identify which genes contain the most variants.
* We need to connect the position of the SNPs to the annotation coordinates (`bedtools intersect` gene coordinates with SNP coordinates)

```bash
bedtools intersect -c -a chloe/Av.cp.final.chloe.gff -b Av.hiQ.vcf | grep "CDS" | sort -nk 10
```

* the intersect lifts out the annotations that match the the SNPs from the `.gff`
* `grep "CDS"` filters lines that correspond protein-coding regions in the genome

* `sort -nk 10` sorts the filtered output numerically (`-n`) by the 10th column which contains the count of intersecting variants


* if you want to filter out more 

```bash
bedtools intersect -c \
  -a chloe/Av.cp.final.chloe.gff \
  -b Av.hiQ.vcf | \
grep "CDS" | \
awk -F'\t' '{
  match($9, /gene=([^;]+)/, a);
  printf "%s\t%s\t%s\t%s\t%s\n", a[1], $4, $5, $7, $NF
}' | \
sort -k5,5n | \
column -t
```
**Output:** 

| Gene   | Start   | End     | Strand | SNPs |
|--------|---------|---------|--------|------|
| atpA   | 8921    | 10444   | -      | 1    |
| atpI   | 13145   | 13888   | -      | 1    |
| rpl20  | 66322   | 66702   | +      | 1    |
| rpoB   | 22171   | 25383   | -      | 1    |
| rpoC1  | 19377   | 20993   | -      | 1    |
| matK   | 402     | 1925    | -      | 2    |
| psaA   | 38139   | 40391   | -      | 5    |
| ycf1   | 108094  | 113754  | -      | 5    |



### Wrap-up of Comparison and Biological Interpretation (15 min)

1. **No Overlap with High-Quality Variants**:
   - The majority of the CDS regions have 0 SNPs => highly conserved

   - The samples are too closely related to have significant variation (same species)
   
   - Most SNPs may be located outside of coding regions.

2. **Few CDS Regions with Variants**:
   - SNPs in coding sequences may indicate regions with potential functional significance or evolutionary importance.

3. **Specific Genes with Variants**:
   - **`matK`** has 2 and **`ycf1`** has 5 variants
   - potential polymorphisms or evolutionary changes within these genes even within the same species
   - could be useful for identifying intraspecific populations

### Closing notes:
* Cleaning removes noise: Filters out contamination and poor-quality reads, reducing false SNPs.
* Filtering boosts confidence: Retains only high-quality variants (QUAL > 30, DP > 20).
* Interpretation improves: Reveals true biological variation and highlights outlier samples.

* Gene Conservation: Most CDS regions lack variants → essential, conserved functions.

* Functional Relevance: Genes with SNPs may affect function or physiology.

* Practical Use: Conserved regions suitable for primers, barcoding, species ID.




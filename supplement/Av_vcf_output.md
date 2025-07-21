### Interpreting `bcftools stats` PSC Output

The `PSC` lines in the `bcftools stats` output show **per-sample variant statistics**. These are especially useful for checking the consistency and quality of variant calls across samples.

Each line has the following columns:

| Column        | Description                                                   |
|---------------|---------------------------------------------------------------|
| PSC           | Line type: Per-Sample Count                                   |
| [2] id        | Always 0 (placeholder)                                        |
| [3] sample    | Path to the BAM file (sample ID)                              |
| [4] nRefHom   | Homozygous reference SNPs (not applicable to haploid cpDNA)   |
| [5] nNonRefHom| Homozygous variant SNPs (not applicable to haploid cpDNA)     |
| [6] nHets     | Heterozygous SNPs (not applicable to haploid cpDNA)           |
| [7] nTransitions | Number of transition SNPs (A↔G or C↔T)                     |
| [8] nTransversions | Number of transversion SNPs ( purine ↔ pyrimidine change)        |
| [9] nIndels   | Number of insertions/deletions                                |
| [10] Avg depth| Average sequencing depth across sites                         |
| [11] nSingletons | Variants unique to this sample                             |
| [12] nHapRef  | Haploid calls that match the reference allele                 |
| [13] nHapAlt  | Haploid calls that differ from the reference allele           |
| [14] nMissing | Sites with missing data (no confident call could be made - low quality or insufficient depth)                                       |

#### Rows are Example Observations

```text
PSC     0       .../10.sorted.bam   ...   378   73   0
PSC     0       .../16.sorted.bam   ...   187  240  24
```

#### Lets extract the data we need

```bash
bcftools stats -s - Av.vcf | grep "PSC" | awk '{print $3, $10, $12, $13, $14}'
```
---
### Column Descriptions

| Column          | Description                                                                  |
| --------------- | ---------------------------------------------------------------------------- |
| Sample BAM File | The file name for the sample (aligned and sorted).                           |
| Avg Depth       | Average read depth across variant sites. Higher values indicate better data. |
| nHapRef         | Number of haploid positions matching the reference.                          |
| nHapAlt         | Number of haploid positions with a variant (different from reference).       |
| nMissing        | Number of sites with no data (gaps or low coverage).                         |
| nIndels         | (Not shown here, but part of full output) – number of insertions/deletions.  |

---

### Simplified Per-Sample Variant Summary (Chloroplast DNA)

| Sample BAM File | Avg Depth | nHapRef | nHapAlt | nMissing | nIndels |
| --------------- | --------- | ------- | ------- | -------- | ------- |
| 10.sorted.bam   | 118.6     | 378     | 73      | 0        | –       |
| 11.sorted.bam   | 113.5     | 385     | 66      | 0        | –       |
| 12.sorted.bam   | 107.3     | 350     | 101     | 0        | –       |
| 13.sorted.bam   | 102.7     | 355     | 96      | 0        | –       |
| 14.sorted.bam   | 58.6      | 328     | 123     | 0        | –       |
| 15.sorted.bam   | 102.0     | 308     | 143     | 0        | –       |
| 16.sorted.bam   | 52.7      | 187     | 240     | 24       | –       |
| 1.sorted.bam    | 108.4     | 373     | 78      | 0        | –       |

### Notes

* **Sample 16** stands out: low depth, high number of alternate alleles, and non-zero missing data — possibly an outlier or low-quality sample.
* Others show consistent depth and a typical ratio of reference to alternate calls.
* **nIndels** are not shown here but can be included if needed using the full `bcftools stats` output.

---
### Simplified Per-Sample Variant Summary After Cleaning

| Sample BAM File | Avg Depth | nHapRef | nHapAlt | nMissing |
| --------------- | --------- | ------- | ------- | -------- |
| 10.sorted.bam   | 117.4     | 142     | 122     | 0        |
| 11.sorted.bam   | 113.3     | 127     | 137     | 0        |
| 12.sorted.bam   | 97.8      | 137     | 127     | 0        |
| 13.sorted.bam   | 102.8     | 170     | 94      | 0        |
| 14.sorted.bam   | 57.9      | 150     | 114     | 0        |
| 15.sorted.bam   | 100.4     | 187     | 77      | 0        |
| 16.sorted.bam   | 13.0      | 141     | 115     | 8        |
| 1.sorted.bam    | 99.0      | 147     | 117     | 0        |
| 2.sorted.bam    | 103.9     | 148     | 116     | 0        |
| 3.sorted.bam    | 110.3     | 196     | 68      | 0        |
| 4.sorted.bam    | 85.3      | 192     | 72      | 0        |
| 5.sorted.bam    | 118.2     | 139     | 125     | 0        |
| 6.sorted.bam    | 111.6     | 192     | 72      | 0        |
| 7.sorted.bam    | 107.1     | 171     | 93      | 0        |
| 8.sorted.bam    | 103.8     | 131     | 133     | 0        |
| 9.sorted.bam    | 56.2      | 180     | 84      | 0        |

---

### Notes

* **Sample 16** still has low depth and is the only sample with missing data, indicating persistent quality issues despite cleaning.
* Other samples show improved and more consistent depth, and the total variant counts are more balanced.
* Cleaning removed potential contamination and spurious variants, improving data quality for chloroplast analysis.


---
### High-Quality Variant Summary (Filtered: QUAL > 30, DP > 20)

| Sample | Avg Depth | Singletons | Alt Alleles | Missing |
|--------|-----------|------------|-------------|---------|
| 10     | 112.6     | 74         | 104         | 0       |
| 11     | 108.6     | 76         | 102         | 0       |
| 12     | 90.4      | 68         | 110         | 0       |
| 13     | 97.2      | 50         | 128         | 0       |
| 14     | 57.0      | 62         | 116         | 0       |
| 15     | 96.6      | 37         | 141         | 0       |
| 16     | 11.3      | 68         | 106         | 4       |
| 1      | 92.3      | 59         | 119         | 0       |
| 2      | 98.4      | 64         | 114         | 0       |
| 3      | 106.2     | 28         | 150         | 0       |
| 4      | 81.4      | 46         | 132         | 0       |
| 5      | 112.8     | 69         | 109         | 0       |
| 6      | 106.8     | 38         | 140         | 0       |
| 7      | 101.3     | 47         | 131         | 0       |
| 8      | 96.4      | 76         | 102         | 0       |
| 9      | 57.4      | 44         | 134         | 0       |


### Notes

* **Sample 16** remains an outlier — low depth and 4 missing genotypes — suggesting poor data quality.
* All other samples passed the quality filter (QUAL > 30, DP > 20) and show consistent variant profiles.
* High-quality filtering sharpens the focus on confident chloroplast variants, reducing noise and enhancing reliability for downstream analysis.

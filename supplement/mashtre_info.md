## Mashtree – Command-line Phylogeny Tool (Low-resource setup)

Mashtree builds neighbour-joining phylogenetic trees from genome assemblies or raw reads using Mash distances (MinHash). It works on low-resource servers (single CPU, no parallelism), though it may be slower for large datasets.

## Basic Usage (Single CPU)

```bash
mashtree --numcpus 1 *.fasta > tree.dnd
````

* Input: `.fasta`, `.fastq`, `.gbk`, `.embl` (gzipped accepted)
* Output: Newick-format tree (`.dnd`)
* `--numcpus 1`: restricts processing to a single thread (required on single-core machines)

## Parameter Summary (Low-resource Recommendations)

| Option          | Purpose                                   | Recommended Setting                    |
| --------------- | ----------------------------------------- | -------------------------------------- |
| `--kmerlength`  | K-mer size for sketching                  | `16` (smaller = faster, less specific) |
| `--sketch-size` | Number of hashes per genome (sketch size) | `10000` (lower memory, less accurate)  |
| `--mindepth`    | Min k-mer count to include in sketch      | `5` (default); avoid `0` on low RAM    |
| `--outtree`     | Write output tree to file                 | Use to avoid redirecting with `>`      |
| `--outmatrix`   | Write distance matrix to file             | Optional                               |

## Understanding k-mer and sketch size

* **K-mer size (`--kmerlength`)**: Controls the resolution of the genomic comparison.

  * Smaller values (e.g. 16) are more tolerant of sequencing errors and faster to compute but may reduce specificity.
  * Larger values (e.g. 21–32) improve resolution for closely related sequences but require more memory.

* **Sketch size (`--sketch-size`)**: Number of hashes kept per genome.

  * Low values (e.g. 10000) are faster and use less memory.
  * Higher values (e.g. 100000) increase accuracy and stability, especially for larger genomes or fine-scale comparisons.

Choose values based on your available memory and how accurate the resulting tree needs to be.

## Fast and Light Example

```bash
mashtree --numcpus 1 --kmerlength 16 --sketch-size 10000 *.fasta > tree_fast.dnd
```

## Avoid Accurate Mode on Low-resources

Avoid this unless using only a few genomes:

```bash
mashtree --mindepth 0 --sketch-size 100000 --numcpus 1 *.fasta > tree.dnd
```

## Confidence Estimation (Not Recommended with 1 CPU)

```bash
mashtree_bootstrap.pl --reps 100 --numcpus 1 ...
mashtree_jackknife.pl --reps 100 --numcpus 1 ...
```

These require multiple re-runs and are slow without parallelism.

## Additional output (optional):
--outmatrix: tab-delimited pairwise Mash distance matrix
--tempdir: if specified, retains intermediate Mash sketches and distance files


## Best Practices

* Use a smaller number of genomes on limited hardware.
* Use `--kmerlength 16` and `--sketch-size 10000` to reduce resource use.
* Rename output nodes and visualise trees using [iTOL](https://itol.embl.de)
* Always reroot the tree using a known outgroup for comparison.




## Reference

Katz et al. (2019). *Mashtree: a rapid comparison of whole genome sequence files*. JOSS, 4(44), 1762. [https://doi.org/10.21105/joss.01762](https://doi.org/10.21105/joss.01762)
GitHub: [https://github.com/lskatz/mashtree](https://github.com/lskatz/mashtree)
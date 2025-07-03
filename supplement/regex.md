# Understanding the Regular Expression in Block 2

In Block 2, we use a **regular expression** (regex) with the tool `sed` to extract clean gene names from column 9 of the GFF file.

The goal is to clean up the messy attributes field (column 9) so it only shows the **gene name**, e.g.:

```

original: ID=cds-XYZ123;gene=psbA;product=Photosystem II protein D1
result:   psbA

````

## Breakdown of the Regex

We use this pattern inside a `sed` command:

```bash
sed -E -i 's/[^\t]*;gene=([^;]*);.*/\1/' Av.geseq.txt
````

### Explanation:

1. `[^\t]*;`
   Match any characters **except tabs**, up to and including the first semicolon.

2. `gene=([^;]*);`
   Match the text `gene=`, then capture any characters **until the next semicolon**.
   This gives us the **gene name** (captured in `\1`).

3. `.*`
   Match and discard everything else on the line.

### The replacement:

`\1`
This keeps only the captured gene name.


## Example

```bash
head Av.geseq.txt
sed -E -i 's/[^\t]*;gene=([^;]*);.*/\1/' Av.geseq.txt
head Av.geseq.txt
```


Being able to extract and clean specific fields from large text files is a key skill in bioinformatics.
Regex and `sed` are powerful tools for this.

For more examples of regex:
🔗 [https://en.wikipedia.org/wiki/Regular\_expression](https://en.wikipedia.org/wiki/Regular_expression)
🔗 [https://www.geeksforgeeks.org/sed-command-in-linux-unix-with-examples/](https://www.geeksforgeeks.org/sed-command-in-linux-unix-with-examples/)

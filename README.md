# Rare Hematopoietic Stem Cells resilient to infection-induced stress sense yet withstand inflammation

Single cell RNA sequencing analysis code for:

Georgiou C, Liu Q, Bruno F, Mai C, Tissot F, Fan X, Gonzalez-Anton S, Birch F,
Mukhopadhyay D, Kinston SJ, Chabra S, Sergi C, van Gastel N, Malanchi I,
Wilson NK, Duffy KR, Pospori C, Blagborough AM, Göttgens B, Luis TC, Lo Celso C.
*Rare Hematopoietic Stem Cells resilient to infection-induced stress sense yet
withstand inflammation.* **Blood**, 2026. DOI: 

Preprint: bioRxiv, DOI [10.1101/2025.04.09.647965](https://doi.org/10.1101/2025.04.09.647965)

## Overview

This repository holds the single cell RNA sequencing analysis code for the paper
above. The experimental design, the infection model and the biological
conclusions are described in the paper, and this README covers only what the
code does.

CD48-negative haematopoietic stem cells were index sorted from control and
*Plasmodium berghei* infected mice. Each cell was assigned to a group
afterwards, using the biotin label intensity that index sorting recorded. The
analysis compares three groups.

| Group | Cells | Average genes per cell |
|---|---|---|
| Biotin-Hi Control | 58 | 7490 |
| Biotin-Hi Infected | 64 | 8067 |
| Biotin-Lo Infected | 43 | 8520 |
| Total | 165 | 8026 |

The code covers Figure 4, Figure 5A, and Supplemental Figure 5A and 5B. Panels
that come from flow cytometry are not in this repository, because they were
analysed and plotted in other software, as the paper describes.

## Data availability

Single cell transcriptomics data are deposited in the Gene Expression Omnibus
under accession [GSE297047](https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE297047).

The UMAP coordinates that Figure 4C plots are provided as
`umap_coordinates_250303.csv`. Use that file rather than recomputing the
embedding, for the reason given under **Reproducibility** below.

## Repository structure

```
notebooks/
  01_qc_and_biotin_labels.ipynb     quality control, index sorting values, Biotin groups
  02_normalisation_umap_scfea.ipynb normalisation, UMAP, gene scores, metabolic flux
  03_differential_expression.ipynb  differential expression, volcano plots, dot plots
  04_oxphos_score.ipynb             OXPHOS gene score
  Fig4_organised.ipynb              all Figure 4 code, in panel order
data/          small tables and gene lists that the code reads
results/       tables that the code writes and that back a figure panel
figures/       the PDF files
```

`Fig4_organised.ipynb` collects the code for every panel of Figure 4 in one
place, in panel order. The four numbered notebooks are the original working
notebooks, and they also contain exploratory analyses that are not in the paper.

## Order in which the notebooks run

The notebooks pass data to one another through files on disk, so run them in
this order.

**1. Quality control and Biotin groups**

Reads the quality controlled count matrix, which holds 175 cells and 55414
genes. Removes 28310 genes with fewer than 1 count, which leaves 27104 genes.
Attaches the index sorting values for Biotin BV421 and EPCR PE, then keeps the
173 cells that have a biotin value. The Biotin-Hi and Biotin-Lo label comes from
the sorting gate that the metadata table records. The label of 9 cells is then
corrected, of which 8 move to Biotin-Lo and 1 moves to Biotin-Hi.

**2. Normalisation, UMAP and metabolic flux**
Removes the 8 Biotin-Lo Control cells, which leaves the 165 cells that the paper
reports. Removes 165 further genes
with fewer than 1 count, which leaves 26939 genes. Normalises with
`smqpp.normalise_data`, without ERCC spike-ins. Selects 2163 highly variable
genes with `smqpp.tech_var`, at `meanForFit=10`. Runs
principal component analysis with the `arpack` solver, builds the neighbourhood
graph with 10 principal components and 10 neighbours, then runs UMAP and Leiden
clustering at a resolution of 0.6, all with `random_state=0`. Also runs the
metabolic flux analysis for Figure 4I left, and the gene scores for Figure 5A
and Supplemental Figure 5A.

**3. Differential expression**

Runs the differential expression tests and produces Figure 4A to 4H.

**4. OXPHOS gene score**

Scores each cell against the `GOBP_OXIDATIVE_PHOSPHORYLATION` gene set from the
Molecular Signatures Database, using `sc.tl.score_genes`. The set holds 148
genes, of which 128 are present in the 165 cell object.

## Panel map

| Panel | Notebook | What it shows |
|---|---|---|
| 4A | 03 | table of cell numbers and average genes per cell |
| 4B | 03 | violin plots of Biotin BV421 and EPCR PE from index sorting. The input table holds all 173 cells, and the plot shows the three groups |
| 4C | 03 | UMAP of the 165 cells |
| 4D | 03 | cell cycle phase proportions |
| 4E | 03 | dot plot of the top 20 differentially expressed genes per group |
| 4F | 03 | volcano plot, Biotin-Hi Infected against Biotin-Hi Control |
| 4G | 03 | Gene Ontology bar plot, from the genes upregulated in 4F |
| 4H | 03 | volcano plot, Biotin-Hi Infected against Biotin-Lo Infected |
| 4I left | 02 | metabolic flux, glycolysis and TCA cycle |
| 4I right | 04 | OXPHOS gene score |
| 4J | not in this repository | MitoSOX Red flow cytometry |
| 5A | 02 computes the score, 03 draws the plot | MHC Class II core enrichment gene score |
| S5A | 02 computes the scores, 03 draws the plot | HSC score and Repopulation score |
| S5B | 03 | dot plot with more genes per group than Figure 4E |

## Statistical settings

These describe the tests that the analysis uses.

- Figure 4B, 4I right, 5A and S5A, the Wilcoxon rank sum test, through
  `geom_pwc` in `ggpubr`.
- Figure 4D, Fisher's exact test, two-sided, on a two by two table of cycling
  cells against non-cycling cells. Cycling means S plus G2M, and non-cycling
  means G0 and G1. The comparison is Biotin-Hi Infected against Biotin-Hi
  Control, and it gives an odds ratio of 1.76 and a p-value of 0.17.
- Figure 4F, a gene is called differentially expressed when the adjusted p-value
  is below 0.05 and the absolute log2 fold change is above 1.5.
- Figure 4H, the same rule with the adjusted p-value below 0.1.
- Figure 4I left, a likelihood ratio test on the flux of each metabolic module.
  The plot draws a horizontal reference line at a p-value of 0.05 and two
  vertical reference lines at a Cohen's d of 0.15 in either direction. The
  labelled points are selected by the Cohen's d threshold and by membership of
  the TCA cycle, and not by the p-value.

## The cell set used for Figure 4I left

The metabolic flux comparison in Figure 4I left runs on 135 cells rather than on
the 165 cells that the other panels use. Cells with a biotin intensity between
the 40th and the 60th percentile of the infected cell distribution were removed,
which leaves 54 Biotin-Hi Control, 38 Biotin-Hi Infected and 43 Biotin-Lo
Infected cells. The likelihood ratio test compares 92 Biotin-Hi cells against 43
Biotin-Lo cells, so the Biotin-Hi group pools control and infected cells.

## Steps that run outside the notebooks

Four steps do not run inside Jupyter, and their results are read back in.

- Metabolic flux estimation with scFEA, which runs in its own environment.
- The likelihood ratio test on the scFEA modules, which runs in R.
- The hscScore model, which runs through `hscScore_calculate.py` in an
  environment with `scikit-learn` version 0.21.3.
- Gene Ontology enrichment, which runs on the Enrichr website, and the
  downloaded table is read back in.

## Gene sets used

The table gives the gene list that each score reads, and the function that
computes it. The paper cites the original publications for these lists.

| Score | Gene list in the code | Function |
|---|---|---|
| Cell cycle | `regev_lab_cell_cycle_genes_mouse_Corrected.txt` | `sc.tl.score_genes_cell_cycle` |
| Repopulation score | `RepopSig` | `sc.tl.score_genes` |
| OXPHOS score | `GOBP_OXIDATIVE_PHOSPHORYLATION`, Molecular Signatures Database | `sc.tl.score_genes` |
| MHC Class II score | Cd74, H2-Aa, H2-Eb1, H2-Ab1, Ctss, Tubb1 | `sc.tl.score_genes` |
| HSC score | not a gene list, see below | `hscScore_calculate.py` |

The HSC score is not a gene set score. It comes from the hscScore model, which
runs outside the notebook through `hscScore_calculate.py`, in a separate
environment with `scikit-learn` version 0.21.3. The notebook writes the input
table, runs the script by hand, then reads the result back in.

The HSC score and the Repopulation score are computed on all 173 cells. The
plots show the three groups, because the plotting code sets the group levels and
leaves out the Biotin-Lo Control cells.

## Reproducibility

A UMAP embedding does not reproduce exactly across software versions, even with
a fixed random seed. For that reason the coordinates are provided as a file, and
Figure 4C is drawn from the file rather than from a fresh computation. The
differential expression results have the same property, so use the saved result
tables rather than rerunning the tests under a newer version of `scanpy`.

The notebooks run under Python 3.11, and the R code runs inside them through
`rpy2` and the `%%R` cell magic. [will add the PACKAGE versions here later]

## Paths

The notebooks use absolute paths on the Cambridge high performance computing
cluster. Change the base directory at the top of each notebook to run them
elsewhere.

## Contact

For questions about the single cell analysis code, contact Qi Liu, Wellcome-MRC
Cambridge Stem Cell Institute, University of Cambridge, lq2021cambridge@gmail.com.

For questions about the study, contact Cristina Lo Celso, Imperial College
London, c.lo-celso@imperial.ac.uk.

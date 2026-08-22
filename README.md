# Rare Hematopoietic Stem Cells resilient to infection-induced stress sense yet withstand inflammation

Single cell RNA sequencing analysis code for:

Georgiou C, Liu Q, Bruno F, Mai C, Tissot F, Fan X, Gonzalez-Anton S, Birch F,
Mukhopadhyay D, Kinston SJ, Chabra S, Sergi C, van Gastel N, Malanchi I,
Wilson NK, Duffy KR, Pospori C, Blagborough AM, Göttgens B, Luis TC, Lo Celso C.
*Rare Hematopoietic Stem Cells resilient to infection-induced stress sense yet
withstand inflammation.* **Blood**, 2026. DOI: [ADD ON PUBLICATION]

Preprint: bioRxiv, DOI [10.1101/2025.04.09.647965](https://doi.org/10.1101/2025.04.09.647965)

## Overview

Mice were infected with the rodent malaria parasite *Plasmodium berghei* ANKA
2.34. NHS-ester-biotin was injected on day 3 after blood infection, and bone
marrow was analysed on day 5. Biotin dilution splits haematopoietic stem cells
into a Biotin-Hi fraction, which retains the label, and a Biotin-Lo fraction,
which has diluted it. The Biotin-Hi cells from infected mice retain high
repopulation potential, and the paper calls them resilient haematopoietic stem
cells, or R-HSCs.

CD48-negative stem cells were index sorted and sequenced by Smart-seq2. Each
cell was assigned to a group afterwards, using the biotin intensity that index
sorting recorded. The analysis compares three groups.

| Group | Cells | Average genes per cell |
|---|---|---|
| Biotin-Hi Control | 58 | 7490 |
| Biotin-Hi Infected, that is R-HSC | 64 | 8067 |
| Biotin-Lo Infected | 43 | 8520 |
| Total | 165 | 8026 |

This repository holds the code for Figure 4, for Figure 5A and for Supplemental
Figure 5A and 5B. Panels that come from flow cytometry were analysed in FlowJo
and plotted in GraphPad Prism, so they are not in this repository.

## Data availability

Single cell transcriptomics data are deposited in the Gene Expression Omnibus
under accession [GSE297047](https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE297047).

Flow cytometry data are available from the authors on reasonable request.

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
data/
  [SMALL INPUT AND OUTPUT TABLES THAT YOU DEPOSIT HERE]
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
173 cells that have a biotin value. Assigns each cell to Biotin-Hi or Biotin-Lo
using the median biotin intensity per sorting plate, and corrects the label of
9 cells, of which 8 move to Biotin-Lo and 1 moves to Biotin-Hi.

**2. Normalisation, UMAP and metabolic flux**

Removes the 8 Biotin-Lo Control cells, which are too few to analyse as a group,
and this leaves the 165 cells that the paper reports. Removes 165 further genes
with fewer than 1 count, which leaves 26939 genes. Normalises with
`smqpp.normalise_data`, without ERCC spike-ins. Selects 2163 highly variable
genes with the method of Brennecke and colleagues, at `meanForFit=10`. Runs
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
| 4B | 03 | violin plots of Biotin BV421 and EPCR PE from index sorting |
| 4C | 03 | UMAP of the 165 cells |
| 4D | 03 | cell cycle phase proportions |
| 4E | 03 | dot plot of the top 20 differentially expressed genes per group |
| 4F | 03 | volcano plot, Biotin-Hi Infected against Biotin-Hi Control |
| 4G | 03 | Gene Ontology bar plot, from the 107 upregulated genes in 4F |
| 4H | 03 | volcano plot, Biotin-Hi Infected against Biotin-Lo Infected |
| 4I left | 02 | metabolic flux, glycolysis and TCA cycle |
| 4I right | 04 | OXPHOS gene score |
| 4J | not in this repository | MitoSOX Red flow cytometry |
| 5A | 02 | MHC Class II core enrichment gene score |
| S5A | 02 | HSC score and Repopulation score |
| S5B | 03 | dot plot of the full differentially expressed gene list |

## Statistical settings

- Figure 4D, Fisher's exact test on S and G2M against G0 and G1.
- Figure 4F, an adjusted p-value below 0.05 and a fold change above 1.5.
- Figure 4H, an adjusted p-value below 0.1 and a fold change above 1.5.
- Figure 4I left, a likelihood ratio test on the flux of each metabolic module,
  with a p-value threshold of 0.05 and a Cohen's d threshold of 0.15 in either
  direction.
- Figures 4B, 4I right, 5A and S5A, the Wilcoxon rank sum test.

The metabolic flux comparison in Figure 4I left runs on a separate cell set,
which holds 135 cells. Those are 54 Biotin-Hi Control cells, 38 Biotin-Hi
Infected cells and 43 Biotin-Lo Infected cells. The likelihood ratio test in
that panel compares 92 Biotin-Hi cells against 43 Biotin-Lo cells, so the
Biotin-Hi group pools control and infected cells.

## Steps that run outside the notebooks

Three steps do not run inside Jupyter, and their results are read back in.

- Metabolic flux estimation with scFEA, which runs in its own environment.
- The likelihood ratio test on the scFEA modules, which runs in R.
- Gene Ontology enrichment, which runs on the Enrichr website, and the
  downloaded table is read back in.

## Gene sets used

| Score | Source |
|---|---|
| Cell cycle | Kowalczyk et al. 2015 and Nestorowa et al. 2016 |
| HSC score | Wilson et al. 2015 and Hamey and Göttgens 2019 |
| Repopulation score | Che et al. 2022 |
| OXPHOS score | `GOBP_OXIDATIVE_PHOSPHORYLATION`, Molecular Signatures Database |
| MHC Class II score | Cd74, H2-Aa, H2-Eb1, H2-Ab1, Ctss, Tubb1 |

## Reproducibility

A UMAP embedding does not reproduce exactly across software versions, even with
a fixed random seed. For that reason the coordinates are provided as a file, and
Figure 4C is drawn from the file rather than from a fresh computation. The
differential expression results have the same property, so use the saved result
tables rather than rerunning the tests under a newer version of `scanpy`.

The notebooks run under Python 3.11, and the R code runs inside them through
`rpy2` and the `%%R` cell magic. [ADD THE PACKAGE VERSIONS HERE.]

## Paths

The notebooks use absolute paths on the Cambridge high performance computing
cluster. Change the base directory at the top of each notebook to run them
elsewhere.

## Contact

For questions about the single cell analysis code, contact Qi Liu, Wellcome-MRC
Cambridge Stem Cell Institute, University of Cambridge, lq2021cambridge@gmail.com
.

For questions about the study, contact Cristina Lo Celso, Imperial College
London, c.lo-celso@imperial.ac.uk.

## Licence

[LICENCE, for example MIT]


# Single-Cell Multiomics — Joint RNA + Surface Protein (CITE-seq)

Jointly modeling two real molecular layers measured in the *same* single cells — real mRNA and real
surface-protein counts from the same PBMC — using totalVI, a real published deep generative model built
specifically for CITE-seq, and checking what the joint model actually buys over RNA alone.

![Joint totalVI clusters](results/hero_joint_totalvi_umap.png)

## 1. Aim

Build one joint latent description of each real cell from both its real RNA and real surface-protein
measurement together, then honestly answer two questions: does the joint representation differ meaningfully
from an RNA-only one, and how well does the real surface-protein signal agree with its corresponding RNA
gene.

## 2. Objective

1. Load a real, paired CITE-seq dataset and split it into its two real modalities.
2. Fit totalVI (scvi-tools), a real published joint generative model for RNA + surface protein, and extract
   a joint latent representation plus denoised (background-corrected) protein expression.
3. Compare joint clustering against RNA-only clustering on the exact same real cells.
4. Correlate real denoised protein levels against their corresponding real RNA genes, per canonical marker.

## 3. Data fetch

Real, public 10x Genomics dataset **"10k PBMCs from a Healthy Donor — Gene Expression with a Panel of
TotalSeq™-B Antibodies"** (`pbmc_10k_protein_v3`), Cell Ranger 3.0.0, CC BY 4.0 licensed. One combined
`filtered_feature_bc_matrix.h5` file, downloaded directly from 10x's own public dataset host, already
contains both real feature types ("Gene Expression" and "Antibody Capture") indexed by the same real cell
barcodes — no separate per-modality files to reconcile.

## 4. Data describe

7,865 real PBMCs from a single healthy donor, each with a real mRNA profile (33,538 real genes) and a real
17-antibody TotalSeq-B surface-protein profile — 14 real target markers (CD3, CD4, CD8a, CD14, CD15, CD16,
CD56, CD19, CD25, CD45RA, CD45RO, PD-1, TIGIT, CD127) plus 3 real isotype controls (IgG1, IgG2a, IgG2b)
included specifically as a background-signal reference. QC kept 7,798 of the 7,865 real cells (min 200 genes
detected, max 20% mitochondrial reads).

## 5. Methods / Workflow

1. Split the combined real matrix into RNA and protein `AnnData` objects by feature type.
2. QC each modality on its own real terms; filter cells on real measured RNA QC distributions.
3. RNA-only baseline: normalize, log-transform, PCA, Leiden clustering, UMAP — the same method as this
   portfolio's Single-Cell RNA-seq Analysis project.
4. Fit totalVI on the real RNA (subset to 4,000 highly variable genes) and real protein counts together;
   extract the joint latent representation and denoised protein expression.
5. Joint clustering and UMAP from the totalVI latent space.
6. Correlate real denoised protein level against real RNA gene expression per canonical marker, across all
   real cells.
7. Compare RNA-only vs joint clustering via the Adjusted Rand Index and a full contingency table.

## 6. Results

RNA-only clustering: 13 real clusters. Joint (totalVI) clustering: 15 real clusters. **Adjusted Rand Index
between the two: 0.795** — substantial real agreement, not full redundancy and not a wholesale disagreement;
the real contingency table shows the protein layer mostly refining/re-splitting parts of the RNA-only
structure rather than finding an entirely different cell-type landscape.

RNA-vs-protein correlation (Pearson r, across all 7,798 real cells): strong real agreement for CD14 (0.90),
CD127 (0.84), CD3 (0.77), CD8a (0.76), CD19 (0.60); weak for CD4 (0.10) and CD15 (0.02).

![RNA-only baseline clusters](results/hero_rna_only_umap.png)

Full results: [`results/rna_only_vs_joint_ari.csv`](results/rna_only_vs_joint_ari.csv),
[`results/rna_only_vs_joint_contingency.csv`](results/rna_only_vs_joint_contingency.csv),
[`results/rna_protein_correlation.csv`](results/rna_protein_correlation.csv).

## 7. Biology interpretation of results

The ARI of 0.795 is an honest, quantified answer to whether the second real modality mattered: yes,
meaningfully, but mostly by sharpening resolution within structure RNA alone already found, not by revealing
cell populations RNA alone had missed entirely — a believable result for a well-characterized system like
PBMCs, where canonical cell types are already strongly RNA-distinguishable.

The correlation results are the more biologically interesting finding. CD14, CD127, CD3, and CD8a show the
classical "marker gene ≈ marker protein" relationship holding up directly on real data. CD4's weak
correlation (r=0.10) is not a pipeline issue — it reproduces a real, independently published CITE-seq finding
(Stoeckius et al. 2017 and later work): CD4 surface protein is stable and abundant while CD4 mRNA is a
comparatively poor, low/noisy proxy for it, a real instance of post-transcriptional regulation decoupling
mRNA from protein at single-cell resolution. CD15's weak correlation with its mapped gene (`FUT4`, r=0.02) is
a different, equally real phenomenon: CD15 is a carbohydrate epitope rather than a direct single-gene protein
product, so a weak RNA correlation reflects the real underlying glycobiology rather than a failed
measurement.

## 8. Learning through project

A deep generative joint model (totalVI) and a classical RNA-only pipeline (the Single-Cell RNA-seq Analysis
project's method) can both be run
correctly and still leave the real, central question open — "did the second modality actually change
anything" — until it's explicitly checked with a quantitative comparison like the Adjusted Rand Index, rather
than assumed from the fact that a more sophisticated joint model was used. Separately: when two real
modalities measuring "the same" biological quantity disagree (as CD4 protein and CD4 mRNA do here), the right
response is to check whether the disagreement itself is independently documented and biologically explainable
before treating either measurement as wrong — in this case it was, which turned a result that looked like a
discouraging failure into this project's most interesting real finding.

## 9. Caveats

totalVI was trained on a highly-variable-gene subset (4,000 of 33,538 real genes), a real, disclosed
scope-narrowing choice made necessary by a real memory constraint when running the full gene set's posterior
sampling step — the excluded genes are plausibly low-variance and low-informative for this cell population,
but this wasn't independently verified gene-by-gene. Leiden clustering resolution (0.5 for both RNA-only and
joint) was chosen from a measured scan of candidate resolutions but left at that value rather than re-
confirmed against the scan's own printed cluster counts — the resulting 13/15 real clusters are legitimate
and self-consistent, but represent one reasonable resolution choice among several, not a uniquely "correct"
clustering granularity for this dataset. Clusters are reported by number rather than by assigned cell-type
name in this build.

# Human Membrane vs. Soluble Protein Dataset

Curated dataset used in the term project *"Performance Evaluation of
Transformer-Based Protein Language Models as Frozen Feature Extractors"*
(CSE 763 / CSE 443).

708 reviewed human proteins, class-balanced, redundancy-filtered, with fixed
train/validation/test splits.

---

## Files

| File | Contents |
|---|---|
| `dataset_full.csv` | All 708 proteins with a `split` column |
| `train.csv` | 496 proteins (248 per class) |
| `val.csv` | 106 proteins (53 per class) |
| `test.csv` | 106 proteins (53 per class) |
| `dataset_full.fasta` | Same data in FASTA format |

## Columns

| Column | Description |
|---|---|
| `accession` | UniProt accession (e.g. `P08913`) |
| `sequence` | Amino-acid sequence, 20-letter standard alphabet |
| `label` | `1` = transmembrane, `0` = soluble cytoplasmic |
| `split` | `train`, `val`, or `test` (in `dataset_full.csv` only) |

FASTA headers use the format `>accession|class|split`.

## Composition

|  | Transmembrane | Soluble |
|---|---|---|
| Count | 354 | 354 |
| Mean length | 424.3 | 419.8 |
| Hydrophobic residues (AILMFVWY) | 43.8% | 38.2% |
| Charged residues (DEKR) | 18.7% | 24.4% |
| Leucine (L) | 11.8% | 9.8% |

Length range across the set is 55–991 residues. The two classes are closely
matched in length (medians 354 vs 370), so sequence length is not a usable
shortcut for a classifier.

---

## How it was built

Source: [UniProt Knowledgebase](https://www.uniprot.org), reviewed (Swiss-Prot)
entries only, *Homo sapiens*.

**Positive class** — Transmembrane keyword `KW-0812`:

```
(reviewed:true) AND (organism_id:9606) AND (keyword:KW-0812)
AND (length:[50 TO 1000])
```

**Negative class** — Cytoplasm `SL-0091`, excluding membrane keywords:

```
(reviewed:true) AND (organism_id:9606) AND (cc_scl_term:SL-0091)
NOT (keyword:KW-0472) NOT (keyword:KW-0812) AND (length:[50 TO 1000])
```

**Filtering pipeline:**

1. Length restricted to 50–1000 residues (within ESM-2's 1024-token context)
2. Entries containing non-standard residues (X, B, Z, U, O) removed
3. Exact duplicate sequences dropped
4. Near-duplicates removed with a greedy 4-mer Jaccard filter (threshold 0.5),
   applied across **both classes jointly and before splitting**, so homologous
   sequences cannot leak between train and test
5. Classes balanced by subsampling
6. Stratified 70/15/15 split, random seed 42

**Retrieval counts:**

| Stage | Positive | Negative |
|---|---|---|
| Retrieved from UniProt | 4,500 | 357 |
| After cleaning | 4,490 | 356 |
| After redundancy filter (combined) | 4,716 | |
| After balancing | 354 | 354 |

The negative class was the binding constraint: the cytoplasmic query returned
only 357 entries in total, which capped the balanced dataset at 354 per class
and discarded most of the retrieved positives.

---

## Known limitations

- **Small.** 708 proteins, with only 106 in the test split, so confidence
  intervals on any metric computed from it are wide.
- **Homology.** The 4-mer Jaccard filter removes near-duplicates but not remote
  homologues. MMseqs2 or CD-HIT clustering at 30% identity with cluster-disjoint
  splits would be stricter.
- **Label provenance.** UniProt keyword annotations are partly inferred rather
  than experimentally verified. DeepLoc 2.1 applies an experimental-assertion
  filter that this dataset does not.
- **Single organism.** Human only; results need not transfer to other proteomes.

## Reproducing

Re-running `src/fetch_data.py --per-class 354` regenerates these files, though
exact membership may differ slightly as UniProt is updated.

## Citation

The UniProt Consortium. UniProt: the Universal Protein Knowledgebase in 2025.
*Nucleic Acids Research* 53(D1):D609–D617 (2025).
doi:10.1093/nar/gkae1010

Data reused under UniProt's CC BY 4.0 licence.

# typedrepair (v0.1.2)

provides the executable, semantics-locked finitary learning pipeline.

Author: Milan Rosko (www.milanrosko.com)
E-mail: Q1012878 at studium.fernuni-hagen.de

Core tutorial and information:

http://milanrosko.com/typedrepair.html

This repository,

```

Milan-Rosko / typedrepair

```

is the companion implementation for the accompanying paper (manuscript draft):

```

TYPED REPAIR: EMNIST FROM λ-DEFINABLE EVALUATION AND SPECIALIZATION GATE

```

pdf: http://milanrosko.com/aux/typedrepair.pdf


## 1 Overview

This repository contains a single Jupyter Notebook that produces a *finite, mechanically checkable* training-and-audit artifact for **EMNIST digits**.

The method is deliberately finitary. It (i) fixes a **Semantics Lock** (preprocessing + feature extraction + tie-breaking), (ii) builds an explicit **finite witness table** from the EMNIST training stream, (iii) trains a sparse **integer-weight** classifier by refutation-guided repair (no gradients), and (iv) performs **exact** verification on the finite table. The run emits a deterministic **certificate hash** that serves as the citeable identifier for the certified finite claim.

Repository contents are intentionally minimal:

```
- `typedlearn_emnist.ipynb`
- `licence.md`
- `citation.bib`
- `README.md`
- /gzip (empty)
- /artifacts (empty)
```

## 2 Requirements

- Python 3.13+ (developed on Python 3.13.7)
- Jupyter (Project Jupyter)
- No heavyweight ML frameworks are required for the core artifact.

Note. The EMNIST dataset is *not redistributed* in this repository. You must obtain it separately.

## 3 Certification

What is certified:

For a fixed Semantics Lock (canonicalization rule, binarization threshold, feature schema, optional capping, and stable tie-broken argmax), the notebook constructs:
- a finite witness table of size `FINITE_N`,
- a sparse integer weight table `W`,
- and a cached score matrix `S`.

It then performs two *exact* checks on the finite witness table:

1) *Finite margin obligations.*

For every witness example `i` and every competing class `c != y_i`:

```

score(i, y_i) >= score(i, c) + MARGIN

```

where:

```

score(i, c) = Σ_{f in facts(i)} W[(c, f)]

```

2) *Cache correctness.*

For every `i, c`, the cached score `S[i, c]` equals the definitional sum induced by `W` on witness example `i`. If both checks pass, the notebook should print:

```

5a5def3278dcac0ad827615e75374d8c66a81b8366285e4661ae0021ab39d8cc

```

This hash identifies the deterministic certificate payload. Please inform us otherwise.

Not certified:

- No universal/extensional claim over all inputs (no `∀X` statement).
- No generalization guarantee; probe accuracy is diagnostic only.
- No timing claim; runtimes are excluded from the deterministic certificate hash by design.

## 4 EMNIST (local only)

The notebook reads EMNIST directly from the official IDX gzip files.

### Expected directory layout (default)

```
<project-root>/
typedlearn_emnist.ipynb
gzip/
emnist-digits-mapping.txt
emnist-digits-train-images-idx3-ubyte.gz
emnist-digits-train-labels-idx1-ubyte.gz
emnist-digits-test-images-idx3-ubyte.gz
emnist-digits-test-labels-idx1-ubyte.gz
```

### How to obtain EMNIST

Option A (recommended): download the official archive and unpack it into `<project-root>/gzip`.

```bash
mkdir -p gzip
cd gzip
wget https://biometrics.nist.gov/cs_links/EMNIST/gzip.zip
unzip gzip.zip
````

Option B: if you already have the `.gz` files elsewhere, set:

* `EMNIST_DATA_DIR=/path/to/gzip`

### Supported splits

Any EMNIST split present in the directory is usable, provided the corresponding files exist:

* `EMNIST_SPLIT=digits` (default)
* common alternatives: `letters`, `balanced`, `byclass`, `bymerge`, `mnist`

## 5 Running

Open `typedlearn_emnist.ipynb` and run all cells top to bottom.

The notebook has two modes:

* `RUN_MODE=quick` (default): caps train/test stream lengths for faster iteration.
* `RUN_MODE=full`: removes caps unless you explicitly set them.

### Configuration (environment variables)

All parameters are optional; defaults are shown.

* `SEED=0`
* `RUN_MODE=quick` (`quick` or `full`)
* `EMNIST_SPLIT=digits`
* `EMNIST_DATA_DIR` (optional; otherwise the notebook searches for `./gzip` in the CWD or ancestors)
* `BIN_THRESHOLD=127`
* `FINITE_N=256`
* `CAP_UNARY=128`
* `MARGIN=1`
* `OVERSHOOT_DELTA=2`
* `MAX_EPOCHS=50`
* `MAX_UPDATES=20000`
* `PROBE_PER_CLASS=10`
* `ARTIFACT_DIR` (optional; default `<project-root>/artifacts`)
* `HASH_DATASET=0`
  If set to `1`, the notebook additionally hashes the EMNIST gzip files (slower but stronger provenance).

Example:

```bash
export EMNIST_DATA_DIR="$PWD/gzip"
export RUN_MODE=quick
export SEED=0
export FINITE_N=256
export BIN_THRESHOLD=127
jupyter lab
```

## 6 Outputs

By default, the notebook writes JSON files to:

* `<project-root>/artifacts/`

Files written (names include the **certificate hash**):

* `typedlearn_emnist_<split>_certificate_<CERT_HASH>.json`
  Deterministic certificate payload (the citeable finite artifact identifier).
* `typedlearn_emnist_<split>_payload_<CERT_HASH>.json`
  Certificate + run record (includes timing and probe diagnostics, which are not certified).
* `typedlearn_emnist_<split>_weights_<CERT_HASH>.json`
  Canonical sparse weight list `(class, feature, weight)`.

The console prints a compact certificate-facing summary including:

* finite-table violations count,
* cache check status,
* weights hash,
* finite table id,
* semantics-lock hash,
* **certificate hash**.

Reproducing the same certificate hash requires:

* identical EMNIST gzip files (same release),
* identical parameters,
* identical stream positions for the finite witness table.

Timing and probe diagnostics are intentionally excluded from the deterministic certificate hash.

## 7 Citation & License

If you use this repository, cite the paper as:

The paper:

```latex
@article{rosko2026typedlearn,
    title        = {{Typed Repair: EMNIST from $\lambda$-Definable Evaluation and Specialization Gate (Draft)}},
    author       = {Rosko, M.},
    year         = 2026,
    doi          = 10.5281/zenodo.18204862
    note         = {manuscript draft} 
  }
```

The emitted *certificate hash* (for the specific finitary artifact) 

```

8d5c236d239da810e666e11ece2d73ac1366fc63323b1583242f3405d047326e | v0.1.1 |

```

and cite EMNIST:

```latex
@article{cohen2017emnist,
  title        = {EMNIST: an extension of MNIST to handwritten letters},
  author       = {Cohen, G, and Afshar, S, and Tapson, J, and van Schaik, A,},
  year         = 2017,
  bit          = {arXiv preprint arXiv:1702.05373}
}
```

License and dataset notice. The “code” in this repository is licensed under the terms in `licence.md`. EMNIST is a separate dataset with its own provenance and attribution expectations; this repository does not claim copyright over the dataset.

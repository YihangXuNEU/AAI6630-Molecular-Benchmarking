# Molecular Image Recognition Benchmarking

**Author:** Yihang (Kevin) Xu  
**Course:** AAI 6630 — Computer Vision  
**Assignment:** 1.2 Individual Benchmarking Code

## Project overview

This project evaluates the pretrained MolGlyph optical chemical structure recognition (OCSR) component from BioMiner. It measures how accurately molecular images are converted into SMILES using a fixed BioVista OCSR subset and a small set of molecular drawings cropped from the course XN dataset.

The notebook implements the complete evaluation workflow: data loading and reference auditing → fixed sample preparation → model inference → official caption decoding → chemical evaluation → CSV/JSON export and visualization. It also includes an auxiliary OpenCV page-processing experiment and visual comparisons of unsuccessful predictions.

This is a component-level recognition benchmark. It does not train a new model or evaluate the full BioMiner protein–ligand–bioactivity extraction system. BioVista supplies the public evaluation data; it is not a second recognition model in this experiment.

## Repository contents

| File or directory | Purpose |
| --- | --- |
| `Xu_1_2_Benchmarking_Code.ipynb` | Main implementation, saved execution outputs, figures, and error comparisons. |
| `README.md` | Project scope, setup instructions, evaluation rules, and repository navigation. |
| `requirements.txt` | Python dependencies matching the recorded experiment environment. |
| `results/` | Exported files from the final 100-sample BioVista experiment, described below. |

The notebook can be reviewed with its saved outputs. The instructions below describe how to reproduce the experiment locally. Input archives, third-party source code, and model weights are acquired separately.

## Evaluation data and sample selection

### BioVista OCSR

The downloaded OCSR archive contains 8,861 image/SMILES annotations. In the recorded audit, 4,017 references contain wildcard atoms and one reference cannot be parsed by RDKit, leaving 4,843 eligible records.

Eligibility is defined before inference: reference SMILES must be RDKit-parseable and contain no wildcard atoms. The eligible pool is sorted by sample ID, sampled without replacement with `PUBLIC_RANDOM_SEED = 42`, and sorted again for evaluation. The final experiment uses `PUBLIC_SAMPLE_SIZE = 100`.

The notebook reads `ocsr/labels/gd_SMIELS.csv` directly from the archive. Image filenames are matched using `pdf_name`, the page prefix of `molminer_id`, and the separate `index` column. Every annotation-to-image mapping is checked before selection. Only selected images are extracted; their original contents are preserved.

### Course XN Dataset 2

The course ZIP contains six PDFs and 222 reference records. Recognition is evaluated on four predefined crops from paper 2, PDF page 2, Figure 1C, rendered at 300 DPI:

- **Exploratory samples:** AKI-062a, D4476, and FP1-24.
- **Previously evaluated pilot:** Compound 1h.

The manifest records each sample's source document, page, compound label, fractional crop coordinates, and reference SMILES. Crops were manually specified before the batch experiment. Their coordinates are executed automatically during a run; these crops are not outputs of the OpenCV region-proposal experiment.

Reference SMILES are used for scoring and subsequent visual inspection only. They are never passed to the recognition model or used to repair its predictions.

## Setup

### 1. Prepare the project directory

Download this repository and open a terminal in its root directory. Use the notebook filename shown above; if the downloaded filename includes a suffix such as `(1)`, rename the local copy accordingly.

### 2. Obtain inputs and third-party files

1. Download **Dataset 2** from the course materials. Keep the archive intact and place it in the project root as `Dataset 2.zip`.
2. Download the [BioVista OCSR archive](https://drive.google.com/file/d/11iCH0j7U9iFgwXahjkWZouNyqATfEPoL/view) and place it in the project root as `ocsr.tar.gz`. The same download is listed under OCSR in the [BioMiner dataset documentation](https://github.com/jiaxianyan/BioMiner#statistics-and-access-of-biovista).
3. Download the [BioMiner source repository](https://github.com/jiaxianyan/BioMiner) using **Code → Download ZIP**, then extract it as `BioMiner-main` in the project root.
4. Download the MolGlyph checkpoint from the [official model repository](https://huggingface.co/jiaxianustc/MolGlyph) and save it as `BioMiner-main/scripts/molglyph_large.pt`. Model access may require signing in and accepting the provider's conditions.
5. Retain `BioMiner-main/BioMiner/MolScribe/molscribe/`, `BioMiner-main/BioMiner/commons/molglyph_utils.py`, and the adjacent `abbrevs.csv` from the source download.

Before execution, the project root must contain the notebook, both archives, and `BioMiner-main`. The notebook creates `BioVista_OCSR/` and `outputs/` automatically. The course ZIP requires access to the course materials and is not publicly redistributed here.

### 3. Install dependencies and launch Jupyter

The recorded run used Windows, Python **3.12.3**, and CPU inference. A CUDA-capable GPU is not required by the notebook's configured execution path.

Create an isolated environment and install the repository dependencies:

```bash
python -m venv .venv
```

Activate it on Windows:

```powershell
.\.venv\Scripts\Activate.ps1
```

Or on macOS/Linux:

```bash
source .venv/bin/activate
```

Install dependencies and start the notebook interface:

```bash
python -m pip install -r requirements.txt
python -m pip install notebook
python -m notebook
```

Select the Python interpreter from this environment as the notebook kernel. The versions below document the successful submitted environment; a clean environment installation has not been independently verified.

| Package | Recorded version |
| --- | --- |
| numpy | 1.26.4 |
| opencv-python-headless | 4.11.0.86 |
| albumentations | 2.0.5 |
| SmilesPE | 0.0.3 |
| timm | 1.0.13 |
| OpenNMT-py | 2.2.0 |
| torch | 2.7.1 |
| torchvision | 0.22.1 |
| torchtext | 0.5.0 |
| pandas | 2.2.2 |
| matplotlib | 3.8.4 |
| PyMuPDF | 1.28.2 |
| rdkit | 2026.9.1 |
| tqdm | 4.66.4 |

## Running the experiment

1. Launch Jupyter from the project root and open `Xu_1_2_Benchmarking_Code.ipynb`.
2. Review the first code cell's configuration. The final public subset settings are:

   ```python
   PUBLIC_SAMPLE_SIZE = 100
   PUBLIC_RANDOM_SEED = 42
   ```

3. Restart the kernel and run all cells in order. Once inputs and dependencies are available, the notebook performs the workflow without interactive sample selection or manual edits between stages.
4. Inspect the dataset-specific summary and recognition-outcome figure. The visual inspection cell compares unsuccessful cases with the original image and reference structure; it does not change benchmark scores.
5. Save the executed notebook. Copy the final run's result files into `results/` when updating the submission.

All configured input and output paths are project-relative. The notebook freezes the combined 104-sample manifest before inference. It rejects changes to an existing manifest rather than silently replacing it.

Benchmark runs are written to:

```text
outputs/benchmark_n100_seed42/run_<UTC timestamp>/
```

Changing the public subset size or seed creates a different benchmark directory. If crop coordinates or reference inputs change while size and seed stay the same, assign a new `BENCHMARK_DIR` before running the experiment. The timestamped run folder preserves separate prediction executions. Input archives, source files, and the checkpoint should be held constant for comparable reruns.

## Metrics and output processing

| Metric | Definition |
| --- | --- |
| Exact-match accuracy | Fraction of all selected samples whose predicted and reference RDKit canonical isomeric SMILES are identical. Stereochemistry and disconnected fragments are retained. |
| Complete valid-output rate | Fraction of all selected samples yielding parseable reconstructed SMILES without unresolved dummy atoms. |
| Valid-only mean Tanimoto | Mean similarity over complete, valid predictions using Morgan fingerprints: radius 2, 2,048 bits, chirality enabled. |
| Failure-penalized mean Tanimoto | Mean over all selected samples, assigning zero to failed, empty, invalid, or unresolved predictions. |
| Inference time | Elapsed time for the image-readability check and model call, including image loading. It excludes model initialization, subsequent official decoding, metric calculation, and figure generation. |

Raw model captions are preserved. BioMiner's official `Translator.refactor` decoder reconstructs captions containing separator and abbreviation annotations. Remaining dummy atoms are classified as unresolved groups. The interface used does not supply confidence values.

All selected samples remain in the evaluation denominator. Failure categories include inference exceptions, empty outputs, invalid SMILES, reconstruction failures, and unresolved groups. Failed predictions have missing raw Tanimoto values and zero failure-penalized similarity. A Tanimoto value of 1 does not necessarily imply an exact structural match.

## Recorded results

The following values are taken from the final 100-sample execution outputs in the submitted notebook.

| Evaluation group | Samples | Exact matches | Exact-match accuracy | Complete valid-output rate | Valid-only mean Tanimoto | Failure-penalized mean Tanimoto | Mean inference time (s) |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| BioVista eligible subset | 100 | 72 | 72% | 87% | 0.9607 | 0.8358 | 1.2087 |
| XN exploratory crops | 3 | 3 | 100% | 100% | 1.0000 | 1.0000 | 1.3399 |
| XN pilot crop | 1 | 1 | 100% | 100% | 1.0000 | 1.0000 | 1.1288 |

BioVista outcomes comprise **72 exact matches, 15 valid but incorrect predictions, 3 reconstruction failures, and 10 unresolved-group outputs**. No model-call exceptions were recorded in this run. Reconstruction failures remain failed predictions rather than being removed or manually corrected.

### Saved result files

Each run exports:

| File | Contents |
| --- | --- |
| `sample_manifest.csv` | Fixed evaluation samples, provenance, image paths, crop coordinates, and reference SMILES. |
| `predictions.csv` | Raw inference-stage outputs, timing, and model-call errors; captions are not yet chemically reconstructed. |
| `predictions_postprocessed.csv` | Decoded outputs, original model status, reconstruction status, and errors. |
| `evaluated_predictions.csv` | Per-sample canonical SMILES, exact match, validity, similarity, timing, and final outcome category. |
| `benchmark_summary.csv` | Dataset- and usage-specific aggregate metrics. |
| `outcome_counts.csv` | Counts of each observed evaluation outcome. |
| `run_metadata.json` | Run identifier, dependency versions, device, checkpoint path and size, and evaluation settings. |
| `biovista_recognition_results.png` | Public-subset recognition rates and outcome counts. |
| `error_comparisons/` | Source/reference/prediction comparison figures for unsuccessful samples. |

Additional audit and preparation files are saved under `outputs/opencv_pilot/`, `BioVista_OCSR/`, and the benchmark directory. For repository review, `results/` contains the final exported summary, evaluated predictions, outcome counts, run metadata, sample manifest, and public result figure.

## Scope and limitations

- The public result applies to the selected 100 RDKit-parseable, nonwildcard references. It is not an estimate for all BioVista structures, the official validation/test split, or the complete literature extraction task.
- The four XN crops are manually selected complete drawings from one page. Their scores are descriptive and do not demonstrate general performance across scientific papers or the full 222-record course dataset.
- The auxiliary OpenCV experiment uses grayscale conversion, Otsu thresholding, Canny edges, morphological closing, and contour filtering. Its unreviewed regions are not used for recognition; molecule-detection precision and recall are not claimed.
- This experiment evaluates one pretrained tool. It does not establish a ranking against other OCSR systems or use the same evaluation protocol as the upstream published scores.
- Molecular image recognition alone does not resolve structures defined jointly by scaffolds, substituent tables, and cross-page references.
- Timing depends on hardware and runtime conditions. The submitted environment succeeded, but installation on a fresh environment has not been validated; exact upstream source revision and checkpoint hashes are not currently recorded.

## Sources and acknowledgments

- [BioMiner source and documentation](https://github.com/jiaxianyan/BioMiner).
- [Official MolGlyph model repository](https://huggingface.co/jiaxianustc/MolGlyph).
- [BioVista OCSR data access](https://github.com/jiaxianyan/BioMiner#statistics-and-access-of-biovista).
- Yan et al. (2025), *BioMiner: A Multi-modal System for Automated Mining of Protein-Ligand Bioactivity Data from Literature*. [doi:10.1101/2025.04.22.648951](https://doi.org/10.1101/2025.04.22.648951).
- Course-provided XN Dataset 2 and OpenCV Modules 2–3 tutorial.

The model, checkpoint, official decoder, and public benchmark are credited to their upstream authors. This repository contributes the experiment configuration, dataset audit, reproducible sample selection, inference orchestration, evaluation, and result presentation.

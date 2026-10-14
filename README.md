# Molecular Image Recognition Benchmarking

**Author:** Yihang (Kevin) Xu  
**Course:** AAI 6630 — Computer Vision  
**Assignment:** 1.2 Individual Benchmarking Code

## Project Overview

This project benchmarks the pretrained MolGlyph optical chemical structure recognition (OCSR) component from BioMiner. It evaluates how accurately molecular images are converted into SMILES using a fixed BioVista OCSR subset and molecular drawings cropped from the course XN dataset.

The notebook implements the following workflow:

1. Load datasets and audit reference records.
2. Prepare and freeze the evaluation sample list.
3. Run MolGlyph inference on CPU.
4. Reconstruct model captions using BioMiner's official decoder.
5. Calculate chemical validity, exact match, and Tanimoto similarity.
6. Export structured results and visual comparisons.

An auxiliary OpenCV experiment demonstrates PDF page preprocessing and exploratory region proposals.

This is a component-level recognition benchmark. It does not train a new model or evaluate the complete BioMiner literature-extraction system. BioVista provides the benchmark data rather than a second recognition model.

## Repository Structure

| File or directory | Description |
| --- | --- |
| `Xu_1_2_Benchmarking_Code.ipynb` | Main implementation with saved execution outputs, tables, figures, and error comparisons. |
| `README.md` | Project description, setup instructions, evaluation methods, and results. |
| `requirements.txt` | Dependencies recorded from the successful experiment environment. |
| `results/` | Structured outputs and the recognition-results figure from the final experiment. |
| `results/error_comparisons/` | Visual comparisons of unsuccessful predictions with source images and reference structures. |

The notebook can be reviewed directly through its saved outputs. Input archives, third-party source code, and model weights are acquired separately.

## Datasets and Sample Selection

### BioVista OCSR

The downloaded archive contains 8,861 molecular image/SMILES annotations.

The reference audit identified:

- 4,017 references containing wildcard atoms.
- One reference that could not be parsed by RDKit.
- 4,843 eligible references.

Eligibility is defined before inference: reference SMILES must be RDKit-parseable and contain no wildcard atoms.

The final experiment selects **100 samples without replacement**, using **random seed 42**. The eligible pool is sorted by sample ID before sampling, and selected records are sorted again before evaluation.

The notebook reads `ocsr/labels/gd_SMIELS.csv` directly from `ocsr.tar.gz`. Images are matched using the source document, page prefix from `molminer_id`, and the separate global `index` column. Annotation-to-image mappings are verified before sampling.

Only selected images are extracted, and their original contents are preserved. Results describe this eligible subset rather than the entire BioVista benchmark.

### Course XN Dataset 2

The course dataset contains six PDFs and 222 reference records.

Recognition is evaluated on four predefined molecular crops from **paper 2, PDF page 2, Figure 1C**, rendered at **300 DPI**:

| Compound | Sample usage |
| --- | --- |
| AKI-062a | Exploratory |
| D4476 | Exploratory |
| FP1-24 | Exploratory |
| Compound 1h | Previously evaluated pilot |

Each sample's document, page, compound label, crop coordinates, image path, and reference SMILES are recorded in the manifest.

These manually specified crops are recognition inputs, not outputs from the OpenCV region proposals. The pilot remains separate from the exploratory samples in aggregate results.

Reference SMILES are used only for evaluation and subsequent visual inspection. They are not passed to MolGlyph or used to repair predictions.

## Setup Instructions

### 1. Prepare the Project Directory

Download this repository and open a terminal in its root directory.

The local project requires:

- `Xu_1_2_Benchmarking_Code.ipynb`
- `requirements.txt`
- `Dataset 2.zip`
- `ocsr.tar.gz`
- `BioMiner-main/`

The notebook creates the extracted-image and output directories automatically.

### 2. Obtain Data and Model Files

1. Download **Dataset 2** from the course materials and place it in the project root as `Dataset 2.zip`. Keep the archive intact.
2. Download the [BioVista OCSR archive](https://drive.google.com/file/d/11iCH0j7U9iFgwXahjkWZouNyqATfEPoL/view) and place it in the project root as `ocsr.tar.gz`.
3. Download the [BioMiner repository](https://github.com/jiaxianyan/BioMiner) using **Code → Download ZIP**. Extract it into the project root as `BioMiner-main`.
4. Download the checkpoint from the [official MolGlyph model repository](https://huggingface.co/jiaxianustc/MolGlyph) and save it as:

   ```text
   BioMiner-main/scripts/molglyph_large.pt
   ```

   Access may require signing in and accepting the model provider's conditions.

5. Retain the following source files and directories from BioMiner:

   ```text
   BioMiner-main/BioMiner/MolScribe/molscribe/
   BioMiner-main/BioMiner/commons/molglyph_utils.py
   BioMiner-main/BioMiner/commons/abbrevs.csv
   ```

The course dataset requires access to the course materials and is not publicly redistributed here.

### 3. Install Dependencies

The recorded experiment used **Windows, Python 3.12.3, and CPU inference**. The notebook's configured execution path does not require a CUDA-capable GPU.

Create an isolated environment:

```bash
python -m venv .venv
```

Activate it on Windows PowerShell:

```powershell
.\.venv\Scripts\Activate.ps1
```

Or on macOS/Linux:

```bash
source .venv/bin/activate
```

Install dependencies and launch Jupyter:

```bash
python -m pip install -r requirements.txt
python -m pip install notebook
python -m notebook
```

Select the interpreter from this environment as the notebook kernel.

`requirements.txt` records the package versions from the successful submitted environment. Installation in a fresh environment has not been independently verified.

## Running the Benchmark

1. Launch Jupyter from the project root.
2. Open `Xu_1_2_Benchmarking_Code.ipynb`.
3. Review the first code cell's configuration:

   ```python
   PUBLIC_SAMPLE_SIZE = 100
   PUBLIC_RANDOM_SEED = 42
   ```

4. Restart the kernel and run all cells in order.
5. Inspect the dataset-specific summary, recognition-outcome figure, and error comparisons.
6. Save the executed notebook.

Once dependencies and input files are available, the workflow runs without interactive sample selection or manual edits between stages.

Input and output paths are configured relative to the project root. The combined manifest contains **104 samples: 100 BioVista images and four XN crops**.

Runs are saved under:

```text
outputs/benchmark_n100_seed42/run_<UTC timestamp>/
```

Each execution creates a separate timestamped run directory.

The notebook checks the frozen manifest before inference. Changing sample size or seed uses a different benchmark directory. If crop coordinates or reference inputs change without changing size or seed, configure a new `BENCHMARK_DIR` to preserve the earlier experiment.

## Evaluation Methods

### Caption Reconstruction

Raw model captions are preserved.

BioMiner's official `Translator.refactor` decoder processes captions containing separator and abbreviation annotations. Reconstructed predictions containing remaining dummy atoms are classified as unresolved structures.

The interface used in this notebook does not provide confidence scores.

### Metrics

| Metric | Definition |
| --- | --- |
| Exact-match accuracy | Fraction of all selected samples whose predicted and reference canonical isomeric SMILES are identical. |
| Complete valid-output rate | Fraction of all selected samples yielding parseable reconstructed structures without unresolved dummy atoms. |
| Valid-only mean Tanimoto | Mean fingerprint similarity over complete, valid predictions. |
| Failure-penalized mean Tanimoto | Mean over all selected samples, assigning zero to failed, empty, invalid, or unresolved predictions. |
| Inference time | Elapsed time for the image-readability check and model call, including image loading. |

Exact match retains stereochemistry and disconnected fragments.

Tanimoto similarity uses **Morgan fingerprints with radius 2, 2,048 bits, and chirality enabled**. A Tanimoto value of 1 does not necessarily establish an exact structural match.

Inference time excludes model initialization, subsequent official decoding, metric calculation, and figure generation.

### Failure Handling

All selected samples remain in the evaluation denominator.

Failure categories include:

- Inference exceptions.
- Empty outputs.
- Invalid SMILES.
- Postprocessing/reconstruction errors.
- Unresolved groups.

Failures count as incorrect exact matches. Their raw Tanimoto values are missing, while their failure-penalized values are zero.

Predictions are not manually corrected using reference structures.

## Recorded Results

The following values are taken from the final 100-sample execution outputs.

| Evaluation group | Samples | Exact matches | Exact-match accuracy | Complete valid-output rate | Valid-only mean Tanimoto | Failure-penalized mean Tanimoto | Mean inference time (s) |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| BioVista eligible subset | 100 | 72 | 72% | 87% | 0.9607 | 0.8358 | 1.2087 |
| XN exploratory crops | 3 | 3 | 100% | 100% | 1.0000 | 1.0000 | 1.3399 |
| XN pilot crop | 1 | 1 | 100% | 100% | 1.0000 | 1.0000 | 1.1288 |

BioVista outcomes are:

| Outcome | Count |
| --- | ---: |
| Exact match | 72 |
| Valid but incorrect | 15 |
| Postprocessing/reconstruction error | 3 |
| Unresolved group | 10 |
| Total | 100 |

No model-call exceptions were recorded in this run. Reconstruction failures and unresolved outputs remain in the reported results.

## Saved Result Files

The final experiment's exported files are included in `results/`.

| File | Contents |
| --- | --- |
| `benchmark_summary.csv` | Aggregate metrics reported separately by dataset and sample usage. |
| `evaluated_predictions.csv` | Per-sample predictions, reference structures, chemical scores, timing, and final outcome categories. |
| `outcome_counts.csv` | Counts of each evaluation outcome. |
| `predictions.csv` | Raw inference-stage outputs, timing, and model-call errors. |
| `predictions_postprocessed.csv` | Reconstructed predictions, original model status, postprocessing status, and errors. |
| `run_metadata.json` | Run identifier, dependency versions, device, checkpoint path and size, and evaluation settings. |
| `sample_manifest.csv` | Fixed samples, source provenance, image paths, crop coordinates, and reference SMILES. |
| `biovista_recognition_results.png` | Recognition rates and outcome counts for the public subset. |
| `error_comparisons/` | Visual comparisons for samples without an exact match. |

Additional audit and preparation files are generated under `outputs/opencv_pilot/`, `BioVista_OCSR/`, and the benchmark directory.

## Recognition Error Comparisons

The [error comparison directory](results/error_comparisons/) contains figures exported for unsuccessful predictions.

Files are numbered `error_case_001.png`, `error_case_002.png`, and so on. Each figure includes a sample ID that can be matched to `evaluated_predictions.csv`.

Each comparison contains three panels:

1. **Source image:** the original recognition input.
2. **Reference structure:** a molecular drawing generated from the supplied ground-truth SMILES.
3. **Predicted structure:** a drawing generated from the decoded output when parseable.

Unresolved predicted structures are labeled accordingly. When the prediction cannot be parsed, the third panel displays the failure category instead.

Figure titles report the outcome and Tanimoto similarity when available. The notebook also prints the reference SMILES, reconstructed prediction, raw model caption, and decoding error beneath each displayed comparison.

These figures support inspection of atom labels, connectivity, bond types, substituent positions, and stereochemistry. Differences in drawing orientation alone do not establish an error.

The final public experiment contains **28 cases without an exact match**: 15 valid but incorrect predictions, three reconstruction failures, and 10 unresolved-group outputs.

Visual inspection supports the failure analysis for Assignment 1.3. It does not repair predictions, remove unsuccessful samples, or change benchmark scores.

## Auxiliary OpenCV Experiment

The notebook demonstrates:

- PDF rendering.
- Grayscale conversion.
- Gaussian smoothing.
- Otsu thresholding.
- Canny edge detection.
- Morphological closing.
- Contour-based region proposals.

This experiment uses paper 1, PDF page 2. Text and nonchemical diagrams also generate edges and foreground regions.

The proposed regions are unreviewed and are not used as recognition inputs. Molecule-detection precision and recall are not evaluated.

## Scope and Limitations

- Public results apply to the selected 100 RDKit-parseable, nonwildcard references, not the entire BioVista dataset or its official validation/test protocol.
- The four XN crops are manually selected drawings from one page. Their results do not establish representative performance across scientific papers or the complete 222-record course dataset.
- This experiment evaluates one pretrained recognition tool and does not establish a ranking against other OCSR systems.
- Recognition of isolated molecular images does not resolve structures defined jointly by scaffolds, substituent tables, or cross-page references.
- Timing depends on hardware and runtime conditions.
- The recorded environment successfully executed the benchmark, but installation in a fresh environment has not been validated.
- Exact upstream source revision and checkpoint hashes are not currently recorded.

## Sources and Acknowledgments

- [BioMiner source and documentation](https://github.com/jiaxianyan/BioMiner).
- [Official MolGlyph model repository](https://huggingface.co/jiaxianustc/MolGlyph).
- [BioVista dataset documentation](https://github.com/jiaxianyan/BioMiner#statistics-and-access-of-biovista).
- Yan et al. (2025), *BioMiner: A Multi-modal System for Automated Mining of Protein-Ligand Bioactivity Data from Literature*. [DOI: 10.1101/2025.04.22.648951](https://doi.org/10.1101/2025.04.22.648951).
- Course-provided XN Dataset 2 and OpenCV Modules 2–3 tutorial.

The model, checkpoint, official decoder, and public benchmark are credited to their upstream authors. This project contributes the experiment configuration, reference auditing, reproducible sample selection, inference workflow, evaluation, and result presentation.

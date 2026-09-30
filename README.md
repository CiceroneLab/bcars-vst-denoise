# BCARS denoising on variance-stabilized data

Code for *Denoising broadband CARS hyperspectral images on variance-stabilized data: a unified
benchmark*.

This repository It holds the processing pipeline: the
heterodyne variance-stabilizing transform (VST), the detrending front end, CCV rank selection
and the phase-retrieval back end plus one notebook that reports every number in the paper.
The benchmarked deep denoisers live in their own repositories, linked below.

**Data:** [doi.org/10.5281/zenodo.22968813](https://doi.org/10.5281/zenodo.22968813)

## Project map

| | |
|---|---|
| [CiceroneLab/bcars-vst-denoise](https://github.com/CiceroneLab/bcars-vst-denoise) | **this repository** — VST, detrending, CCV, phase retrieval, all paper metrics |
| [CiceroneLab/BCARS_N2N](https://github.com/CiceroneLab/BCARS_N2N) | Noise2Noise baseline (CSBDeep/CARE) |
| [CiceroneLab/BCARS_S2DIP](https://github.com/CiceroneLab/S2DIP) | deep image prior baseline (fork) |
| [CiceroneLab/BCARS_DDS2M](https://github.com/CiceroneLab/DDS2M) | diffusion baseline (fork) |
| [Zenodo 10.5281/zenodo.22968813](https://doi.org/10.5281/zenodo.22968813) | data: Raman cubes, input cubes |


## Install

```
conda env create -f environment/crikit3.yml
pip install CRIkit2==0.4.4          # provides `crikit` (unmodified release)
```
`environment/image-proc.yml` is only needed to go from raw HDF5 to VST cubes (step 1).


## Example: 64x64 simulated ROI

`examples/sim_roi_AD_05.mat` (19 MB) is a crop of a simulated cube that carries its exact ground truth, so denoising can be scored. Run the
released pipeline on it:

```bash
conda run -n crikit3 python bcars_processing/step2_process.py --config examples/params_example.yaml

`examples/example.ipynb` demonstrates the results.

```
step1_preprocess.py   raw .h5  → dark/illumination correction, spectral alignment, VST,
                                 spectral clip, Whittaker detrend, normalization  → .mat
                                 [image-proc env]
step2_process.py      .mat     → CCV (tau) → phase retrieval + PEC → Raman .tif
                                 [crikit3 env]
```

Run either on its own:

```bash
conda run -n crikit3    python bcars_processing/step2_process.py    --config params.yaml
conda run -n image-proc python bcars_processing/step1_preprocess.py --config params.yaml
```
`run_pipeline.sh` is a convenience wrapper that runs both in their respective conda
environments; it needs raw `.h5` input.


## Benchmarked denoisers

The paper compares six denoisers. SVD and CCV are implemented here; the other four live in
their own repositories, each documenting its input format, run command and the settings used
for the paper:

| Method | Code | Original |
|---|---|---|
| SVD | this repository (`bcars_processing`) | — |
| **CCV** | this repository (`bcars_processing/ccvsvd`) — **first released here** | — |
| BM4D | `pip install bm4d` | Maggioni et al. 2013 |
| Noise2Noise | [CiceroneLab/BCARS_N2N](https://github.com/CiceroneLab/BCARS_N2N) | Lehtinen et al. 2018, on CSBDeep/CARE (Weigert et al. 2018); odd/even split after SPEND (Ding et al. 2025) |
| DIP | [CiceroneLab/S2DIP](https://github.com/CiceroneLab/S2DIP) (fork) | Luo et al. 2021 |
| DDS2M | [CiceroneLab/DDS2M](https://github.com/CiceroneLab/DDS2M) (fork) | Miao et al. 2023 |

All four read the same `.mat` cube this pipeline writes (`y_0_real`, `img_clean`,
`norm_min`/`norm_max`, `wn`), so no conversion is needed between the stages. Their **outputs**
— the retrieved Raman cubes behind every figure — are in the data deposit, together with the
metric tables.

## Citation and license

MIT, see `LICENSE`. The denoiser repositories listed above carry their own terms: BCARS_N2N
follows CSBDeep (BSD-3-Clause), while the DDS2M and S2DIP forks inherit their upstream
projects, which publish no license.

Please cite the paper, the data deposit
([10.5281/zenodo.22968813](https://doi.org/10.5281/zenodo.22968813)), and the original method
papers for any baseline you use.

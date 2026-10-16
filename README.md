<h1 align="center">Bare PCB Defect Detection — YOLO26</h1>

<p align="center">
  <strong>Inspection of bare (unassembled) printed circuit boards: a YOLO26 detector for the 7 classic copper and drilling defects — missing hole, mouse bite, open circuit, short, spur, spurious copper and pin hole — trained on a cleaned merge of PKU-Market-PCB, DeepPCB and web photos.</strong>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/model-YOLO26s-7c3aed.svg" alt="YOLO26s">
  <img src="https://img.shields.io/badge/mAP@50-0.895-16a34a.svg" alt="mAP@50 0.895">
  <img src="https://img.shields.io/badge/mAP@50--95-0.601-16a34a.svg" alt="mAP@50-95 0.601">
  <img src="https://img.shields.io/badge/classes-7-2563eb.svg" alt="7 classes">
  <img src="https://img.shields.io/badge/weights-included-f59e0b.svg" alt="Weights included">
  <img src="https://img.shields.io/badge/Colab-T4%20ready-f9ab00.svg?logo=googlecolab&logoColor=white" alt="Colab ready">
  <img src="https://img.shields.io/badge/license-MIT-green.svg" alt="MIT">
</p>

<!--PRED
<p align="center">
  <img src="assets/predictions.jpg" alt="YOLO26 bare PCB defect detection on held-out boards: open circuit, short, mouse bite, spur, spurious copper and missing hole detected" width="100%">
  <br><em>Detections of the trained model on held-out test boards (board designs never seen in training).</em>
</p>
PRED-->

---

## At a glance

| | |
|---|---|
| **Task** | Object detection of fabrication defects on bare PCBs (pre-assembly AOI) |
| **Model** | Ultralytics **YOLO26s**, 1024 px input, tiled inference for full-resolution scans |
| **Accuracy (validation)** | **mAP@50 = 0.895 · mAP@50-95 = 0.601 · Precision 0.867 · Recall 0.871** |
| **Data** | 5,488 unique images from 3 sources (PKU-Market-PCB full-resolution, DeepPCB, web photos), 46,098 boxes |
| **Cleaning** | 4,295 pixel-identical and 617 mirrored/rotated copies removed; split by physical board |
| **Weights** | [`weights/best.pt`](weights/) — 20 MB, plus per-class confidence thresholds |
| **Hardware** | Free Google Colab **Tesla T4**; inference on CPU or GPU |

## Defect classes

<p align="center">
  <img src="assets/defect_gallery.jpg" alt="Gallery of bare PCB defect classes: missing hole, mouse bite, open circuit, short, spur, spurious copper, pin hole" width="100%">
  <br><em>Two real annotated examples per class from the validation/test splits (colour photos: PKU-Market-PCB and web; black-and-white: DeepPCB binarised scans).</em>
</p>

| # | Class | Description |
|---|---|---|
| 0 | `missing_hole` | A drill hole that was never drilled |
| 1 | `mouse_bite` | Bite-shaped notch eaten into the edge of a trace |
| 2 | `open_circuit` | A trace broken through — no electrical connection |
| 3 | `short` | Copper joining two conductors that must be isolated |
| 4 | `spur` | Thin copper protrusion sticking out of a trace |
| 5 | `spurious_copper` | Leftover copper island where there should be none |
| 6 | `pin_hole` | Small hole inside a copper area |

<p align="center">
  <img src="assets/zoom_examples.jpg" alt="Zoomed example of a missing hole and a mouse bite on a bare PCB" width="80%">
  <br><em>How small these defects are: the whole board, and the zoomed defect.</em>
</p>

## Results

<!--CURVES
<p align="center">
  <img src="results/training_curves.png" alt="Training curves of the YOLO26s bare PCB defect model" width="100%">
</p>
CURVES-->

| Metric (validation, best checkpoint) | Value |
|---|---|
| Precision | 0.867 |
| Recall | 0.871 |
| mAP@50 | **0.895** |
| mAP@50-95 | **0.601** |

The model already reaches mAP@50 0.82 after one epoch — the merged dataset gives every class at least 1,100 training boxes. The validation and test splits contain **different board designs** from training, so the numbers measure generalisation to new layouts, not memorisation of the 10 PKU boards.

## Dataset

<p align="center">
  <img src="assets/data_sources.png" alt="Sources of the bare PCB defect dataset and their train/val/test split" width="100%">
  <img src="assets/class_distribution.png" alt="Annotated bare PCB defects per class and split" width="100%">
</p>

Public PCB-defect datasets are heavily duplicated: an audit of 52 Roboflow datasets found that ~20 of the large ones are resized, cropped or augmented copies of the same **693 PKU-Market-PCB photos**. This repository builds the dataset from originals only:

| Source | Images | Physical boards / groups | Notes |
|---|---|---|---|
| PKU-Market-PCB (official, full-resolution) | 693 | 10 boards | official labels |
| DeepPCB | 1,500 | 11 scans | binarised 640 px template/test pairs |
| Web photo collection (`aaa_hr`) | 5,135 | 57 groups | 2048 px originals re-cropped to the real photo, padding removed |

- **Group-aware split** by board / scan / group — a board design never appears in two splits.
- **De-duplication** — 4,295 pixel-identical files and 617 mirrored/rotated copies excluded (every exclusion and reason recorded).
- **Exact 90° training copies** → 18,276 training files.
- Every class has ≥ 1,132 training boxes and 139–485 boxes in each of val and test.

| Split | Files | Boxes |
|---|---|---|
| train | 18,276 | 41,268 |
| val | 1,090 | 2,426 |
| test | 1,017 | 2,404 |

Details: [`docs/dataset_report.txt`](docs/dataset_report.txt).

## Quick start

```bash
git clone https://github.com/ParsaVictor/pcb-bare-board-defect-detection-yolo26.git
cd pcb-bare-board-defect-detection-yolo26
pip install -r requirements.txt

python infer/predict.py --weights weights/best.pt --source path/to/board.jpg --imgsz 1024 --out results/my_run
```

Outputs per image: `<name>_annotated.jpg`, `<name>.json` (class, confidence, box) and a run-level `summary.csv`. Full-resolution scans (e.g. PKU 3000 × 2000 px) are tiled and merged with class-aware NMS so pin holes a few pixels wide are not lost to down-scaling.

```python
from ultralytics import YOLO
model = YOLO("weights/best.pt")
results = model.predict("board.jpg", imgsz=1024, conf=0.25)
results[0].save("board_pred.jpg")
```

## Train it yourself (Google Colab)

[`notebooks/Bare_Model_Colab.ipynb`](notebooks/Bare_Model_Colab.ipynb) — *Run all* on a free T4: live training panel, checkpoints to Google Drive every 2 epochs, automatic resume after a disconnect (from any account), divergence guard (rollback + LR halving), per-class thresholds and a test report at the end.

```bash
python train/train_solder.py --data final_bare --work runs --drive /content/drive/MyDrive/runs/bare \
       --model yolo26s.pt --imgsz 1024 --epochs 50 --patience 20
```

(`train_solder.py` is the shared training script of both defect models.)

## Rebuild the dataset

```bash
export ROBOFLOW_API_KEY=...
python tools/download_versions.py      # sources in configs/downloads.csv
python tools/aaa_hr.py                 # re-crop the web photos from their 2048 px originals
python run_pipeline.py                 # extract → map classes → de-duplicate → group split → verify
```

## Repository layout

```
├── weights/            best.pt (YOLO26s, 20 MB) + thresholds.json
├── infer/predict.py    tiled inference with per-class thresholds
├── train/              training script with Drive sync, resume and divergence guard
├── notebooks/          one-click Colab notebook
├── tools/              audit of 52 public datasets, de-duplication, group-aware split, verification
├── configs/            class taxonomy and source → class mapping
├── results/            results.csv, training curves
├── docs/               dataset report
└── assets/             README figures
```

## FAQ

**Which defects does it find?**
The seven standard bare-board defects: missing hole, mouse bite, open circuit, short, spur, spurious copper and pin hole.

**Is it trained on PKU-Market-PCB or DeepPCB?**
Both, plus 5,000 web photos of other boards — merged into one taxonomy, de-duplicated, and split by board so the score reflects unseen designs.

**How accurate is it?**
mAP@50 = 0.895 and mAP@50-95 = 0.601 on the validation boards with YOLO26s at 1024 px.

**What about soldering defects on assembled boards?**
Use the companion model [**pcb-solder-defect-detection-yolo26**](https://github.com/ParsaVictor/pcb-solder-defect-detection-yolo26).

## Part of a PCB inspection suite

| Repository | Stage | What it does |
|---|---|---|
| **pcb-bare-board-defect-detection-yolo26** | before assembly | copper / etching defects on bare boards |
| [pcb-solder-defect-detection-yolo26](https://github.com/ParsaVictor/pcb-solder-defect-detection-yolo26) | after assembly | soldering and placement defects |
| [pcb-component-detection-yolo26](https://github.com/ParsaVictor/pcb-component-detection-yolo26) | after assembly | locates and identifies every component |
| [pcb-component-classifier](https://github.com/ParsaVictor/pcb-component-classifier) | per part | explainable Random-Forest component classifier |

## Citation

```bibtex
@software{karkooti_2026_bare_pcb_defects,
  author = {Karkooti, Mohammad Parsa},
  title  = {Bare PCB Defect Detection with YOLO26},
  year   = {2026},
  url    = {https://github.com/ParsaVictor/pcb-bare-board-defect-detection-yolo26}
}
```

Please also cite the source datasets: PKU-Market-PCB (Huang & Wei, 2019) and DeepPCB (Tang et al., 2019).

## License

Code and weights: MIT — see [LICENSE](LICENSE). Source images keep their original licences (see [`configs/downloads.csv`](configs/downloads.csv)).

## Author

**Mohammad Parsa Karkooti** — AI Engineer (Computer Vision, Deep Learning)
[GitHub](https://github.com/ParsaVictor) · [LinkedIn](https://www.linkedin.com/in/parsa-karkooti) · [Email](mailto:1.parsa.karkooti@gmail.com)

<sub>Keywords: PCB defect detection, bare board inspection, PKU-Market-PCB, DeepPCB, mouse bite, open circuit, short circuit, spur, spurious copper, missing hole, pin hole, YOLO26, automated optical inspection, AOI, printed circuit board quality control.</sub>

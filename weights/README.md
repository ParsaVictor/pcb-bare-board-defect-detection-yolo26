# Weights

| File | Content |
|---|---|
| `best.pt` | YOLO26s, 1024 px, 7 classes, epoch 17 of 50 (best validation mAP@50-95) |
| `thresholds.json` | per-class confidence (F1 maximum on validation), read automatically by `infer/predict.py` |
| `summary.json` | chosen run and its validation / test mAP@50-95 |

Test set: mAP@50 0.928, mAP@50-95 0.617.

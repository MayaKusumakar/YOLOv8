# YOLOv8

# Fine-Tuning YOLOv8 to Detect Clownfish

Technical report on experimenting with training, evaluating, and fine-tuning a pretrained YOLOv8 model and using a custom dataset to add a new detecting class: **clownfish**.

| Item | Detail |
|---|---|
| **Base model** | YOLOv8 (pretrained weights) |
| **New class** | `clownfish` (class ID `0`) |
| **Dataset** | 18 images: 14 train / 4 validation, annotated in Roboflow |
| **Best run** | Run 3: up to 200 epochs, batch 2, 640 px, patience 30 (early-stopped at epoch 33) |
| **Validation results** | Precision 0.80 · Recall 1.00 · mAP50 0.945 · mAP50-95 0.721 |

## Contents

1. [Objective and Dataset](#1-objective-and-dataset)
2. [Model Training Procedure](#2-model-training-procedure)
3. [Debugging](#3-debugging)
4. [Validation & Performance](#4-validation--performance)
5. [Limitations and Future Plans](#5-limitations-and-future-plans)

---

## 1. Objective and Dataset

### Objective

The objective of this exercise was to fine-tune YOLO to detect another subclass of images: **clownfish**. To do this, I collected images of clownfish from the internet, annotated the images with a label and bounding box, and fine-tuned YOLO using this new custom dataset.

### Dataset

| Property | Value |
|---|---|
| Total images | 18 clownfish images |
| Training split | 14 |
| Validation split | 4 |
| Classes | 1: `clownfish` (class ID `0`) |
| Annotation | Bounding boxes and labels manually annotated in Roboflow |
| Export format | YOLO |

### Label format

Each label has the class ID and the normalized bounding box (center, width, and height):

```text
<class_id> <x_center> <y_center> <width> <height>
```

The class, clownfish, has class ID `0`.

---

## 2. Model Training Procedure

### 2.1 Warm-up on COCO8

To start, I used the pretrained YOLOv8 weights and practiced training on `coco8.yaml`. I then tested the outputs of the newly trained model on the same COCO8 dataset. After this, I tested the model on random images from the internet.

### 2.2 Fine-tuning on the clownfish dataset

For the custom dataset, I started back at the pretrained YOLO model and fine-tuned it on my custom clownfish dataset (14 training / 4 val). I did training a few times and changed some training configs to improve performance:

| Run | Epochs | Batch size | Image size | Patience | Result |
|:---:|---:|---:|---:|---:|---|
| 1 | 10 | — | 640 | — | No prediction boxes drawn & no strong prediction (~0.01 max confidence) |
| 2 | 50 | 2 | 640 | — | Multiple boxes drawn, each with ~0.02 confidence |
| **3** | **200** | **2** | **640** | **30** | **Best performance. One bounding box drawn with 0.51 confidence** |

**Run 3** yielded the best performance:

- It started training from the **last run's (50-epoch) weights**.
- The run **early stopped 33 epochs in**.

---

## 3. Debugging

The first run yielded very poor results: it had no real "strong" prediction, just a bunch of very low-confidence predictions (~0.014). At first, there were no bounding boxes drawn and I was wondering what was going wrong.

To figure it out, I:

1. **Checked the dataset** and confirmed that the bounding boxes were in the labels.
2. **Lowered the confidence threshold iteratively** to expose the low-confidence predictions.
3. **Experimented with different training configs and confidence thresholds** in the following runs, to eventually arrive at a much better-performing model.

---

## 4. Validation & Performance

The successful run is saved under **`runs/clownfish_150epochs`**.

### Metrics (validation set)

| Metric | Value | What it measures |
|---|---:|---|
| Precision | 0.80 | Share of predicted boxes that were correct |
| Recall | 1.000 | Share of actual clownfish that were detected |
| F1 | 0.889 (88.9%) | Harmonic mean of precision and recall |
| mAP50 | 0.9450 | Mean average precision at IoU 0.50 |
| mAP50-95 | 0.7206 | Mean average precision averaged over IoU 0.50–0.95 |
| Fitness | 0.7430 | Ultralytics model-selection score: 0.1 × mAP50 + 0.9 × mAP50-95 |

### Confusion matrix

<p align="center">
  <img src="https://github.com/user-attachments/assets/e2d195b3-1e51-4730-8156-62fe608871c9" alt="Confusion matrix for the validation set" width="640">
</p>

*Figure 1. Validation confusion matrix: 4 correct detections, 1 false positive and 0 missed clownfish.*

### Precision–recall curve

<p align="center">
  <img src="https://github.com/user-attachments/assets/e92891c9-1841-4fe0-a517-c11eda9a7d0a" alt="Precision-recall curve" width="640">
</p>

*Figure 2. Precision–recall curve for the clownfish class.*

### F1–confidence curve

<p align="center">
  <img src="https://github.com/user-attachments/assets/56ceebbc-c338-438a-b92b-b1df67870b0b" alt="F1-confidence curve" width="640">
</p>

*Figure 3. F1 score against confidence threshold. F1 stays at its maximum of 0.89 for thresholds up to about 0.5 and falls to 0 above about 0.84.*

### Training curves


<p align="center">
  <img src="https://github.com/user-attachments/assets/601fedad-4e14-4490-a91c-f045dcdd2da4" alt="Training and validation losses and metrics per epoch" width="900">
</p>

*Figure 4. Per-epoch training and validation losses and validation metrics for Run 3.*

### Sample predictions

<table>
  <tr>
    <th>Validation images</th>
    <th>Unseen test image</th>
  </tr>
  <tr>
    <td align="center"><img src="https://github.com/user-attachments/assets/8299eb9d-e51c-4fe5-9e9a-af01c4bc156f" alt="Predictions on the four validation images" width="240"></td>
    <td align="center"><img src="https://github.com/user-attachments/assets/bff3dc85-4790-48ee-b944-eb18fc916999" alt="Prediction on an unseen test image" width="480"></td>
  </tr>
</table>

*Figure 5. Left: predictions on the four validation images (confidence 0.47–0.77). Right: prediction on an unseen test image (confidence 0.52).*

---

## 5. Limitations and Future Plans

Fine-tuning the model on a new dataset did indeed expand the model's detection ability to include another class (clownfish), but its performance can still be improved.

### Limitations

- **Small dataset.** The biggest limitation of this model and the performance metrics results is the small dataset size. There are only 14 images used for training and only 4 images used for validation. This small dataset cannot represent all different images of clownfish (different backgrounds, positions, lighting, quality, etc.).
- **Each image affects metrics heavily.** The confusion matrix showed 4 correct detections, one false positive, and no missed fish. These results seem very good at surface level, but because of the small dataset size, each image has a very large impact on each metric. For example, the one false positive changes precision from 1.00 to 0.80.
- **No independent test set.** The model was also tested on one unseen test image, but the metrics were calculated from the validation set. A larger, more diverse test set would be needed for an unbiased, realistic evaluation.
- **No negative examples.** Every image in the validation set has at least one clownfish, so it didn't properly evaluate the model on negative detections.

### Future plans

Overall, future improvements include:

- [ ] Collecting more images
- [ ] Broadening the dataset and diversifying it (including images without any fish)
- [ ] Creating a robust, separate test set

With these, we can improve training configurations to make the model more robust and perform better.

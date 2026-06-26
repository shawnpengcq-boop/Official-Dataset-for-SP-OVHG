# SP-OVHG Sichuan Pepper Dataset

Supporting dataset for the paper:

> **SP-OVHG: A Vision-Language Hierarchical Framework for Automated Maturity Assessment and Quality Grading of Sichuan Pepper**

This repository releases the raw images and object-detection annotations used to build and evaluate the SP-OVHG framework. The dataset was collected from Hanyuan county, Sichuan, China — a major production region of Hanyuan Sichuan Pepper (*Zanthoxylum bungeanum*) — and is annotated for pepper fruit detection at two granularities.

## Repository Structure

```
public/
├── Hanyuan Sichuan Pepper Dataset/
│   ├── images/      # 669 .jpg images (full set, field-collected)
│   └── labels/      # 669 YOLO-format .txt annotation files
├── Precisely Annotated Dataset/
│   ├── images/      # 353 .jpg images (curated high-quality subset)
│   └── labels/      # 353 YOLO-format .txt files + classes.txt
└── README.md
```

## Sub-datasets

| Sub-dataset | Images | Label files | Instances | Class 0 instances | Class 1 instances |
|---|---|---|---|---|---|
| Hanyuan Sichuan Pepper Dataset (full) | 669 | 669 | 20,415 | 1,202 | 19,213 |
| Precisely Annotated Dataset (curated subset) | 353 | 353 | 10,461 | 654 | 9,807 |
| **Total** | **1,022** | **1,022** | **30,876** | **1,856** | **29,020** |

- **Hanyuan Sichuan Pepper Dataset** is the full collection of in-field images. Filenames follow two patterns: timestamped captures (`IMG_20230630_xxxxxx.jpg`) and a separate numbered series (`pepper (n).jpg`).
- **Precisely Annotated Dataset** is a curated, high-quality subset re-annotated with stricter quality control. Its images are a subset of the full set (matching timestamped filenames), selected to support cleaner training/evaluation. Every image has a corresponding `.txt` label.

## Annotation Format

Labels follow the [YOLO](https://github.com/ultralytics/yolov5) text format. Each `.txt` file contains one row per detected instance:

```
<class_id> <x_center> <y_center> <width> <height>
```

where `x_center`, `y_center`, `width`, and `height` are normalized to `[0, 1]` relative to the image size. Coordinates are top-left–origin, axis-aligned bounding boxes.

### Classes

There are **2 classes**, encoding a two-granularity detection scheme:

| `class_id` | Name (EN) | Name (中文) | Description |
|---|---|---|---|
| `0` | pepper cluster | 花椒簇 | A whole pepper cluster /更大的包围框 |
| `1` | individual pepper | 花椒粒 | A single pepper fruit / smaller box |

A `classes.txt` file (listing the class names/indices, one per line) is included in the `Precisely Annotated Dataset/labels/` directory for convenience.

## Image Properties

- Format: JPEG (`.jpg`)
- Resolution: high-resolution, typically `3072×2386` to `4096×3072` (varies per device/capture)
- Location: orchards in Hanyuan county, Sichuan, China
- Capture dates are embedded in the timestamped filenames

## Task

This release is provided for **object detection** of Sichuan pepper fruit at cluster and individual-granularity. Maturity assessment and quality grading in SP-OVHG are built on top of these detections via downstream vision-language reasoning; no predefined train/val/test split is included — users may split the data themselves. The curated subset can be used for cleaner experiments, while the full set supports larger-scale training.

## Data Splits

No predefined splits are provided. Example `data.yaml` for use with Ultralytics YOLO:

```yaml
path: ./public
train: Hanyuan Sichuan Pepper Dataset/images
val: Precisely Annotated Dataset/images
nc: 2
names: ['pepper_cluster', 'individual_pepper']
```

> Adjust the split to match your own experimental protocol. To reproduce the paper, please use the split described in the paper's Methods / Supplementary.

## Getting Started

```bash
git clone https://github.com/<your-user>/<your-repo>.git
cd <your-repo>
# Example: verify a sample image and its label
ls "Hanyuan Sichuan Pepper Dataset/images" | head -5
ls "Hanyuan Sichuan Pepper Dataset/labels" | head -5
```

A simple loader / visualization snippet:

```python
from pathlib import Path
import cv2

imgs = sorted(Path("Hanyuan Sichuan Pepper Dataset/images").glob("*.jpg"))
for img_path in imgs[:1]:
    img = cv2.imread(str(img_path))
    h, w = img.shape[:2]
    lbl_path = Path("Hanyuan Sichuan Pepper Dataset/labels") / (img_path.stem + ".txt")
    for line in lbl_path.read_text().splitlines():
        c, x, y, bw, bh = map(float, line.split())
        x1, y1 = int((x - bw / 2) * w), int((y - bh / 2) * h)
        x2, y2 = int((x + bw / 2) * w), int((y + bh / 2) * h)
        color = (0, 0, 255) if int(c) == 0 else (0, 255, 0)
        cv2.rectangle(img, (x1, y1), (x2, y2), color, 8)
    cv2.imwrite("preview.jpg", img)
```

## Online System

Beyond the dataset released here, we have deployed a full **Sichuan Pepper Detection and Quality Grading System** that implements the SP-OVHG pipeline end-to-end. The system takes orchard / crop images as input, performs hierarchical fruit detection (cluster-level and individual-level), and produces per-image **maturity assessment** together with a **quality grade** through vision-language reasoning — mirroring the methodology described in the paper.

You can try the model online, no installation required:

> **Sichuan Pepper Detection & Quality Grading System** — [http://47.108.142.21:7180/](http://47.108.142.21:7180/)

### What it does

- **Hierarchical detection** — localizes pepper clusters (class 0) and individual pepper fruits (class 1) in a single forward pass, using the two-granularity annotations provided in this repository.
- **Maturity assessment** — estimates maturity stage from visual cues (color, dehiscence, pericarp coverage) via vision-language prompts.
- **Quality grading** — aggregates detection and maturity signals into an overall grade, supporting rapid post-harvest sorting decisions for the Hanyuan Sichuan Pepper industry.
- **Web interface** — upload an image to get detection boxes, maturity labels, and a grade in the browser.

> The online system may be intermittently available for maintenance. For reproducible research, train and evaluate models on the data released here using the protocol in the paper.

## License

This dataset is released under the **MIT License**. You are free to use, copy, modify, merge, publish, and distribute the data, provided that the copyright notice and permission notice are included in all copies.

## Citation

If you use this dataset in your research, please cite the accompanying paper:

```bibtex
@article{spo7hg2026,
  title  = {SP-OVHG: A Vision-Language Hierarchical Framework for Automated
            Maturity Assessment and Quality Grading of Sichuan Pepper},
  author = {<Pengjun Xianga,, Xubo Zhanga,, Chengkai Yu, Qiang Huang, Jianjun Chen, Francesco Marinello>},
  year   = {2026},
  note   = {Manuscript under review. Dataset available at
            https://github.com/<your-user>/<your-repo>}
}
```

> Replace the author list, year, and repository URL with the final values once the paper is accepted. A DOI will be added upon publication.

## Acknowledgements

We thank the pepper growers in Hanyuan county, Sichuan, for granting access to their orchards during the 2023 harvest season, and the annotation team for the carefully labeled bounding boxes.

## Contact

For questions about this dataset, please open an issue in this repository or contact the corresponding author of the SP-OVHG paper.

# Analysis of Person Detection & Tracking (YOLOv8 + ByteTrack)

> **Project Type:** Computer Vision / Object Detection & Tracking Experiment  
> **Tech Stack:** Python, YOLOv8 (Ultralytics), ByteTrack, OpenCV, Jupyter Notebooks  
> **Dataset:** MOT20-03 Benchmark (MOT Challenge)  
> **Course / Context:** Machine Learning (COMP 542)

---

##  Overview
This project was my first foray into computer vision, focusing on pedestrian detection, multi-object tracking (MOT), and directional line-crossing counting. Using the MOT20-03 benchmark dataset, the primary objective was to experiment with temporal sampling rates (75ms to 400ms) to analyze the trade-off between detection accuracy, processing speed, and file/storage efficiency.


https://github.com/user-attachments/assets/4080edf9-5cad-41c3-93d6-6a6d5aae4a85




---

##  System Architecture & Workflow

1. **Detection Backbone:** **YOLOv8** for frame-by-frame object detection.
2. **Object Tracking:** **ByteTrack** for maintaining identity consistency across frames.
3. **Boundary Counting:** Custom line-crossing detection algorithm to track unidirectional flow.
4. **Sampling Optimization:** Evaluated variable frame sampling rates to determine optimal operational thresholds.

---

##  Experimental Results & Sampling Analysis

By varying the frame processing interval from $75\text{ ms}$ to $400\text{ ms}$, three distinct operational regions were identified:

| Frame Interval (ms) | Person Count | File Size (MB) | Relative Accuracy (%) | Operational Status |
| :--- | :--- | :--- | :--- | :--- |
| **75 ms** | 140 | 44.2 | 99.3% | High-Fidelity |
| **100 ms** | **141** | **27.6** | **100.0%** | **Optimal Baseline** |
| **115 ms** | 141 | 27.6 | 100.0% | High-Fidelity Upper Limit|
| **120–150 ms** | 126 | 20.6 | 89.4% | Balanced / Moderate Loss|
| **200–400 ms** | 60–108 | 7.74–14.0 | 42.6%–76.6% | Low-Fidelity (High Degradation) |

* **Key Finding:** The **100ms–115ms frame interval** proved to be the optimal sweet spot, preserving **100% relative accuracy** while achieving a ~37% reduction in storage size compared to 75ms sampling.

---

##  Real-World Challenges & Self-Critique

Working with raw video data exposed several practical computer vision edge cases:

### 1. Static Object Misclassification (The "Statue" Problem)
* **The Issue:** A stationary statue in the scene consistently registered false positives.
* **Workaround:** Reiterated the counting line placement to avoid static false positives.
* **Self-Critique:** Rather than fine-tuning the detector or implementing a ROI (Region of Interest) mask/static object filter, line position was adjusted manually.

### 2. Low Confidence Thresholding & Variable Lighting
* **The Issue:** Unstable camera footage and dark lighting led to low detection confidence.
* **Workaround:** Lowered the detection threshold to `0.15` and tracking threshold to `0.15` to avoid dropping trajectories.
* **Self-Critique:** A detection threshold of `0.15` is aggressively low in production settings. Proper image preprocessing (e.g., CLAHE for lighting normalization) or fine-tuning YOLO on domain-specific data would have been a cleaner approach.

### 3. Evaluation & Metrics
* **Self-Critique:** The initial evaluation lacked formal error classification tools like **Confusion Matrices**, Precision-Recall curves, or standard MOT metrics (MOTA/IDF1). Performance was judged primarily on raw count delta across sampling rates.

---

##  Growth & Next Steps for Model Fine-Tuning

Looking back with a deeper understanding of ML engineering, future iterations would implement:
* **Custom Model Fine-Tuning:** Fine-tune YOLO weights on dense crowd datasets instead of relying purely on default COCO pre-trained weights to handle occlusion cleanly.
* **Rigorous Evaluation:** Compute Precision, Recall, F1-Score, and MOTA/IDF1 metrics across varying confidence thresholds.
* **Camera Stabilization & Filtering:** Add video preprocessing (e.g., optical flow / digital stabilization) to smooth camera shake prior to inference.

---

##  References
* [ByteTrack: Multi-Object Tracking by Associating Every Detection Box](https://github.com/ifzhang/ByteTrack)
* Ultralytics YOLOv8 Documentation
* MOT Challenge Benchmark (`MOT20-03`)

---
layout: page
title: CVPR 2025 Anti-UAV Challenge
description: Developed a UAV tracking and detection system using SiamFC and YOLOv11.
img: assets/img/uav.jpg
importance: 1
category: project
related_publications: False
---

Project Overview:
CVPR 2025 had an Anti-UAV Challenge (https://anti-uav.github.io), where the goal is accurate drone/uav detection. Two of my friends and I decided to participate. We decided to try two different methods of detection: SiamFC and YOLOv11. The following is a short writeup for the project, but a full report, as well as all the code, is available at https://github.com/tomaszfrelek2/CVPR_2025_Anti-UAV_Challenge.
---

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/drone_images.png" title="Example drone images" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    The Anti-UAV dataset includes diverse scenarios such as clouds, buildings, mountains, and sea backgrounds to test model generalization.
</div>

---

We utilized a curated subset of the Anti-UAV dataset, subsampling more than 300,000 frames down to 20,000 representative images to ensure computational tractability.

Baseline Approach: SiamFC
* Implemented Siamese Fully-Convolutional Networks (SiamFC) for UAV tracking.
* The tracker formulates the problem as similarity learning between an exemplar image and search frames.
* Limitations: SiamFC struggles with occlusions, rapid motion, and re-identification after disappearance.

Advanced Approach: YOLOv11
* Utilized the YOLO11s architecture, a lightweight model with ~9.4 million parameters.
* Performs frame-wise detection independently, allowing for automatic re-detection after occlusions.
* Strengths: High inference speed and superior robustness across diverse environments.

---

<div class="row justify-content-sm-center">
    <div class="col-sm-4 mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/siamfc_output.png" title="SiamFC Output" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm-4 mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/yolo11_output.png" title="YOLOv11 Output" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Left: A heatmap produced by SiamFC , where the peak corresponds to the drone's location. Right: Various examples of YOLOv11's drone detections.
</div>

---

Experimental Results

The models were evaluated on 4,000 unseen images to assess their ability to maintain accurate tracking and detection.

| Metric    | SiamFC | YOLOv11 |
| --------- | ------ | ------- |
| Precision | 0.409  | 0.96    |
| mAP@50    | 0.364  | 0.88    |
| Recall    | 0.391  | 0.834   |

The detection-based approach (YOLO) significantly outperformed the tracking-based baseline, achieving higher precision and handling challenging re-identification scenarios better.
YOLOv8n Training Pipeline Guide
Overview

This guide explains how to build a complete YOLOv8n training pipeline starting from raw images and COCO annotations, up to a fully trained model.

Requirements

To begin, you need:

A folder containing all images used for annotation
A folder containing all COCO JSON annotation files
Required Scripts

You will need a set of scripts to process your dataset. These scripts handle different stages of preparation:

Rename images (optional)
Used to ensure all image filenames are unique.
Example:
Folder: MV100
Image: frame_0003.png
Result: MV100_frame_0003.png
This prevents duplicate filenames across different folders.
Merge images
Combine all images from multiple folders into a single dataset.
Merge COCO JSON files
Combine multiple annotation files into one unified COCO file.
Add negatives to COCO
Ensure images without annotations are included in the dataset.
Create COCO splits
Split the dataset into training and validation sets (based on video groups to avoid leakage).
Convert COCO → YOLO format
Generate YOLO label files and organize dataset structure.
Create dataset.yaml
Define dataset paths and class names for YOLO training.
Recommended Project Structure
x_dataset/
├── Frames/                  # Raw extracted frames (organized by video)
├── coco_parts/              # Individual annotation JSON files
├── Frames_all_combined/     # Final merged image dataset
├── merged/                  # Final merged COCO annotations
├── scripts/                 # All processing scripts
└── yolo/                    # Final YOLO-ready dataset
YOLO Dataset Structure

Your YOLO dataset must follow this structure:

C:\Code\x_dataset\yolo\
├── dataset.yaml
├── images\
│   ├── train\
│   └── val\
└── labels\
    ├── train\
    └── val\
Step 1 — Prepare Final Dataset

Before splitting, you must have:

A merged COCO file:
C:\Code\x_dataset\merged\coco_merged_scale_only_final.json
A merged image folder:
C:\Code\x_dataset\Frames_all_combined

(File names can vary — structure is what matters.)

Step 2 — Create Train / Validation Splits

This step generates:

C:\Code\x_dataset\merged\splits\
├── instances_train.json
├── instances_val.json
├── train_groups.txt
└── val_groups.txt

The split is done by video group, not randomly, to avoid data leakage between train and validation.

Run the split script:

cd C:\Code\x_dataset\scripts
python split_coco_by_video.py
Step 3 — Convert COCO to YOLO Format

Ensure the script build_yolo_from_split.py is in your scripts folder.

Run:

cd C:\Code\x_dataset\scripts
python build_yolo_from_split.py

This step will:

Copy images into train/val folders
Create YOLO label files
Generate empty label files for negative images
Build the full YOLO dataset structure
Step 4 — Verify Dataset Integrity

Run the following commands to confirm everything is correct:

(Get-ChildItem C:\Code\x_dataset\yolo\images\train -File).Count
(Get-ChildItem C:\Code\x_dataset\yolo\images\val -File).Count
(Get-ChildItem C:\Code\x_dataset\yolo\labels\train -File).Count
(Get-ChildItem C:\Code\x_dataset\yolo\labels\val -File).Count

Expected:

Number of images = number of labels (for both train and val)
Every image must have a corresponding .txt file
Negative images must have empty label files
Step 5 — Training (CPU Example)

Once everything is verified, start training:

yolo detect train model=yolov8n.pt data=C:\Code\x_dataset\yolo\dataset.yaml epochs=50 imgsz=640 batch=16 project=C:\Code\x_dataset\runs_v2 name=xxx
Training Time Expectations
CPU: ~4–6 hours
GPU: ~30–40 minutes

(Depends on dataset size and hardware.)

Summary

Pipeline:

Raw Frames → Merge Images → Merge COCO → Add Negatives
→ Split by Video → Convert to YOLO → Train Model

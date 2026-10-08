# Computer Vision Models

## Purpose
Perception layer - detect objects, segment scenes, estimate depth, classify images

## Models in This Suite

### Object Detection

#### YOLO v8 / v12
- **Task**: Real-time object detection
- **Output**: Bounding boxes + class labels
- **Use case**: Identify objects for manipulation

#### DETR / GroundingDINO
- **Architecture**: Transformer-based
- **Advantage**: Better accuracy for complex scenes
- **Use case**: Fine-grained object detection

### Segmentation

#### SAM (Segment Anything Model)
- **Task**: Universal image segmentation
- **Input**: Image + point/box prompt
- **Output**: Precise object masks

#### SAM2
- **Enhancement**: Video segmentation + tracking
- **Use case**: Track objects across frames

#### FastSAM / MobileSAM
- **Advantage**: Lightweight, faster inference
- **Trade-off**: Slightly lower accuracy
- **Use case**: Real-time applications

### Classification

#### CLIP
- **Task**: Zero-shot image classification
- **Input**: Image + text descriptions
- **Use case**: "Is this a red apple or green apple?"

### Depth Estimation

#### Depth Anything V2
- **Task**: Monocular depth estimation
- **Input**: Single RGB image
- **Output**: Depth map
- **Use case**: 3D scene understanding without depth camera

### Feature Extraction & Matching

#### SuperPoint
- **Task**: Interest point detection
- **Use case**: SLAM, visual odometry

#### LightGlue
- **Task**: Feature matching
- **Use case**: Match keypoints across images

## In This Suite

### Object Detection Example
- **Path**: `src/components/object-detection/`
- Shows ROS 2 + OpenVINO integration

### Multi-Camera Demo
- **Path**: `src/components/multicam-demo/`
- 4x RealSense cameras
- Simultaneous inference

## OpenVINO Optimization

All models are optimized for:
- **Device**: NPU (perception models) / GPU (heavy models)
- **Quantization**: INT8 for faster inference
- **Batch processing**: Multiple frames at once

## Typical Pipeline

```
Camera → Preprocessing → Model (OpenVINO) → Post-processing → ROS 2 Topic
                                                                    ↓
                                                            Downstream tasks
                                                        (manipulation, navigation)
```

## Key Files to Explore
- [ ] `src/components/object-detection/`
- [ ] `src/components/multicam-demo/`
- [ ] Model optimization scripts

## Learning Resources
- YOLO: https://github.com/ultralytics/ultralytics
- SAM: https://segment-anything.com/
- OpenVINO Model Zoo: https://github.com/openvinotoolkit/open_model_zoo

## My Experiments

### Date: [YYYY-MM-DD]
**Model tested:**

**Input source:**

**Performance (FPS):**

**Accuracy observations:**

---

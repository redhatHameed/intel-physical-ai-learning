# Vision-Language-Action (VLA) Models

## What They Are
AI models that combine vision understanding, language comprehension, and action generation for embodied AI

**Input**: Camera images + Text instruction  
**Output**: Robot joint positions / gripper commands

## Models in This Suite

### Pi0.5 + RTC (Real-Time Chunking)
- **Path**: `src/pipelines/pi05-rtc-ov/`
- **Size**: Medium (efficient for real-time)
- **Purpose**: General manipulation tasks
- **Key feature**: Chunking for responsive control

### RDT-1B (Robotics Diffusion Transformer)
- **Path**: `src/pipelines/rdt-ov/`
- **Size**: 1.2B parameters (large, high capability)
- **Purpose**: Complex manipulation
- **Key feature**: Diffusion-based foundation model

### Gr00t n1.7
- **Path**: `src/pipelines/gr00t-n1d7-ov/`
- **Provider**: NVIDIA
- **Purpose**: Humanoid robot control
- **Key feature**: Whole-body control

## How VLA Works

```
┌─────────────┐
│ Camera Feed │─────┐
└─────────────┘     │
                    ├──► VLA Model ──► Joint Positions
┌─────────────┐     │                  Gripper State
│ "Pick apple"│─────┘
└─────────────┘
```

## Key Concepts

### Imitation Learning
Model learns from demonstrations (human teleoperation recordings)

### Action Chunking
Generate multiple future actions at once for smooth control

### Multi-Modal Fusion
Combine vision features + language embeddings

## In This Suite

### Training
- Uses LeRobot framework
- Demonstration datasets
- Fine-tuning on specific tasks

### Inference
- OpenVINO optimized
- Real-time action generation
- ROS 2 integration for robot control

## Key Files to Explore
- [ ] `src/pipelines/pi05-rtc-ov/`
- [ ] Model conversion scripts
- [ ] Validation/inference code

## My Experiments

### Date: [YYYY-MM-DD]
**Model tested:**

**Task attempted:**

**Performance observations:**

**Comparison notes:**

---
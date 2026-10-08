# Diffusion Policies for Robot Control

## What They Are
AI models that generate robot actions through an iterative denoising process (like image diffusion models, but for actions)

**Analogy**: Think of "denoising" random movements into coherent robot actions

## Models in This Suite

### Diffusion Policy (Original)
- **Path**: `src/pipelines/diffusion-policy-ov/`
- **Paper**: https://diffusion-policy.cs.columbia.edu/
- **Purpose**: Visuomotor policy learning

### iDP3 (Improved 3D Diffusion Policy)
- **Path**: `src/pipelines/idp3-ov/`
- **Enhancement**: Better 3D spatial reasoning
- **Purpose**: Complex manipulation in 3D space

### ACT (Action Chunking with Transformers)
- **Path**: `src/pipelines/act-sample/`
- **Architecture**: Transformer-based
- **Purpose**: Chunked action prediction

## How Diffusion Works for Robotics

### Training
1. Take real robot demonstrations
2. Add noise to actions at different levels
3. Train model to predict noise and remove it

### Inference (Denoising Loop)
```
Random actions → Denoise step 1 → ... → Denoise step N → Clean actions
```

Typically 10-100 denoising steps

## Key Concepts

### Action Space
What the model predicts (joint angles, gripper state, end-effector pose)

### Observation Space
What the model sees (camera images, proprioception)

### Conditional Generation
Generate actions conditioned on current observations

## Advantages
- ✅ Multimodal action distributions (handle ambiguity)
- ✅ Stable training
- ✅ High-quality action generation

## Challenges
- ⚠️ Inference latency (multiple denoising steps)
- ⚠️ Memory footprint
- **Solution in this suite**: OpenVINO optimization, GPU acceleration

## In This Suite

### OpenVINO Optimization
- Model conversion to IR format
- INT8 quantization
- GPU/NPU acceleration

### Integration
- ROS 2 nodes for inference
- Real-time control loops
- Camera input processing

## Key Files to Explore
- [ ] `src/pipelines/diffusion-policy-ov/`
- [ ] `src/pipelines/idp3-ov/`
- [ ] `src/pipelines/act-sample/`

## My Experiments

### Date: [YYYY-MM-DD]
**Model tested:**

**Task complexity:**

**Inference latency:**

**Success rate:**

**Notes:**

---
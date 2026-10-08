# LLM Robotics Demo

**Path**: `src/pipelines/llm-robotics-demo/`  
**Documentation**: https://developer.robotics.intel.com/development-stack/software_references/humanoid/sample_pipelines/llm_robotics/

## Overview
Real-time system to control a robot arm with movement commands generated using an LLM

## Architecture
```
Speech (ASR) → LLM (Phi-4) → Code Generator → Robot Controller
                ↓
        Vision (MobileSAM) for object detection
```

## Prerequisites
- [ ] Phi-4 model (INT8 optimized)
- [ ] FunASR for speech recognition
- [ ] MobileSAM for object detection
- [ ] Robot arm / simulation
- [ ] RealSense camera

## Setup Steps

### 1. Model Downloads
```bash
# Phi-4 model path: ~/ov_models/Phi-4-mini-instruct-int8-ov
```

### 2. Launch Components
```bash
# ASR server
# TTS server
# Main LLM bridge
```

## Experiment Log

### Attempt #1 - [Date]
**Command tested**: "Pick up the red apple"

**Generated code:**
```python

```

**Execution result:**

**Issues:**

**Notes:**

---

### Attempt #2 - [Date]
**Command tested**:

**Generated code:**
```python

```

**Success rate:**

**Observations:**

---

## Performance Metrics

### LLM Inference
- **Latency** (command → code):
- **Token generation speed**:
- **Device**: GPU

### Vision Pipeline
- **Object detection latency**:
- **Accuracy**:

### End-to-End
- **Total latency** (speech → robot motion):

## Key Learnings

### What Works Well


### Challenges


### Potential Improvements


## Code Insights

### Interesting Prompt Engineering
```python
# From llm_bridge.py
```

### Robot Primitives Used
- `prepare_state()`
- `get_pick_pose()`
- `move()`
- `suck()`
- `release()`

## Questions / ToDo
- [ ] Test complex multi-step commands
- [ ] Measure failure rate
- [ ] Compare Phi-4 vs Qwen

---

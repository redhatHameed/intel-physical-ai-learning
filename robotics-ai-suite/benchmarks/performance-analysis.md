# Performance Benchmarking & Analysis

## Benchmarking Tools in Suite

**Path**: `src/benchmarks/`, `src/components/ros-kpi/`

## Benchmark Categories

### 1. Model Inference Performance

#### LLM (Phi-4)
- [ ] **Latency** (time to first token, tokens/sec)
- [ ] **Throughput** (requests/sec)
- [ ] **Memory usage**
- [ ] **Device**: GPU

**Results**:

| Metric | Value | Device | Config |
|--------|-------|--------|--------|
| First token latency | | | |
| Tokens/sec | | | |
| GPU memory | | | |

---

#### VLA Models (Pi0.5, RDT-1B)
- [ ] **Inference latency** (observation → action)
- [ ] **Batch size impact**
- [ ] **Memory footprint**

**Results**:

| Model | Latency (ms) | Memory (GB) | Device |
|-------|--------------|-------------|--------|
| Pi0.5 | | | |
| RDT-1B | | | |

---

#### Diffusion Policies
- [ ] **Denoising steps** vs **latency**
- [ ] **Quality** vs **speed** trade-off

**Results**:

| Steps | Latency (ms) | Quality Score |
|-------|--------------|---------------|
| 10 | | |
| 50 | | |
| 100 | | |

---

#### Computer Vision
- [ ] **FPS** (frames per second)
- [ ] **Detection accuracy**
- [ ] **Device comparison** (CPU vs GPU vs NPU)

**Results**:

| Model | CPU (FPS) | GPU (FPS) | NPU (FPS) |
|-------|-----------|-----------|-----------|
| YOLOv8 | | | |
| SAM | | | |

---

### 2. End-to-End Pipeline Performance

#### LLM Robotics Demo
```
Speech → ASR → Vision → LLM → Robot
```

**Breakdown**:

| Stage | Latency (ms) | Notes |
|-------|--------------|-------|
| ASR | | |
| Vision (MobileSAM) | | |
| LLM (code gen) | | |
| Robot execution | | |
| **Total** | | |

---

#### VLA Control Loop
```
Camera → Perception → VLA → Action → Robot
```

**Breakdown**:

| Stage | Latency (ms) | Target | Achieved |
|-------|--------------|--------|----------|
| Camera capture | | 33ms (30fps) | |
| Perception | | | |
| VLA inference | | | |
| Action execution | | | |
| **Loop frequency** | | 10Hz | |

---

### 3. Optimization Impact

#### Quantization (FP32 → INT8)

| Model | FP32 Latency | INT8 Latency | Speedup | Accuracy Delta |
|-------|--------------|--------------|---------|----------------|
| | | | | |

#### Device Selection

| Model | CPU | iGPU | NPU | Best Choice |
|-------|-----|------|-----|-------------|
| | | | | |

---

### 4. System Resource Usage

#### Memory Profile

| Component | RAM | GPU VRAM | Notes |
|-----------|-----|----------|-------|
| ROS 2 core | | N/A | |
| LLM (Phi-4) | | | |
| Vision models | | | |
| Total | | | |

#### CPU/GPU Utilization

| Workload | CPU % | GPU % | Bottleneck |
|----------|-------|-------|------------|
| Idle | | | |
| Object detection | | | |
| VLA inference | | | |
| Full pipeline | | | |

---

## Comparison: OpenVINO vs Other Runtimes

| Runtime | Model | Latency | Throughput | Notes |
|---------|-------|---------|------------|-------|
| OpenVINO | | | | |
| PyTorch (native) | | | | |
| ONNX Runtime | | | | |
| vLLM | | | | |

---

## Key Performance Insights

### What's Fast
1. 
2. 
3. 

### Bottlenecks Identified
1. 
2. 
3. 

### Optimization Opportunities
1. 
2. 
3. 

---

## Benchmarking Methodology

### Tools Used
- ROS 2 KPI monitoring
- OpenVINO benchmark_app
- Custom timing scripts

### Test Environment
- **Hardware**:
- **OS**:
- **Software versions**:

---

## Recommendations

### For Production Deployment
1. 
2. 
3. 

### For Further Optimization
1. 
2. 
3. 

---
